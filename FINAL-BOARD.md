# Final board — generated from `gh api`, 2026-09-11 00:46 UTC

Regenerate with the commands at the bottom. Treat the timestamp as this table's
expiry.

| Item | State (from API) | Owner |
|---|---|---|
| solana-keychain#301 @ `b1e50bfe` | **Maintainer has taken the branch.** Jo rebased all 15 of our commits onto current main and added two of his own — a −640-line comment prune and a fix commit (host-path redaction, unpinned version test, envelope note on the `SolanaSigner` trait). CI **59 pass / 1 fail / 1 skipped**; the single red is `fork-live-gate`, which he pre-cleared. AI-disclosure and attribution checks both pass. review=CHANGES_REQUESTED, not yet re-reviewed | **Solana — actively landing** |
| pay-kit#300 @ `e8c81134` | review=CHANGES_REQUESTED, mergeable_state=**dirty** — conflicts with #308, merged 2026-09-05. Rebased locally onto `c143bfab`, one commit, 968 tests green; **not pushed**, sequencing is the maintainer's call | Solana (sequencing), Ledger (rebase ready) |
| pay-kit#309 @ `b935639d` | review=APPROVED by EfeDurmaz16, mergeable_state=unstable, 0 unresolved threads. Still unmerged | Solana |
| agave#15100 | labels=[community, need:merge-assist], ci-gate=**pending**, review=REVIEW_REQUIRED. Unchanged since 2026-09-05 | **Anza — no named owner** |
| solana-keychain#306 | TypeScript/DMK signing design. Jo's `b1e50bfe` pre-answers part of it by documenting the envelope deviation on the trait | Solana (roadmap call) |
| solana-keychain#307 | Two attached Ledgers cannot be used concurrently | Unassigned |

## Notes

**#301's branch is shared now.** It is pushed to only with Ian's explicit go,
and never force-pushed.

**The repo gained an AI-disclosure gate on 2026-09-10** (`367d2f6`): a PR
template with a mandatory disclosure box, and `pr-hygiene.yml` which fails and
labels `ai-unreviewed` on any AI tool attribution in the title, body, commits
or branch name. Both checks currently pass on #301. The disclosure's
`Tool and extent:` field is still the template placeholder and should be filled
in properly.

**agave#15100 is still the only item with no named owner**, and it still blocks
Ledger support in pay-kit for every device running Solana app 1.16.0.

## Commands

```bash
gh pr view N --repo OWNER/REPO --json headRefOid,reviewDecision,mergeable,mergeStateStatus
gh pr checks N --repo OWNER/REPO
gh api repos/OWNER/REPO/commits/SHA/check-runs --jq '.check_runs[] | {name, conclusion}'
gh api graphql -f query='{repository(owner:"O",name:"R"){pullRequest(number:N){reviewThreads(first:100){nodes{isResolved}}}}}'
```
