# Final board — generated from `gh api`, 2026-09-11 01:06 UTC

Regenerate with the commands at the bottom. Treat the timestamp as this table's
expiry.

| Item | State (from API) | Owner |
|---|---|---|
| solana-keychain#301 @ `b1e50bfe` | **Maintainer holds the branch.** Jo rebased all 15 of our commits onto current main and added two: a −640-line comment prune and a fix commit (host-path redaction, unpinned version test, envelope note on the `SolanaSigner` trait). Both reviewed and agreed; one factual note raised on a pruned comment. AI disclosure now filled in properly and **both hygiene checks green**, no `ai-unreviewed` label. Checks **59 pass / 1 fail**, the one red being `fork-live-gate`, pre-cleared. review=CHANGES_REQUESTED, not yet re-reviewed | **Solana — Jo's move** |
| pay-kit#300 @ `e8c81134` | review=CHANGES_REQUESTED, mergeable_state=**dirty** — conflicts with #308, merged 2026-09-05. Rebased locally onto `c143bfab`, one commit, 968 tests green; **not pushed**, sequencing is the maintainer's call | Solana (sequencing), Ledger (rebase ready) |
| pay-kit#309 @ `b935639d` | review=APPROVED by EfeDurmaz16, mergeable_state=unstable. Still unmerged | Solana |
| agave#15100 | labels=[community, need:merge-assist], ci-gate=**pending**, review=REVIEW_REQUIRED. Unchanged since 2026-09-05 | **Anza — no named owner** |
| solana-keychain#306 | TypeScript/DMK signing design. `b1e50bfe` pre-answers part of it by documenting the envelope deviation on the trait | Solana (roadmap call) |
| solana-keychain#307 | Two attached Ledgers cannot be used concurrently | Unassigned |

## Notes

**Everything on #301 is now with Jo.** Nineteen review threads answered, both
his commits reviewed and agreed in a posted reply, disclosure completed. The
branch is shared: pushed to only on Ian's explicit go, never force-pushed.

**The AI-disclosure gate is satisfied on the merits, not just mechanically.**
The `Tool and extent` field was the template placeholder until 2026-09-11; it
now states which tools were used, what they drafted, and how the work was
verified — hardware on a physical Gen5, mutation-checked tests, independent
re-audit before each push.

**agave#15100 remains the only item with no named owner**, and still blocks
Ledger support in pay-kit for every device running Solana app 1.16.0.

## Commands

```bash
gh pr view N --repo OWNER/REPO --json headRefOid,reviewDecision,mergeable,mergeStateStatus
gh pr checks N --repo OWNER/REPO
gh api repos/OWNER/REPO/actions/runs --jq '[.workflow_runs[]|select(.name=="PR hygiene")]|sort_by(.created_at)|last'
```
