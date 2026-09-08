# Final board — generated from `gh api`, 2026-09-08 18:04 UTC

Regenerate with the commands in each row. Treat the timestamp as this
table's expiry.

| Item | State (from API) | Owner |
|---|---|---|
| solana-keychain#301 @ `8fb07b8a` | review=**CHANGES_REQUESTED** by dev-jodee, mergeable_state=blocked, **19 unresolved threads** — all answered locally, replies not yet posted. Repository CI has now run: 60+ checks green, `rust-lint` red (fixed locally), `fork-live-gate` red (normal on a fork, confirmed by the reviewer) | **Ledger** |
| pay-kit#300 @ `e8c81134` | review=CHANGES_REQUESTED by EfeDurmaz16, mergeable_state=**dirty** — conflicts with #308, which merged 2026-09-05. Rebased locally onto `c143bfab`, one commit, suite green; **not pushed**, sequencing is the maintainer's. 6 unresolved threads | Solana (sequencing), Ledger (rebase, ready) |
| pay-kit#309 @ `b935639d` | review=APPROVED by EfeDurmaz16, mergeable_state=unstable, 0 unresolved threads. Still unmerged | Solana |
| agave#15100 | labels=[community, need:merge-assist] applied by [mergify[bot]], ci-gate=**pending**, review=REVIEW_REQUIRED. Unchanged since 2026-09-05. Still needs someone with Agave rights to add the CI label | **Anza — no named owner** |

## Notes

**#301 is the only item whose ball is on our side**, and it is answered: 17 of
the 19 threads are fixed in commits, 2 are answered by issues. Nothing is
posted yet.

**agave#15100 is the item most likely to still be sitting here in three weeks.**
It is the only one with no named owner, and it is the one that blocks Ledger
support in pay-kit for anyone running Solana app 1.16.0 — which is every current
device. Reproduced again on hardware on 2026-09-08; see the evidence pack.

## Commands

```bash
gh pr view N --repo OWNER/REPO --json headRefOid,reviewDecision,mergeable,mergeStateStatus
gh pr checks N --repo OWNER/REPO
gh api graphql -f query='{repository(owner:"O",name:"R"){pullRequest(number:N){reviewThreads(first:100){nodes{isResolved}}}}}'
gh api repos/anza-xyz/agave/issues/15100/labels
```
