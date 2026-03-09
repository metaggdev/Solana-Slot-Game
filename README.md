# Slot Game on Solana — Decentralized Blockchain Slot Machine

![Version](https://img.shields.io/badge/version-0.0.1-blue.svg?cacheSeconds=2592000)
[![Solana](https://img.shields.io/badge/Solana-Blockchain-purple)](https://solana.com)
[![Anchor](https://img.shields.io/badge/Built%20with-Anchor-orange)](https://www.anchor-lang.com/)

A **slot game** built on the Solana blockchain: a fully on-chain slot machine with transparent logic, Phantom wallet support, and instant payouts. Play a **blockchain slot game** with provably fair outcomes and low fees.

**Live demo:** [solslot.coverlet.io](https://solslot.coverlet.io/)

![Solana Slot Game Demo](https://github.com/coverlet/solslot/blob/main/demo.gif)

---

## Why This Slot Game?

- **On-chain logic** — Spin and payouts run inside Solana smart contracts.
- **Fast & cheap** — Solana’s speed and low fees make each spin quick and affordable.
- **Self-custody** — Use Phantom; you keep control of your funds.
- **Transparent** — Open-source program and frontend; outcomes verifiable on-chain.

---

## Project Architecture

High-level layout of the **slot game** repo:

```
Solana-Slot-Game/
├── program/                 # Anchor (Solana) program — on-chain slot logic
│   ├── programs/slots/      # Rust crate: slot machine program
│   │   └── src/lib.rs       # Instructions: init, create_user_vault, spin, claim_winnings
│   ├── migrations/          # Deploy script (Anchor)
│   ├── tests/               # TypeScript integration tests (slots.ts)
│   ├── Anchor.toml          # Program ID, cluster, wallet config
│   └── Cargo.toml           # Workspace (programs/*)
│
├── client/                   # React frontend — slot game UI and wallet flow
│   ├── src/
│   │   ├── core/            # conn.ts, wallet.ts, transactions.ts, utils.ts
│   │   ├── components/      # slot-machine, slot-screen, reel, lever, blank
│   │   ├── slots.json       # IDL (program interface for the client)
│   │   └── App.tsx          # Main app: connect wallet, spin, claim
│   └── package.json
│
└── README.md
```

### On-chain (program)

| Component        | Role |
|-----------------|------|
| **Vault (PDA)** | Treasury PDA holds house bankroll; stores spin count and RNG seed. |
| **User vault (PDA)** | Per-player PDA holds pending winnings until claim. |
| **Instructions** | `init` (init treasury), `create_user_vault` (first-time user), `spin` (bet + outcome), `claim_winnings` (withdraw to wallet). |

### Off-chain (client)

| Component     | Role |
|---------------|------|
| **conn.ts**   | Solana connection (devnet) and Anchor provider (Phantom). |
| **wallet.ts** | Connect / disconnect Phantom, read balance and vault balance. |
| **transactions.ts** | Build and send `spin`, `claim_winnings`, `create_user_vault`, `init` via Anchor. |
| **UI**        | Slot machine, reels, lever; shows balance, jackpot, and claim button. |

Data flow: **User clicks Spin** → client calls `spin()` with vault/user_vault/signer → program takes 0.1 SOL, runs RNG, credits win to user vault PDA → user **claims** winnings from their PDA to wallet.

---

## How to Run This Project

### Prerequisites

- **Node.js** (v16+)
- **Rust** (stable) — [rustup](https://rustup.rs/)
- **Solana CLI** — `sh -c "$(curl -sSfL https://release.solana.com/stable/install)"`
- **Anchor** — [Install Anchor](https://www.anchor-lang.com/docs/installation)
- **Phantom** wallet (browser extension)
- **Devnet SOL** — [Solana Faucet](https://faucet.solana.com/) (for testing)

### 1. Build and deploy the program (slot game on-chain)

```bash
cd program
anchor build
```

Deploy to **devnet** (default cluster for the client):

```bash
solana config set --url devnet
anchor deploy --provider.cluster devnet
```

After deploy, copy the **program ID** from the build output and:

- Update `program/Anchor.toml` → `[programs.localnet]` / devnet if you use a different cluster.
- Update `client/src/slots.json` → `metadata.address` with the same program ID.
- If you use **localnet** for development, run a local validator and deploy to it:

```bash
# Terminal 1
solana-test-validator

# Terminal 2
cd program
solana config set --url localhost
anchor deploy --provider.cluster localnet
```

Then point the client to localnet in `client/src/core/conn.ts` (e.g. `http://127.0.0.1:8899`).

### 2. Run the client (slot game UI)

```bash
cd client
npm install
npm start
```

Browser opens at `http://localhost:3000`. Connect Phantom (set to **Devnet**), fund with devnet SOL, then use **SPIN** and **Claim** to play the slot game.

### 3. Run program tests

From `program/`:

```bash
# Start local validator in another terminal, then:
anchor test
```

---

## Slot Game Mechanics (On-Chain)

- **Bet:** 0.1 SOL per spin (fixed).
- **Outcomes (RNG from vault seed):**
  - **Lose** — 0 SOL.
  - **Small win** — 0.05 SOL.
  - **Standard win** — 0.1 SOL (break even).
  - **Big win** — 0.2 SOL (2×).

Winnings are credited to your **user vault PDA**; use **Claim** in the UI to withdraw to your Phantom wallet.

---

## Tech Stack

| Layer      | Tech |
|-----------|------|
| Blockchain | Solana (Devnet / Localnet) |
| Program   | Anchor, Rust |
| Frontend  | React, TypeScript, SCSS |
| Wallet    | Phantom (Solana) |

---

## Limitations & Disclaimer

- **RNG:** Pseudo-random (xorshift seeded from vault state); not suitable for production as-is. For a production slot game, use oracles (e.g. Switchboard, Chainlink VRF).
- **Educational use:** This repo is for learning Solana and Anchor. Use at your own risk; no guarantees on fairness or security for real money.
- **Known gaps:** Basic error handling, no full test suite, UI/UX and graphics are minimal.

---

## Author

**Telegram:** [@microRustyme](https://t.me/microRustyme) (microRustyme)

---

## Links

- **Live demo:** [solslot.coverlet.io](https://solslot.coverlet.io/)
- **Solana:** [solana.com](https://solana.com)
- **Devnet SOL:** [Solana Faucet](https://faucet.solana.com/)
- **Anchor:** [anchor-lang.com](https://www.anchor-lang.com/)
- **Solana Docs:** [docs.solana.com](https://docs.solana.com/)
