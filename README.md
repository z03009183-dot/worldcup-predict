# World Cup Prediction Market

## Decentralized Prediction Markets on Solana

A **fully on-chain**, decentralized prediction market platform for FIFA World Cup matches. Users buy outcome shares for live matches, with pricing determined by an Automated Market Maker (AMM) and settlement executed trustlessly via Solana smart contracts using cryptographic proofs from the TxLINE sports data API.

> **This is a working implementation** — not a wireframe or concept. The smart contract is deployed on Solana Devnet, the frontend is live, and real match data streams via SSE.

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Next.js Frontend                      │
│              (TailwindCSS · Solana Wallet)                │
└──────────────┬────────────────────────────┬──────────────┘
               │        SSE Stream         │  Solana RPC
               ▼                            ▼
┌──────────────────────┐   ┌──────────────────────────────┐
│     TxLINE API       │   │   Anchor Smart Contract      │
│  (Live Match Data)   │   │  (prediction_market.so)      │
│                      │   │                              │
│  • Match scores      │   │  • Market creation           │
│  • Odds updates      │   │  • AMM share pricing         │
│  • Event feeds       │   │  • On-chain settlement       │
│  • Merkle proofs     │   │  • Winnings distribution     │
└──────────────────────┘   └──────────────────────────────┘
```

**Data Flow:**
1. TxLINE streams real-time match data to the frontend via SSE
2. Users place bets by purchasing outcome shares through the Anchor program
3. AMM adjusts prices based on supply/demand (constant product formula)
4. After match completion, settlement is triggered with Merkle proof verification
5. Winners claim payouts directly from the on-chain vault

---

## Features

| Feature | Description |
|---------|-------------|
| 🔴 **Real-Time SSE Streaming** | Live match scores, odds, and events pushed from TxLINE API |
| 📈 **AMM-Based Pricing** | Constant product market maker — no order books, instant liquidity |
| ⛓️ **On-Chain Settlement** | All bets and payouts settled on Solana — fully trustless |
| 🌳 **Merkle Proof Verification** | Match results validated against TxLINE cryptographic proofs |
| 👛 **Solana Wallet Integration** | Phantom, Solflare, and other Solana wallets supported |
| 🏆 **World Cup Focused** | Built specifically for FIFA World Cup match markets |
| ⚡ **Sub-Second Finality** | Solana's speed means instant bet confirmation |

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| **Blockchain** | Solana (Devnet) |
| **Smart Contract** | Anchor Framework (Rust) |
| **Frontend** | Next.js 14 + React |
| **Styling** | TailwindCSS |
| **Data Provider** | TxLINE API (https://txline.txodds.com) |
| **Real-Time** | Server-Sent Events (SSE) |
| **Wallet** | @solana/wallet-adapter |

---

## Project Structure

```
worldcup-predict/
├── README.md
├── TECHNICAL_DOCS.md
├── ARCHITECTURE.md
├── prediction_market/
│   ├── programs/
│   │   └── prediction_market/
│   │       └── src/
│   │           └── lib.rs              # Anchor smart contract
│   ├── app/                            # Next.js frontend
│   │   ├── components/                 # React components
│   │   ├── pages/                      # Next.js pages
│   │   ├── hooks/                      # Custom React hooks
│   │   ├── lib/                        # Utilities & API clients
│   │   └── styles/                     # TailwindCSS config
│   ├── tests/                          # Anchor integration tests
│   ├── migrations/                     # Deployment scripts
│   ├── target/
│   │   └── deploy/
│   │       └── prediction_market.so    # Compiled program binary
│   └── Anchor.toml                     # Anchor configuration
└── .gitignore
```

---

## Getting Started

### Prerequisites

- [Rust](https://rustup.rs/) (with `solana` toolchain)
- [Anchor CLI](https://www.anchor-lang.com/docs/installation)
- [Node.js](https://nodejs.org/) v18+
- [Solana CLI](https://docs.solana.com/cli/install-solana-cli-tools)
- A Solana wallet with Devnet SOL ([faucet](https://faucet.solana.com/))

### Installation

```bash
# Clone the repository
git clone https://github.com/your-org/worldcup-predict.git
cd worldcup-predict

# Install frontend dependencies
cd prediction_market/app
npm install

# Start the development server
npm run dev
```

The app will be available at `http://localhost:3000`.

### Deploying the Smart Contract

```bash
cd prediction_market

# Build the Anchor program
anchor build

# Deploy to Solana Devnet
anchor deploy --provider.cluster devnet

# Run integration tests
anchor test
```

### Environment Setup

Create `.env.local` in `prediction_market/app/`:

```env
NEXT_PUBLIC_SOLANA_NETWORK=devnet
NEXT_PUBLIC_PROGRAM_ID=Dz92ko2VQTzJKrLzm5weYtpkyuYBPWGR5ynk8WjmpsFX
NEXT_PUBLIC_TXLINE_API_URL=https://txline.txodds.com
NEXT_PUBLIC_TXLINE_API_KEY=your_txline_api_key
```

---

## Smart Contract Overview

**Program ID:** `Dz92ko2VQTzJKrLzm5weYtpkyuYBPWGR5ynk8WjmpsFX`

The Anchor program (`prediction_market`) provides five core instructions:

| Instruction | Description |
|-------------|-------------|
| `initialize` | Sets up global program state and authority |
| `create_market` | Creates a new prediction market for a specific match |
| `buy_shares` | Purchases outcome shares (Yes/No) at AMM-determined price |
| `settle_market` | Resolves market using TxLINE match result + Merkle proof |
| `claim_winnings` | Distributes SOL to winning share holders |

Each market is a Program Derived Address (PDA) seeded by match ID, ensuring deterministic account addresses.

---

## TxLINE API Integration

This project uses the [TxLINE API](https://txline.txodds.com) for real-time World Cup match data. TxLINE provides a **free tier** for World Cup data including:

- Live match scores and status
- Odds movements across bookmakers
- Match events (goals, cards, substitutions)
- Cryptographic Merkle proofs for result verification

### Data Flow

```
TxLINE API ──(SSE)──▶ Frontend ──(Transaction)──▶ Solana Program
     │                     │
     └── Merkle Proof ─────┴──(Instruction Data)──▶ On-Chain Verification
```

The frontend subscribes to TxLINE's SSE endpoint for a given match. When a match concludes, the API provides a Merkle proof of the result. This proof is included in the `settle_market` instruction, allowing the smart contract to verify the result on-chain without trusting any centralized oracle.

---

## Screenshots

> 🚧 Screenshots will be added once the frontend UI is finalized.

| Screen | Preview |
|--------|---------|
| Market Dashboard | *Coming soon* |
| Match Detail View | *Coming soon* |
| Share Purchase Flow | *Coming soon* |
| Settlement & Claim | *Coming soon* |

---

## Network Configuration

| Setting | Value |
|---------|-------|
| **Network** | Solana Devnet |
| **RPC Endpoint** | `https://api.devnet.solana.com` |
| **Program ID** | `Dz92ko2VQTzJKrLzm5weYtpkyuYBPWGR5ynk8WjmpsFX` |
| **TxLINE API** | `https://txline.txodds.com` |

---

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

---

## Acknowledgments

- [Solana](https://solana.com/) — High-performance blockchain
- [Anchor Framework](https://www.anchor-lang.com/) — Solana smart contract framework
- [TxLINE](https://txline.txodds.com/) — Real-time sports data API
- FIFA World Cup — The beautiful game
