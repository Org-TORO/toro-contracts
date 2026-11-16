# TORO Solana Program

Anchor port of the original EVM `ToroRegistry` contract — seafood supply-chain
traceability with a two-level lifecycle:

- **Batches** (raw material): Source → Inventory → Manufacturing
- **Lots** (finished product): merged atomically from 1–16 input batches
  (all must be at Manufacturing), then Warehouse → Distribution

Rich trace data is event-only (same as EVM): payloads are the familiar
`abi.encode(uint256[] codes, bytes32[] values)` blobs decoded by
`data/CODE_REGISTRY.md`.

## Accounts (PDAs)

| Account | Seeds | Purpose |
|---------|-------|---------|
| `Config` | `["config"]` | authority, batch/lot counters |
| `Role` | `["factory", wallet]` / `["station", wallet]` | existence = authorized |
| `Batch` | `["batch", batch_id]` | stage 1–3 |
| `Lot` | `["lot", lot_code]` | stage 3–5, `input_batches[16]`, `total_cans` |

## Deployment

- **Cluster:** devnet
- **Program ID:** `2cbYretd93guxpURxqhq1UedBtwSHzT2NX6MsrBc4FWc`
- **Upgrade authority:** deployer wallet (`~/.config/solana/id.json`) — mock only

### Program keypair (not in git)

`target/deploy/toro-keypair.json` is gitignored because it is a secret. On a
fresh clone `anchor build` generates a *new random* keypair and then fails with
`Program ID mismatch`.

- **Restore the real keypair** from your secure backup to
  `target/deploy/toro-keypair.json` (its address must be
  `2cbYretd93guxpURxqhq1UedBtwSHzT2NX6MsrBc4FWc`).
- **Never run `anchor keys sync` on a deployed program** — it rewrites
  `declare_id!` to the new keypair's address and you would build/deploy to a
  different program address.
- For builds that do not deploy (CI, type checks) use `anchor build --ignore-keys`.

## Commands

```bash
anchor build                 # compile program + IDL
anchor test                  # local validator, 14 mocha tests
anchor deploy --provider.cluster devnet
yarn seed                    # seed TORO-01..TORO-05 demo traces (resumable)
yarn idl:sync                # build + publish idl/toro.json (committed artifact)
```

## Published artifact: `idl/toro.json`

`target/` is gitignored, so the IDL produced by `anchor build` never reaches
consumers. `idl/toro.json` is the **committed release artifact** that the
dapp's indexer reads. After any change to instructions, events, or account
layouts:

```bash
yarn idl:sync     # then commit idl/toro.json alongside the program change
```

Consumers must pin this file by commit SHA — an unversioned IDL is how a
UI silently starts decoding events with the wrong schema.

The seed script is idempotent: it skips steps already recorded on-chain,
paces transactions, and retries on RPC 429s. Re-run freely after failures.

## Indexer (lives in ui/)

```bash
cd ../../ui
npx tsx scripts/indexer-solana.ts   # crawls program events → src/data/traceIndex.json
```
