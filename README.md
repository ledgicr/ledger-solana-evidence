# Ledger + Solana: hardware validation evidence

Public copies of two artifacts referenced from
[solana-keychain#301](https://github.com/solana-foundation/solana-keychain/pull/301).
Nothing else lives here.

| File | What |
|---|---|
| [`HARDWARE-EVIDENCE-gen5.md`](HARDWARE-EVIDENCE-gen5.md) | Nineteen-phase hardware run on a Nano Gen5, with the scope gaps stated. **v3** corrects a phase v1-v2 reported on the strength of a test that could not fail; see the changelog at the top |
| [`FINAL-BOARD.md`](FINAL-BOARD.md) | PR and issue state, generated from `gh api`, timestamped in the file |

`FINAL-BOARD.md` is script-generated. Treat its timestamp as its expiry: if you
need current state, run the `gh api` commands in it rather than trusting the
table.
