# Decisions Log (contracts)

Newest first. Each entry: date, decision, rationale. The dapp has its own log.

## 2026-11-16 — Repository split: this repo owns the program only

**Decision:** the Anchor program lives in its own git repository
(`toro-contracts`); the Next.js dapp lives in `toro-dapp`. History was carried
per path with `git filter-repo` on clones, so the original monorepo remains
intact as an archive and nothing was rewritten in place.

**Rationale:** Rust/Anchor and Node/Next need different CI, and the program will
be audited and released on its own cadence. The only coupling was the IDL
artifact, which the dapp consumed from this repo's gitignored `target/`.

**Consequences:**
- `idl/toro.json` is committed as the release artifact; `yarn idl:sync` refreshes
  it after `anchor build`. Every change to instructions, events, or account
  layouts must update it in the same commit.
- `target/deploy/toro-keypair.json` stays untracked (it is a secret). A fresh
  clone generates a *different* program address, so restore the real keypair
  from backup — and never `anchor keys sync` on a deployed program.
- Cross-cutting changes are two commits in two repos, ordered: program → IDL →
  dapp decoding.
