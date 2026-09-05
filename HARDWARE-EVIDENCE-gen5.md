# Ledger hardware evidence: Nano Gen5

| | |
|---|---|
| Device | Nano Gen5 (`apex_p`), USB PID `0x8000` |
| BOLOS | 1.1.1 |
| Solana app | **1.16.0** |
| Host | `Darwin 25.6.0 arm64` (macOS 15) |
| Date | 2026-09-05 |
| Branch / head | `ledger-signer-v2` @ **`8fb07b8`** |
| Derived address | `8BnH5nwebNCY9txnohLUNECaUR6bQmwS67V7tBQF6hqY` (`m/44'/501'/0'`) |
| Device locator | `usb://ledger/EwL64GerPRHBXnjZQma8rPQ4Q7vF8czgzwYJwJ4yNEXg` |

Verify the head:

```bash
gh api repos/solana-foundation/solana-keychain/pulls/301 --jq '.head.sha'
```

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

Fourteen phases. **Thirteen as expected, one not reproduced.** All with the
patch applied.

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
| 9 | Non-ASCII off-chain, blind signing **off** | **pass** | Refused with `0x6808` |
| 10 | Locked device | **pass** | Suite **failed** rather than skipped |
| 11 | Ledger Live running | **not reproduced** | Suite passed 40/40 |
| 12 | Unplug/replug mid-run, 18 iterations | **pass** | **0 signal kills**; recovered after unlock |
| 13 | **Second signer during a live prompt** | **pass** | **Refused in 101µs** |
| 14 | Reconnect driver via its new Just recipe | **pass** | Exit 0, classified PASS by the four-outcome logic |

Phases 1–12 were run against head `508aa79`; 13–14 against `8fb07b8` after the
second review round. Nothing in 1–12 is invalidated by the commits between those
heads, which changed device admission, HID path selection, documentation and the
runbook driver, not the signing or envelope paths those phases exercise.

### 5 — what the off-chain phase actually proves

Three assertions, and the third is the one that matters:

1. the signature is 64 bytes;
2. it **verifies** against the bytes `ledger_offchain_envelope` builds;
3. it does **not** verify against the raw payload.

So the hand-built 85-byte V0 envelope is byte-correct against a real device.
This is the path that failed for months, because the obvious choice —
`solana_offchain_message`'s 20-byte header — is rejected outright.

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

### 11 — Ledger Live, not reproduced

With `/Applications/Ledger Wallet.app` running, device unlocked, Solana app open,
the suite **passed 40/40** in 92s, including both signing tests.

Recorded as *not reproduced* rather than pass or fail: the phase exists to
observe a contention that did not occur. The `map_rw_err` comment asserts this
was once seen on a Gen5 with Ledger Live running; that observation predates this
work and could not be confirmed. The "another application is holding the device"
cause is still worth naming — any process holding the HID handle produces it —
but Ledger Live merely *running* was not sufficient on macOS.

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
second signer refused in 101µs: Ledger is busy with another operation or
awaiting on-device confirmation
```

Measurement context: `test_ledger_probe_returns_while_a_signature_is_pending`,
multi-thread tokio runtime, first signature dispatched and left pending for 3
seconds before the second call. Elapsed measured with `Instant::now()` around the
second `sign_transaction` await only. Asserted: refused, the error is the busy one
rather than a timeout, and under 2 seconds.

Before the atomic claim that call would have queued and waited out its full
120-second signing timeout. Four orders of magnitude, and it is the difference
between a contract and a comment.

The operator approved rather than rejected the pending prompt at the end. That
changes nothing: all three assertions had already run and passed, and the test's
closing `let _ = signing.await;` accepts either outcome deliberately.

---

## Three of our own error messages were wrong, and hardware said so

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

- **Blind signing enabled.** Left off deliberately; it is a device security
  setting and enabling it to make a test pass is the wrong instinct.
- **Two devices attached.** One Gen5 available, so the wrong-device dashboard fix
  (`5607551`) is verified by synthetic path lists, not hardware.
- **The runbook's crash branch.** Provoking a real SIGTRAP means reintroducing
  the regression it detects.
- **Nano S Plus / Nano X, and Linux.** Matrix is Gen5-on-macOS only. The
  app-version blocker is model-independent, so a second device hits the same wall
  unless it runs an older Solana app.
- **GitHub Actions.** Four workflow runs exist on this head and all are
  `action_required`, awaiting maintainer approval; none has executed. Greptile
  ran and passed. Everything above is a local run.

```bash
gh api "repos/solana-foundation/solana-keychain/actions/runs?head_sha=8fb07b8abf68a99e5c88489f893509738d39d072" \
  --jq '.workflow_runs[] | "\(.name) \(.status)/\(.conclusion)"'
```

## Reproducing

```bash
just rust-ledger-diagnose                  # read-only; prints the blocker's raw payloads
just rust-test-ledger                      # fails without the patch, on the app-config length
just rust-ledger-hw-test <test_name>        # one #[ignore]d hardware test
just rust-ledger-evidence model="Nano Gen5" --firmware 1.1.1 --app-version 1.16.0
```
