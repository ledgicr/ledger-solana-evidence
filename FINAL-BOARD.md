# Final board — generated from `gh api`, 2026-09-08 19:20 UTC

Regenerate with the commands at the bottom. Treat the timestamp as this table's
expiry.

| Item | State (from API) | Owner |
|---|---|---|
| solana-keychain#301 @ `4e37f6a9` | review=CHANGES_REQUESTED, mergeable_state=blocked, 15 commits. **All 19 reviewer threads answered**, none resolved by us. Two replies claimed removals `ffca874` did not contain; both are true as of `4e37f6a`, corrections posted on their threads, and the summary comment edited in place with a visible edit note. Greptile: `success` on this head. **Actions: 4 runs, all `action_required`** | **Solana — needs re-review** |
| pay-kit#300 @ `e8c81134` | review=CHANGES_REQUESTED, mergeable_state=**dirty** — conflicts with #308, merged 2026-09-05. Rebased locally onto `c143bfab`, one commit, 968 tests green; **not pushed**, sequencing is the maintainer's call | Solana (sequencing), Ledger (rebase ready) |
| pay-kit#309 @ `b935639d` | review=APPROVED by EfeDurmaz16, mergeable_state=unstable, 0 unresolved threads. Still unmerged | Solana |
| agave#15100 | labels=[community, need:merge-assist], ci-gate=**pending**, review=REVIEW_REQUIRED. Unchanged since 2026-09-05 | **Anza — no named owner** |
| solana-keychain#306 | Filed 2026-09-08 at the reviewer's request: TypeScript/DMK signing design, moved out of `docs/` | Solana (roadmap call) |
| solana-keychain#307 | Filed 2026-09-08 at the reviewer's request: two attached Ledgers cannot be used concurrently | Unassigned |

## Notes

**#301 needs two clicks from the reviewer, not one.** Workflow approval is
per-head, so the approval that let CI execute on `8fb07b8` carried to none of
`a9acfec4`, `18149e9d` or `4e37f6a9`. All four runs sit at `action_required`: approve the
Actions run, then review. Greptile is not a GitHub Action and ran on its own.

**The lint fix is therefore not yet confirmed by their CI**, only locally —
`just rust-test` green across sdk-v2/v3/v4 and `just ledger::lint` clean with
the ledger feature enabled, which the repo's own `rust-fmt` recipe does not do
because `all` omits `ledger`.

**Review re-request could not be sent.** `ledgicr` has `pull` permission only on
the base repo, and GitHub requires write or triage to request a reviewer.
`amilz` remains requested from before; the summary comment asks in words.

**agave#15100 is still the item most likely to be sitting here in three weeks.**
Only one with no named owner, and it blocks Ledger support in pay-kit for every
device running Solana app 1.16.0 — reproduced again on hardware 2026-09-08.

**Posted claims are now checked against the tree, not the plan.** Two replies in
the previous round described removals that their named commit did not contain.
They survived verification because that pass confirmed the replies were posted
and the SHAs existed, never what those SHAs contained. `claims-vs-tree.py` now
pulls posted text from the API and asserts, for each reply on a thread anchored
to a path, that any SHA it names actually touches that path, following renames.

## Commands

```bash
gh pr view N --repo OWNER/REPO --json headRefOid,reviewDecision,mergeable,mergeStateStatus
gh pr checks N --repo OWNER/REPO
gh api repos/OWNER/REPO/commits/SHA/check-runs --jq '.check_runs[] | {name, conclusion, summary: .output.summary}'
gh api "repos/OWNER/REPO/actions/runs" --jq '[.workflow_runs[] | select(.head_sha|startswith("SHA"))] | map("\(.name)=\(.status)/\(.conclusion)")'
gh api graphql -f query='{repository(owner:"O",name:"R"){pullRequest(number:N){reviewThreads(first:100){nodes{isResolved}}}}}'
```
