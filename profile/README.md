# Crowder-Stellar

<p align="left">
  <img src="https://img.shields.io/badge/Stellar-Testnet-08b5e5?logo=stellar&logoColor=white" alt="Stellar Testnet">
  <img src="https://img.shields.io/badge/Soroban-Smart%20Contracts-7B2FF7" alt="Soroban">
  <img src="https://img.shields.io/badge/ZK-Noir%20%2F%20UltraHonk-000000" alt="Noir ZK">
</p>

**ShieldFund** is a ZK-gated treasury dashboard for the Stellar / Soroban ecosystem, built for organizations
that need to spend public money and prove it. Deposit funds into a multi-sig vault, run fundraising
campaigns, stream real-time payments to contributors, and disburse against a zero-knowledge proof that the
payment was authorized — without publishing the full payroll list or the exact budget on a public ledger.

Everything below is deployed and wired together on **Stellar Testnet**.

## Repositories

| Repo | Stack | What it does |
|------|-------|---------------|
| [shieldfund-frontend](https://github.com/Crowder-Stellar/shieldfund-frontend) | React 19 · Vite · Tailwind v4 | Dashboard UI — Freighter wallet, treasury / campaigns / streams / proofs tabs |
| [shieldfund-backend](https://github.com/Crowder-Stellar/shieldfund-backend) | Express · TypeScript | Read-only indexer — Soroban RPC simulation reads, Horizon event history, campaign metadata (SQLite) |
| [shieldfund-contracts](https://github.com/Crowder-Stellar/shieldfund-contracts) | Rust · Soroban SDK | On-chain layer — treasury vault, payment streaming, ZK proof registry |
| [shieldfund-proof-server](https://github.com/Crowder-Stellar/shieldfund-proof-server) | Noir · Barretenberg (bb) · Express | Off-chain prover — proves payroll/budget compliance in zero knowledge, anchors the proof hash on-chain |

## Architecture

```
Freighter wallet
     │ signs deposit / disburse / stream / register_proof
     ▼
shieldfund-frontend ──reads (RPC simulation, no signing)──► shieldfund-backend
     │                                                            │ indexes
     ▼                                                            ▼
                    shieldfund-contracts (Soroban)
     ┌─────────────────┬───────────────┬───────────────────┐
     │  treasury_vault  │   streaming   │  proof_registry    │
     │  deposit/disburse│  per-second   │  register_proof     ◄── proof_hash,
     │  requires a      │  payment drip │  verify_proof_exists    public_inputs_hash
     │  proof_hash ─────┘               │                    │
     └──────────────────────────────────┴─────────▲──────────┘
                                                    │
                                     shieldfund-proof-server
                                (Noir circuit → bb prove → bb verify)
```

`treasury_vault.disburse()` requires a `proof_hash` already anchored in `proof_registry`. The proof-server
is what makes that hash mean something: it proves — via a real Noir circuit, actually proved and locally
verified with Barretenberg, not just asserted — that the recipient is on an approved allowlist and the
amount doesn't exceed a committed budget, then anchors the resulting hash. The rest of the allowlist and
the exact budget figure stay private; only membership and compliance are proven.

## Live Testnet Deployment

Deployed 2026-06-28. Frontend and backend both read from these by default — no setup required to try it.

| Contract | ID |
|----------|-----|
| Treasury Vault | [`CAUWJPC7…76Y5RETIG`](https://stellar.expert/explorer/testnet/contract/CAUWJPC73YLQMSV6X4QPLUVS2UZFE2PMRIQSSCDN62DNN6J76Y5RETIG) |
| Streaming | [`CDU7ZIVQ…RWL6THE6WQ`](https://stellar.expert/explorer/testnet/contract/CDU7ZIVQ3UC4K3DHV3NMQGW5UMSYFCKCC6YJKHT4YLNEZJRWL6THE6WQ) |
| Proof Registry | [`CBDLHQQP…3F2F6AO2332`](https://stellar.expert/explorer/testnet/contract/CBDLHQQPKC5524CFWPD4HMPTZGWBYQNW3IKGAFH6IAYBU3F2F6AO2332) |
| XLM Token SAC | [`CDLZFC3S…U2HHGCYSC`](https://stellar.expert/explorer/testnet/contract/CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC) |
| Admin Account | [`GBJ5FP5U…IDVWAN3C7P`](https://stellar.expert/explorer/testnet/account/GBJ5FP5UB4YUE2EONTPPSAGKZZGDETFZLEJXJRCALSYTJZIDVWAN3C7P) |

## Getting started

Each repo runs independently — start with whichever layer you want:

```bash
git clone https://github.com/Crowder-Stellar/shieldfund-frontend.git   && cd shieldfund-frontend   && npm install && npm run dev   # :3000
git clone https://github.com/Crowder-Stellar/shieldfund-backend.git    && cd shieldfund-backend    && npm install && npm run dev   # :4000
git clone https://github.com/Crowder-Stellar/shieldfund-proof-server.git && cd shieldfund-proof-server && npm install && npm start # :4100
```

`shieldfund-contracts` requires the Rust + Soroban toolchain; `shieldfund-proof-server` requires Noir
(`nargo`) and Barretenberg (`bb`) — see each repo's README for exact versions and install steps.

## Status

This is a working prototype, not a production system: testnet only, contract IDs are hardcoded rather than
configurable per-deployment, and there's no on-chain ZK verifier — `proof_registry` anchors a hash that the
proof-server promises corresponds to a real, locally-verified proof, but the chain itself still trusts
whichever address is authorized to call `register_proof()`. That trust boundary, and the rest of what's
still missing, is written up honestly in each repo's own README rather than glossed over here.
