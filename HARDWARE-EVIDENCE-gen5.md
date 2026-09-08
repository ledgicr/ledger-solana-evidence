# Ledger hardware evidence: Nano Gen5

| | |
|---|---|
| Device | Nano Gen5 (`apex_p`), USB PID `0x8000` |
| BOLOS | 1.1.1 |
| Solana app | **1.16.0** |
| Host | `Darwin 25.6.0 arm64` (macOS 15) |
| Date | 2026-09-08 (**v3**; v1–v2 were 2026-09-05) |
| Branch / head | `ledger-signer-v2` @ **`14d0840`** |
| Derived address | `8BnH5nwebNCY9txnohLUNECaUR6bQmwS67V7tBQF6hqY` (`m/44'/501'/0'`) |
| Device locator | `usb://ledger/EwL64GerPRHBXnjZQma8rPQ4Q7vF8czgzwYJwJ4yNEXg` |

Verify the head:

```bash
gh api repos/solana-foundation/solana-keychain/pulls/301 --jq '.head.sha'
```

---

## What changed in v3, and a correction to v2

**The blind-signing phase in v1 and v2 was backed by a test that could not
fail.** `test_ledger_non_ascii_offchain_message_needs_blind_signing` matched on
the result and printed which branch it took, asserting nothing. It therefore
passed whether the device signed or refused. The runbook ran it twice — once
with blind signing disabled, once enabled — and both phases expected a pass, so
both were green regardless of what the device did. Phase 9 of v2 reported
"**pass** — Refused with `0x6808`" on that basis. The refusal was real, but the
phase was not evidence of it: nothing would have gone red had the device signed.

Caught by @dev-jodee reviewing #301, in the plainest possible terms: "this test
passes whether signing succeeds or fails so both runbook phases can pass without
checking that blind signing changed the result."

The test now takes the device's configuration as an input and asserts the
opposite outcome for each value, and refuses to run at all if it is not told
which way round the device is set up. It has been **re-run in both directions,
with both negative controls**, and the four cells are phases 9a–9d below. The
old test passed in all four; the new one passes in two and fails in two.

Also in v3: phase 11 (Ledger Live) now **reproduces**, where v2 could not; the
busy refusal was re-measured; and a defect found in our own device-selection
code during this run is written up under *A serial is not an identity*.

