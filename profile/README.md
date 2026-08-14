# Crowder-Stellar

Building **ShieldFund** — a ZK-gated treasury dashboard for the Stellar / Soroban ecosystem. Manage
multi-sig vaults, run fundraising campaigns, stream real-time payments to contributors, and anchor
zero-knowledge proofs on-chain so treasury spending stays publicly auditable.

All contracts are live on **Stellar Testnet**.

## Repositories

| Repo | What it is |
|------|------------|
| [shieldfund-frontend](https://github.com/Crowder-Stellar/shieldfund-frontend) | React 19 / Vite / Tailwind dashboard — Freighter wallet, treasury/campaigns/streams/proofs tabs |
| [shieldfund-backend](https://github.com/Crowder-Stellar/shieldfund-backend) | Express API — read-only Soroban RPC indexing, Horizon event history, campaign metadata |
| [shieldfund-contracts](https://github.com/Crowder-Stellar/shieldfund-contracts) | Soroban/Rust contracts — treasury vault, payment streaming, ZK proof registry |
| [shieldfund-proof-server](https://github.com/Crowder-Stellar/shieldfund-proof-server) | Off-chain Noir proof server — proves payroll/budget compliance, anchors proof hashes on-chain |

## How it fits together

```
Freighter wallet ──signs──► frontend ──reads (RPC sim)──► backend (indexer/cache)
       │                                                          │
       └──────────────────────► contracts (treasury_vault, streaming, proof_registry)
                                          ▲
                                          │ register_proof(hash)
                                   proof-server (Noir circuit → bb prove → bb verify)
```

`treasury_vault.disburse()` requires a proof hash already anchored in `proof_registry` — the proof-server
is what makes that hash mean something, by actually proving allowlist membership and budget compliance in
zero knowledge before anchoring it.
