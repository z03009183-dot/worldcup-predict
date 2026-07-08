# World Cup Prediction Market

> Decentralized Prediction Markets on Solana

A **fully on-chain**, decentralized prediction market platform for FIFA World Cup matches. Users buy outcome shares for live matches, with pricing determined by an Automated Market Maker (AMM) and settlement executed trustlessly via Solana smart contracts using cryptographic proofs from the TxLINE sports data API.

## 🏗️ Architecture

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

## ✨ Features

| Feature | Description |
|---------|-------------|
| 🔴 **Real-Time SSE Streaming** | Live match scores, odds, and events pushed from TxLINE API |
| 📈 **AMM-Based Pricing** | Constant product market maker — no order books, instant liquidity |
| ⛓️ **On-Chain Settlement** | All bets and payouts settled on Solana — fully trustless |
| 🌳 **Merkle Proof Verification** | Match results validated against TxLINE cryptographic proofs |
| 👛 **Solana Wallet Integration** | Phantom, Solflare, and other Solana wallets supported |
| 🏆 **World Cup Focused** | Built specifically for FIFA World Cup match markets |
| ⚡ **Sub-Second Finality** | Solana's speed means instant bet confirmation |

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| **Blockchain** | Solana (Devnet) |
| **Smart Contract** | Anchor Framework (Rust) |
| **Frontend** | Next.js 14 + React |
| **Styling** | TailwindCSS |
| **Data Provider** | TxLINE API (https://txline.txodds.com) |
| **Real-Time** | Server-Sent Events (SSE) |
| **Wallet** | @solana/wallet-adapter |

## 📁 Project Structure

```
worldcup-predict/
├── README.md
├── TECHNICAL_DOCS.md
├── ARCHITECTURE.md
├── SUBMISSION.md
├── prediction_market/
│   ├── programs/
│   │   └── prediction_market/
│   │       └── src/
│   │           └── lib.rs              # Anchor smart contract
│   ├── app/                            # Next.js frontend
│   │   ├── page.tsx                    # Home page
│   │   ├── markets/
│   │   │   └── page.tsx                # Markets listing
│   │   └── portfolio/
│   │       └── page.tsx                # Portfolio tracking
│   ├── Anchor.toml
│   └── package.json
└── SUBMISSION.md
```

## 🚀 Quick Start

### Prerequisites

- Node.js 18+
- Rust & Cargo
- Solana CLI
- Anchor CLI

### Installation

```bash
# Clone the repository
git clone https://github.com/z03009183-dot/worldcup-predict.git
cd worldcup-predict

# Install dependencies
cd prediction_market/app
npm install

# Build smart contract
cd ..
anchor build

# Deploy to devnet
anchor deploy --provider.cluster devnet

# Start frontend
cd app
npm run dev
```

### Environment Variables

Create `.env.local` in the app directory:

```env
NEXT_PUBLIC_SOLANA_NETWORK=devnet
NEXT_PUBLIC_TXLINE_API_URL=https://txline.txodds.com
```

## 📖 Smart Contract Instructions

### 1. `initialize`
One-time setup of global program state.

### 2. `create_market`
Create a new prediction market for a specific match.

### 3. `buy_shares`
Buy outcome shares with USDC (constant product AMM).

### 4. `settle_market`
Settle markets using TxLINE data with Merkle proof verification.

### 5. `claim_winnings`
Claim USDC winnings from resolved markets.

## 🔧 AMM Formula

The prediction market uses a constant product AMM:

```
x * y = k

Where:
- x = shares for outcome A
- y = shares for outcome B  
- k = constant (liquidity pool)
```

**Price Calculation:**
```
Price_A = y / (x + y)
Price_B = x / (x + y)
```

## 🔐 Security

- **On-chain settlement**: All bets resolved on Solana blockchain
- **Merkle proofs**: Match results verified cryptographically
- **PDA accounts**: Program Derived Addresses for secure fund storage
- **USDC escrow**: Funds locked in PDA-controlled token accounts

## 📊 TxLINE Integration

### Data Flow
1. TxLINE streams real-time match data to the frontend via SSE
2. Users place bets by purchasing outcome shares
3. AMM adjusts prices based on supply/demand
4. After match completion, settlement triggered with Merkle proof verification
5. Winners claim payouts directly from on-chain vault

### API Endpoints
- **REST**: Historical match data
- **SSE**: Real-time score/event streaming
- **Merkle**: Cryptographic result proofs

## 🎯 Use Cases

1. **Match Winner**: Predict which team wins
2. **Over/Under**: Predict total goals
3. **Correct Score**: Predict exact final score
4. **First Goal**: Predict which team scores first

## 📈 Roadmap

- [x] Smart contract development
- [x] Frontend implementation
- [x] TxLINE integration
- [ ] Deploy to devnet (pending SOL)
- [ ] Mainnet launch
- [ ] Mobile app
- [ ] Multi-sport support

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [TxLINE](https://txline.txodds.com) for sports data API
- [Solana](https://solana.com) for blockchain infrastructure
- [Anchor](https://anchor-lang.com) for smart contract framework
- [Next.js](https://nextjs.org) for frontend framework

## 📞 Contact

- **GitHub**: [@z03009183-dot](https://github.com/z03009183-dot)
- **Project Link**: https://github.com/z03009183-dot/worldcup-predict

---

**Built with ❤️ for the World Cup Prediction Market Hackathon**