**Read this caveat first.** Every result below was produced with a **one-character
local patch** to `solana-remote-wallet` 4.2.2, on a scratch branch
(`scratch/srw-length-check-proof`) that ships nowhere. Without it, no published
`solana-remote-wallet` can enumerate this device at all. That patch is the fix at
[anza-xyz/agave#15100](https://github.com/anza-xyz/agave/pull/15100).

---

## The blocker: an upstream length check

`solana-remote-wallet` cannot enumerate a device running Solana app 1.16.0.

`GET_APP_CONFIGURATION` (`0xe0 0x04`) answers status `0x9000` with a **7-byte**
payload:

```
00 00 01 10 00 00 00
```

`remote-wallet/src/ledger.rs:349` requires exactly five, so
`get_configuration_vector` fails, `update_devices` fails, and `list_devices`
returns zero ledgers. The deprecated fallback is never reached, because the first
call *succeeded*; probed directly it answers `0x6a83`.

**The five bytes it wants are all present and unchanged.** Against upstream's own
parsers (`get_settings`, `get_firmware_version`):

| Byte | Field | Value |
|---|---|---|
| 0 | `enable_blind_signing` | `00` (disabled) |
| 1 | `pubkey_display` | `00` (Long) |
| 2 | major | `01` |
| 3 | minor | `10` = 16 |
| 4 | patch | `00` |
| 5–6 | *(new, unread by 4.2.2)* | `00 00` |

That reconstructs **1.16.0**, matching BOLOS `getAppAndVersion` independently:

```
01 06 53 6f 6c 61 6e 61 06 31 2e 31 36 2e 30 01 02   -> "Solana" "1.16.0"
```

The app appended two trailing fields. `!= 5` rejects a payload it fully
understands; `< 5` parses it.

Present in the newest published release too: `4.4.0-alpha.3` carries the
identical check. Nothing downstream can bypass it, since it runs inside
`update_devices` with no hook.

**Not Gen5-specific.** The vector comes from the *app*, so any Ledger on 1.16.0
is affected. The Gen5 USB-PID gap (needs `solana-remote-wallet` >= 4.1) is real
but is a separate, earlier problem.

---

## Results

Nineteen phases. **All as expected.** All with the patch applied.

Phases marked *(v3)* were run on 2026-09-08 against head `14d0840`. The rest are
carried forward from v2 with their original heads and were **not** re-run; the
commits since changed device admission, HID path selection, error redaction,
documentation and the justfile, not the signing or envelope paths those phases
exercise.

| # | Phase | Result | Notes |
|---|---|---|---|
| 1 | BOLOS auto-launch, dashboard → Solana app | **pass** | Launched, connected, derived the address above |
| 2 | `test_ledger_pubkey_and_availability` | **pass** | |
| 3 | `test_ledger_reconnect_cycle_does_not_crash` | **pass** | The SIGTRAP regression |
| 4 | Reconnect × 20 consecutive | **20/20** | Process-wide device thread holds |
| 5 | `test_ledger_sign_offchain_message` | **pass** | Pins the 85-byte envelope; see below |
| 6 | `test_ledger_sign_transaction` | **pass** | Signature landed in slot 0 |
| 7 | F-14: reject → sign again, same signer | **pass** | Session survived the rejection |
| 8 | F-1: probe while a prompt is pending | **pass** | Returned inside the 10s tier |
| 9a | Non-ASCII off-chain, blind signing **off**, test told `disabled` *(v3)* | **pass** | Refused; error names blind signing as the remedy |
| 9b | Same, blind signing **off**, test told `enabled` *(v3)* | **fails, as required** | The negative control: proves 9a can go red |
| 9c | Non-ASCII off-chain, blind signing **on**, test told `enabled` *(v3)* | **pass** | Signed; signature **verifies** against the envelope |
| 9d | Same, blind signing **on**, test told `disabled` *(v3)* | **fails, as required** | The other control: proves 9c can go red |
| 10 | Locked device | **pass** | Suite **failed** rather than skipped |
| 11 | Ledger Live / Ledger Wallet.app holding the device *(v3)* | **pass — now reproduced** | See below; v2 could not reproduce this |
| 12 | Unplug/replug mid-run, 18 iterations | **pass** | **0 signal kills**; recovered after unlock |
| 13 | **Second signer during a live prompt** *(v3, re-measured)* | **pass** | **Refused in 130µs** (101µs in v2) |
| 14 | Reconnect driver via its new Just recipe | **pass** | Exit 0, classified PASS by the four-outcome logic |
| 15 | Dashboard auto-launch as a **test**, not an example *(v3)* | **pass** | BOLOS → Solana app; the converted `examples/ledger_open_app.rs` |
| 16 | Full hardware suite on the post-review head *(v3)* | **61 passed, 0 failed** | 25.6s, `--test-threads=1` |
| 17 | F-14 reject → sign again, same signer *(v3, re-run)* | **pass** | Session survived the rejection |

Heads: phases 1–8, 10, 12 against `508aa79`; 14 against `8fb07b8`; and
9a–9d, 11, 13, 15–17 against `14d0840`.

### 5 — what the off-chain phase actually proves

Three assertions, and the third is the one that matters:

1. the signature is 64 bytes;
2. it **verifies** against the bytes `ledger_offchain_envelope` builds;
3. it does **not** verify against the raw payload.

So the hand-built 85-byte V0 envelope is byte-correct against a real device.
This is the path that failed for months, because the obvious choice —
`solana_offchain_message`'s 20-byte header — is rejected outright.

### 9a–9d — blind signing, in both directions

The four cells, and why there are four. A single direction cannot distinguish
"the device gated this" from "the call happened to fail", which is how v2's
phase 9 came to be evidence of nothing.

| Device setting | Test told | Required outcome | Observed |
|---|---|---|---|
| off | `disabled` | refusal naming blind signing | refused |
| off | `enabled` | **failure** | failed |
| on | `enabled` | signature verifying against the envelope | signed and verified |
| on | `disabled` | **failure** | failed |

With the setting off, the refusal is `SigningFailed` and the message must name
blind signing — not merely be an error, because upstream renders APDU `0x6808`
as "Ledger operation not supported", and a regression to that wording would
leave the phase green while making the error useless to a user.

With it on, the signature is required to verify against the bytes
`ledger_offchain_envelope` builds, so this phase also re-confirms the 85-byte V0
envelope on a second payload — a non-ASCII one, format 1 (`LimitedUtf8`), where
phase 5 used printable ASCII.

Unset, the test panics with instructions rather than defaulting to a direction.
`just rust-ledger-evidence` exports `LEDGER_BLIND_SIGNING` per phase, so the
operator gains no new step.

**Blind signing was returned to disabled after this phase.**

### 10 — the locked device, and what it exposed

The point is that the suite must **fail, not skip**. `try_connect` skips only
when no device is attached, because a locked Gen5 once made this whole suite look
green while testing nothing. It failed, on all four hardware tests.

It also produced a finding. The message at the time hedged between "locked" and
"another application is holding the device", because the Solana-app APDUs cannot
separate them. Probing the locked device showed BOLOS `getAppAndVersion`
answering **`0x5515`**, the device-locked status word, which is unambiguous.
`current_app` was already making that call and discarding the answer: any
non-`0x9000` status became `Ok(None)`.

Fixed in `508aa79`, then re-run against the still-locked device:

```
a Ledger is attached but unusable, so this is a real failure rather than a skip:
the Ledger is locked. Enter your PIN on the device, then retry.
```

The hedge remains for every other `Protocol("Unknown error")`, where the two
causes really are indistinguishable.

### 11 — Ledger Live, reproduced this time

v2 could not reproduce this and recorded it as such. On 2026-09-08 it
reproduced without being asked to: `Ledger Wallet.app` was running at the start
of the session, holding the HID handle.

What it looks like from the host, and it is worth recording because two of the
three signals are misleading:

```
hidapi enumeration        -> device visible, pid=0x8000, both interfaces
LedgerSigner::is_attached -> true
BOLOS dashboard           -> cannot open the Ledger
update_devices            -> 0 device(s)
```

So `hidapi` enumerates the device while a raw HID open fails. That is the
signature of another process holding it, and it is distinguishable from the
app-config blocker: with the blocker, the dashboard open *succeeds* and only
`update_devices` fails.

The suite **failed rather than skipped**, which is the false-green guard doing
its job on a cause v2 never exercised. It failed with the wrong explanation,
though — see below.

**What v2 saw, for contrast.** With `Ledger Wallet.app` merely *running* —
device unlocked, Solana app open — the suite passed 40/40 in 92s and the phase
was recorded as *not reproduced*: the contention did not occur. v3 shows that
running is not the variable. The app has to have actually opened the device,
which it had at the start of this session and had not in v2. The
"another application is holding the device" cause was worth naming on the
strength of the `map_rw_err` comment alone; it is now observed.

### 12 — unplug/replug

18 iterations of the reconnect-cycle test across a physical unplug and replug:

```
passed=2  clean-failures=16  SIGNAL-KILLED=0
```

The two passes precede the unplug. The failures follow it and continued because
the device came back **locked**; every one was a clean `NotAvailable`, and the
new locked message identified it correctly. After unlocking, the cycle passed
again.

**Zero signal kills is the assertion.** A SIGTRAP or abort would be a regression
of the process-wide device thread, and both look like "non-zero exit" to a
script, so the loop checks `rc > 128` explicitly rather than inferring.

### 13 — the busy refusal, measured

The observable the atomic device claim exists to provide. With a **live
confirmation prompt on the device screen**, unanswered, a second
`sign_transaction` was fired from the same process:

```
second signer refused in 130µs: Ledger is busy with another operation or
awaiting on-device confirmation
```

(v2 measured 101µs on head `8fb07b8`. Same order of magnitude; the number is a
scheduling artefact, and the assertion is "under 2 seconds", not a specific
figure.)

Measurement context: `test_ledger_probe_returns_while_a_signature_is_pending`,
multi-thread tokio runtime, first signature dispatched and left pending for 3
seconds before the second call. Elapsed measured with `Instant::now()` around the
second `sign_transaction` await only. Asserted: refused, the error is the busy one
rather than a timeout, and under 2 seconds.

Before the atomic claim that call would have queued and waited out its full
120-second signing timeout. Four orders of magnitude, and it is the difference
between a contract and a comment.

What the operator does with the pending prompt at the end changes nothing: all
three assertions have already run by then, and the test's closing
`let _ = signing.await;` accepts either outcome deliberately.

---

## Four of our own error messages were wrong, and hardware said so

**The false-green guard worked, and that is how the blocker was found.** The
suite failed rather than skipping on an attached-but-unusable device, which is
exactly what it was written to do.

**"Locked or busy" was false.** With the device unlocked, the app open and the
dashboard answering, `map_rw_err` reported the two-cause hedge. Fixed in
`d568f6b`: a `Protocol("Version packet...")` now names the app-protocol
incompatibility and says the device is not at fault.

**"Ledger operation not supported" named no remedy.** The blind-signing refusal
surfaced as APDU `0x6808` with nothing actionable. Fixed in `b4e962d`, with the
host wording matched to what the device itself shows:

> ⚠ This transaction cannot be clear-signed
> Enable blind signing in the settings to sign this transaction.
> [Go to settings]  Reject transaction

Two things that screen taught us. The device's vocabulary is "clear-signed", so
the host error uses it, and a user reading the device and a developer reading a
log see the same words. And the modal **stays up and blocks every subsequent
command** until dismissed, which from the host is indistinguishable from a hung
device — confirmed by observation, since the next diagnostic only succeeded after
the prompt was rejected.

**"This requires solana-remote-wallet >= 4.1" was false, and new in v3.** With
`Ledger Wallet.app` holding the device, `no_ledger_enumerated_error` reported:

> a Ledger device is attached (product id 0x8000) but this build did not
> enumerate it. **This is a Nano Gen5, which requires solana-remote-wallet >=
> 4.1.** A build that resolved 4.0.x [...] cannot see it at all.

The build had resolved **4.2.2**. The requirement was met and the message sent
the reader to audit a dependency that was fine, while the actual cause was
process contention. The message asserts a version cause whenever a Gen5 is
attached and unenumerated, without checking what actually resolved — the same
shape as the "locked or busy" hedge above, and not yet fixed.

---

## A serial is not an identity

Found during this run, in our own code, and it is the defect the code was
written to prevent.

`ensure_solana_app_open(None)` must refuse when several Ledgers are attached
rather than acting on an arbitrary one. Doing that requires knowing which HID
interfaces belong to which physical device, because one Ledger can expose more
than one. The grouping used the USB serial number, on the reasoning that
interfaces of one device share it and two devices do not.

The second half is false. This device reports:

```
pid=0x8000 interface=2 usage_page=0xf1d0  path=DevSrvsID:4294981010  serial=Some("0001")
pid=0x8000 interface=0 usage_page=0xffa0  path=DevSrvsID:4294981014  serial=Some("0001")
```

`"0001"` is a fixed value, not a per-unit one, so **two different Ledgers report
the same serial**. Equality therefore proved nothing, two attached devices would
have been fused into one group, and the caller would have received the first of
them. The unit tests missed it because they invented distinct serials
(`"0001"`/`"0002"`); the hardware does not behave that way.

Fixed in `14d0840`. Grouping now requires the serial *and* the paths to agree,
and errs toward "two devices" when they disagree: being wrong permissively
writes app-management APDUs to an arbitrary security device, being wrong
strictly costs the caller an explicit `host_device_path`, and those are not
comparable.

**A known limit, recorded rather than asserted away.** On this platform the
paths are IOKit registry ids, and truncated to the last delimiter the pair above
shares only `DevSrvsID:` — 10 of 20 bytes, below the 80% adjacency threshold. So
two APDU interfaces on one device would be *refused* here rather than grouped.
It does not bite today because only one of the two passes `is_apdu_interface`:
interface 0, via the interface-number arm, since neither usage page is `0xFF00`
exactly. The measured values are in the test that pins this.

---

## A note on the device used

This device holds **11.851033379 SOL** on mainnet at
`8BnH5nwebNCY9txnohLUNECaUR6bQmwS67V7tBQF6hqY`, with its most recent on-chain
activity on 2026-08-20. It was assumed early on to be an empty test device and
that assumption went unchecked until late.

No risk materialised, and this is verifiable rather than assumed: nothing landed
on-chain, confirmed with `getSignaturesForAddress` before and after the signing
phases, and the test transactions are unbroadcastable by construction rather
than by luck. `test_util.rs:33` sets `recent_blockhash = Hash::default()`, which
is never in the recent-blockhash queue; the recipient is a `Pubkey::new_unique()`
existing only in process memory; and no test in the signing path holds an RPC
client.

Future hardware runs should use a device with a throwaway seed.

---

## Not covered

- **Two devices attached.** Still one Gen5 available, so the wrong-device
  dashboard fixes (`5607551`, `6b248eb`, `14d0840`) are verified by path lists
  and not by hardware. v3 narrows the gap rather than closing it: the lists now
  use the paths and serial this device actually reports, which is how the serial
  defect above was found, but no run has had two devices attached at once.
- **A second physical device's serial.** The claim that `"0001"` is not
  per-unit rests on one device reporting a fixed-format value on both its
  interfaces, plus the fact that it is not a plausible unique id. It has not
  been confirmed against a second Gen5. The fix does not depend on the claim
  being exactly right — it no longer trusts serial equality either way.
- **The runbook's crash branch.** Provoking a real SIGTRAP means reintroducing
  the regression it detects.
- **Nano S Plus / Nano X, and Linux.** Matrix is Gen5-on-macOS only. The
  app-version blocker is model-independent, so a second device hits the same wall
  unless it runs an older Solana app.
- **GitHub Actions on the v3 head.** Repository CI has now run, on `8fb07b8`,
  after a maintainer approved the workflow: 60+ checks green, including
  `rust-test` across sdk-v2/v3/v4 with the ledger backend, `miri` and
  `cargo-audit`. Two were red — `rust-lint`, five errors, fixed in `75bae2d`;
  and `fork-live-gate`, which the reviewer confirmed fails normally on a fork
  PR. CI has **not** yet run on `14d0840`. Everything in this pack is a local
  run either way.

```bash
gh pr checks 301 --repo solana-foundation/solana-keychain
```

## Reproducing

The backend's recipes now live in `rust/src/ledger/justfile` and are also
reachable as `just ledger::<recipe>`; the `rust-*` names below are forwarders
kept for exactly this reason.

```bash
just rust-ledger-diagnose                   # read-only; prints the blocker's raw payloads and serials
just rust-test-ledger                       # fails without the patch, on the app-config length
just rust-ledger-hw-test <test_name>        # one #[ignore]d hardware test
just rust-which-remote-wallet               # which solana-remote-wallet the graph resolved
just ledger::lint                           # clippy with the ledger feature on, which `rust-fmt` omits
just rust-ledger-evidence model="Nano Gen5" --firmware 1.1.1 --app-version 1.16.0
```

The blind-signing phases need to be told how the device is configured, and
refuse to run otherwise:

```bash
LEDGER_BLIND_SIGNING=disabled just rust-ledger-hw-test test_ledger_non_ascii_offchain_message_needs_blind_signing
LEDGER_BLIND_SIGNING=enabled  just rust-ledger-hw-test test_ledger_non_ascii_offchain_message_needs_blind_signing
```

**The patch.** No published `solana-remote-wallet` can enumerate this device, so
reproducing anything here needs the fix from
[agave#15100](https://github.com/anza-xyz/agave/pull/15100) applied locally. The
v1–v2 runs used a scratch branch that no longer exists; v3 used a copy of
registry 4.2.2 with that PR's diff applied, wired in as a `[patch.crates-io]`
entry pointing at a path outside the repository, and reverted immediately after
the run. It ships nowhere and is not part of #301.
