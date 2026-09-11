# Final board — generated from `gh api`, 2026-09-11 13:59 UTC

Regenerate with the commands at the bottom. Treat the timestamp as this table's
expiry.

| Item | State (from API) | Owner |
|---|---|---|
| solana-keychain#301 @ `b1e50bfe` | **APPROVED by dev-jodee** 2026-09-11T12:54:49Z. External live tests **passed** (run 34600158104, all four languages). **Not merged**, and blocked by one red required check: `fork-live-gate` fails because the marker comment it reads was posted as `fork-external-live-pass:` with **no head SHA appended**, so the gate reports "Missing marker for fork PR head SHA b1e50bfe…". That is their workflow dropping `head_sha`, not anything about this branch. `reviewDecision=REVIEW_REQUIRED` because `amilz` is still a requested reviewer | **Solana — one CI action away from merge** |
| pay-kit#300 @ `e8c81134` | review=CHANGES_REQUESTED, mergeable_state=**dirty**. Rebase prepared locally onto `c143bfab`, one commit, 968 tests green; **not pushed** | Solana (sequencing) |
| pay-kit#309 @ `b935639d` | review=APPROVED, unmerged | Solana |
| agave#15100 | ci-gate=**pending**, no named owner, unchanged since 2026-09-05. **Now the highest-leverage item** — see below | **Anza** |
| solana-keychain#306 / #307 | Filed, open | Solana / unassigned |

## What changed: a Gen5 now works through pay-kit

Previously impossible. pay-kit resolves `solana-remote-wallet` **4.0.x**
(forced by `solana-pubkey 4.1.0` / `solana-signature 3.3.0`), and 4.0.3 carries
no Nano Gen5 product ids. Patching in 4.2.2 does not help — cargo refuses it as
unusable in the graph.

A **~40-line backport onto 4.0.3** does work: the 33 Gen5 product ids, one line
registering them, and the `config.len()` fix from agave#15100. It resolves,
compiles, and the device enumerates and derives addresses through pay-kit's own
x402 client path.

**This makes agave#15100 the highest-leverage open item.** A 4.0.x point release
carrying those ids plus the length fix would make every current Ledger work for
every pay.sh user with no pay-kit dependency change at all. The alternative —
pay-kit migrating to the Solana 4.x line — is much larger and is not scheduled.
The backport route was named as an option on 2026-09-01; it is now demonstrated.

## Commands

```bash
gh pr view N --repo OWNER/REPO --json headRefOid,reviewDecision,mergedAt,mergeStateStatus
gh pr checks N --repo OWNER/REPO
gh api repos/OWNER/REPO/commits/SHA/check-runs --jq '.check_runs[] | {name, conclusion}'
```
