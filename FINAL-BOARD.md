# Final board — generated from `gh api`, 2026-09-05 15:33 UTC

Regenerate with the commands in each row. Treat the timestamp as this
table's expiry.

| Item | State (from API) | Owner |
|---|---|---|
| pay-kit#309 @ `b935639d` | review=APPROVED by EfeDurmaz16, mergeable_state=unstable, Actions 9/9 action_required (none executed), 0 unresolved threads | Solana |
| pay-kit#300 @ `e8c81134` | review=CHANGES_REQUESTED, mergeable_state=blocked, Actions 9/9 action_required (none executed), 6 unresolved threads (left for the maintainer) | Solana |
| solana-keychain#301 @ `8fb07b8a` | review=REVIEW_REQUIRED, mergeable_state=blocked, Actions 4/4 action_required (none executed), 0 unresolved threads | Solana |
| agave#15100 | labels=[community, need:merge-assist] applied by [mergify[bot]], ci-gate=pending | Anza |
| agave#15099 | labels=[none] applied by [none] | Anza |

## Commands

```bash
gh api repos/OWNER/REPO/pulls/N --jq '.head.sha, .mergeable_state'
gh api 'repos/OWNER/REPO/actions/runs?head_sha=SHA' --jq '.workflow_runs[] | "\(.name) \(.conclusion)"'
gh api repos/anza-xyz/agave/issues/15100/labels
```
