# Final board — generated from `gh api`, 2026-09-08 18:32 UTC

Regenerate with the commands at the bottom. Treat the timestamp as this table's
expiry.

| Item | State (from API) | Owner |
|---|---|---|
| solana-keychain#301 @ `a9acfec4` | review=CHANGES_REQUESTED, mergeable_state=blocked. **All 19 of the reviewer's threads answered with replies naming their fix SHAs**; none resolved by us, deliberately — his review, his threads. 19 threads unresolved in total (19 his, 8 pre-existing greptile threads already resolved). Actions on this head: 4 runs, all `action_required` — approval is per-head, so the maintainer approval that let CI run on `8fb07b8` does not carry over | **Solana — needs re-review** |
| pay-kit#300 @ `e8c81134` | review=CHANGES_REQUESTED, mergeable_state=**dirty** — conflicts with #308, merged 2026-09-05. Rebased locally onto `c143bfab`, one commit, 968 tests green; **not pushed**, sequencing is the maintainer's call | Solana (sequencing), Ledger (rebase ready) |
| pay-kit#309 @ `b935639d` | review=APPROVED by EfeDurmaz16, mergeable_state=unstable, 0 unresolved threads. Still unmerged | Solana |
| agave#15100 | labels=[community, need:merge-assist], ci-gate=**pending**, review=REVIEW_REQUIRED. Unchanged since 2026-09-05. Still needs someone with Agave rights to add the CI label | **Anza — no named owner** |
| solana-keychain#306 | Filed 2026-09-08 at the reviewer's request: TypeScript/DMK signing design, moved out of `docs/` | Solana (roadmap call) |
| solana-keychain#307 | Filed 2026-09-08 at the reviewer's request: two attached Ledgers cannot be used concurrently | Unassigned |

## Notes

**#301's ball is with the reviewer.** 17 of his 19 comments are fixed in
commits, 2 are answered by #306 and #307. Two further defects were found by
hardware during this round and fixed — neither was in his review — and one
pre-existing macOS crash is flagged unfixed; all three are named in the summary
comment.

**Review re-request could not be sent.** `ledgicr` has `pull` permission only on
`solana-foundation/solana-keychain` (no push, no triage), and GitHub requires
write or triage to request a reviewer. `amilz` remains requested from before.
The summary comment asks for the re-review in words instead.

**agave#15100 remains the item most likely to still be sitting here in three
weeks.** It is the only one with no named owner, and it blocks Ledger support in
pay-kit for every device running Solana app 1.16.0 — reproduced again on
hardware on 2026-09-08, raw payloads in the evidence pack.

## Commands

```bash
gh pr view N --repo OWNER/REPO --json headRefOid,reviewDecision,mergeable,mergeStateStatus
gh pr checks N --repo OWNER/REPO
gh api "repos/OWNER/REPO/actions/runs?head_sha=SHA" --jq '.workflow_runs[] | "\(.name) \(.status)/\(.conclusion)"'
gh api graphql -f query='{repository(owner:"O",name:"R"){pullRequest(number:N){reviewThreads(first:100){nodes{isResolved}}}}}'
gh api repos/anza-xyz/agave/issues/15100/labels
```
