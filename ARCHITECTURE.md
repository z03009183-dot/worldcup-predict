# System Architecture

## World Cup Prediction Market — Architecture Overview

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              USER BROWSER                                    │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                      Next.js Frontend (SSR)                          │    │
│  │                                                                       │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐   │    │
│  │  │ Market List   │  │ Match Detail │  │ Share Purchase Widget    │   │    │
│  │  │ Component     │  │ Component    │  │ (Buy YES / Buy NO)       │   │    │
│  │  └──────────────┘  └──────────────┘  └──────────────────────────┘   │    │
│  │                                                                       │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐   │    │
│  │  │ Live Scores   │  │ Position     │  │ Settlement & Claim       │   │    │
│  │  │ (SSE Stream)  │  │ Tracker      │  │ Widget                   │   │    │
│  │  └──────┬───────┘  └──────────────┘  └──────────────────────────┘   │    │
│  │         │                                                             │    │
│  └─────────┼─────────────────────────────────────────────────────────────┘    │
│            │                                                                  │
└────────────┼──────────────────────────────────────────────────────────────────┘
             │ SSE (Server-Sent Events)
             │
┌────────────▼──────────────────────────────────────────────────────────────────┐
│                          TxLINE API Server                                     │
│                    https://txline.txodds.com                                    │
│                                                                                │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────────────────┐  │
│  │ Auth       │  │ Match Data │  │ SSE Stream │  │ Merkle Proof Service   │  │
│  │ Endpoint   │  │ Endpoint   │  │ Endpoint   │  │ (Result Verification)  │  │
│  └────────────┘  └────────────┘  └────────────┘  └────────────────────────┘  │
│                                                                                │
└────────────────────────────────────────────────────────────────────────────────┘
             │
             │ Transaction (signed by user wallet)
             │
┌────────────▼──────────────────────────────────────────────────────────────────┐
│                         SOLANA BLOCKCHAIN (Devnet)                             │
│                                                                                │
│  ┌────────────────────────────────────────────────────────────────────────┐   │
│  │              prediction_market.so (Anchor Program)                     │   │
│  │           Program ID: Dz92ko2VQTzJKrLzm5weYtpkyuYBPWGR5ynk8WjmpsFX    │   │
│  │                                                                        │   │
│  │  ┌────────────────────────────────────────────────────────────────┐   │   │
│  │  │                     INSTRUCTIONS                                │   │   │
│  │  │                                                                  │   │   │
│  │  │  ┌─────────────┐  ┌──────────────┐  ┌───────────────────────┐  │   │   │
│  │  │  │ initialize  │  │ create_market│  │ buy_shares            │  │   │   │
│  │  │  │             │  │              │  │ (AMM pricing)         │  │   │   │
│  │  │  └─────────────┘  └──────────────┘  └───────────────────────┘  │   │   │
│  │  │                                                                  │   │   │
│  │  │  ┌─────────────────┐  ┌────────────────────────────────────┐   │   │   │
│  │  │  │ settle_market   │  │ claim_winnings                     │   │   │   │
│  │  │  │ (Merkle verify) │  │ (SOL payout to winners)            │   │   │   │
│  │  │  └─────────────────┘  └────────────────────────────────────┘   │   │   │
│  │  │                                                                  │   │   │
│  │  └────────────────────────────────────────────────────────────────┘   │   │
│  │                                                                        │   │
│  │  ┌────────────────────────────────────────────────────────────────┐   │   │
│  │  │                     PDA ACCOUNTS                                │   │   │
│  │  │                                                                  │   │   │
│  │  │  ┌──────────────┐                                              │   │   │
│  │  │  │ GlobalState   │ seeds: ["global_state"]                      │   │   │
│  │  │  │ (authority,   │                                              │   │   │
│  │  │  │  counter)     │                                              │   │   │
│  │  │  └──────────────┘                                              │   │   │
│  │  │                                                                  │   │   │
│  │  │  ┌──────────────┐  ┌──────────────┐                            │   │   │
│  │  │  │ Market #1     │  │ Market #2     │  ...                      │   │   │
│  │  │  │ (BRA vs ARG)  │  │ (FRA vs GER)  │                           │   │   │
│  │  │  │               │  │               │                            │   │   │
│  │  │  │ ┌───────────┐│  │ ┌───────────┐│                            │   │   │
│  │  │  │ │ Vault     ││  │ │ Vault     ││  seeds: ["vault", market]  │   │   │
│  │  │  │ │ (SOL)     ││  │ │ (SOL)     ││                            │   │   │
│  │  │  │ └───────────┘│  │ └───────────┘│                            │   │   │
│  │  │  │               │  │               │                            │   │   │
│  │  │  │ ┌───────────┐│  │ ┌───────────┐│                            │   │   │
│  │  │  │ │Position A ││  │ │Position A ││  seeds: [pos, market,user] │   │   │
│  │  │  │ │Position B ││  │ │Position B ││                            │   │   │
│  │  │  │ │Position C ││  │ │Position C ││                            │   │   │
│  │  │  │ └───────────┘│  │ └───────────┘│                            │   │   │
│  │  │  └──────────────┘  └──────────────┘                            │   │   │
│  │  └────────────────────────────────────────────────────────────────┘   │   │
│  └────────────────────────────────────────────────────────────────────────┘   │
│                                                                                │
└────────────────────────────────────────────────────────────────────────────────┘
```

---

## Data Flow: Buying Shares

```
User clicks "Buy YES" (2 SOL)
    │
    ▼
Frontend builds transaction
    │  - Calls buy_shares(Outcome::Yes, 2_000_000_000)
    │  - Attaches user's wallet as signer
    │  - Derives PDAs for market, vault, position
    │
    ▼
User signs with Solana wallet (Phantom/Solflare)
    │
    ▼
Transaction submitted to Solana RPC
    │
    ▼
Anchor program executes:
    │  1. Validate market is Open
    │  2. Check match hasn't started
    │  3. Compute AMM price: shares = yes_reserve - (k / (no_reserve + sol))
    │  4. Transfer SOL: user → market_vault
    │  5. Update position: user's YES shares += shares_out
    │  6. Update reserves: yes_reserve -= shares_out, no_reserve += sol
    │  7. Emit SharesPurchased event
    │
    ▼
Transaction confirmed (< 1 second on Solana)
    │
    ▼
Frontend updates UI with new position
```

---

## Data Flow: Settlement & Claim

```
Match ends (TxLINE SSE: match_end event)
    │
    ▼
Frontend detects match conclusion
    │
    ▼
Frontend requests Merkle proof from TxLINE API
    │  GET /api/matches/{match_id}/proof
    │
    ▼
Authority submits settle_market instruction
    │  - outcome: Winner (Yes/No)
    │  - result_hash: Merkle root
    │  - merkle_proof: Proof path
    │  - leaf_data: Match result JSON
    │
    ▼
Anchor program executes:
    │  1. Verify Merkle proof on-chain ✓
    │  2. Set market.status = Settled
    │  3. Record winning outcome
    │  4. Calculate payout ratio
    │
    ▼
Winners call claim_winnings instruction
    │
    ▼
Anchor program executes:
    │  1. Verify user holds winning shares
    │  2. Calculate payout: (user_shares / total_winning) * vault_balance
    │  3. Transfer SOL: market_vault → user wallet
    │  4. Mark position as claimed
    │
    ▼
SOL received in user's wallet ✓
```

---

## Technology Stack Summary

```
┌─────────────────────────────────────────────────────────────┐
│                        LAYER STACK                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Presentation    Next.js 14 · React · TailwindCSS    │   │
│  ├──────────────────────────────────────────────────────┤   │
│  │  Wallet          @solana/wallet-adapter               │   │
│  ├──────────────────────────────────────────────────────┤   │
│  │  Client SDK      @coral-xyz/anchor · @solana/web3.js  │   │
│  ├──────────────────────────────────────────────────────┤   │
│  │  Blockchain      Solana (Devnet)                       │   │
│  ├──────────────────────────────────────────────────────┤   │
│  │  Smart Contract  Anchor Framework · Rust               │   │
│  ├──────────────────────────────────────────────────────┤   │
│  │  Data Provider   TxLINE API · SSE · REST               │   │
│  ├──────────────────────────────────────────────────────┤   │
│  │  Verification    SHA-256 Merkle Proofs                  │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Deployment Topology

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Vercel / CDN   │     │  Solana Devnet   │     │   TxLINE API    │
│                  │     │                  │     │                  │
│  Next.js App     │     │  prediction_     │     │  Match Data      │
│  (SSR + Static)  │     │  market.so       │     │  SSE Streams     │
│                  │     │                  │     │  Merkle Proofs   │
│  port: 443       │     │  RPC: 443        │     │  port: 443       │
└────────┬─────────┘     └────────┬─────────┘     └────────┬─────────┘
         │                        │                         │
         └────────────────────────┼─────────────────────────┘
                                  │
                          ┌───────▼───────┐
                          │   User's      │
                          │   Browser     │
                          │   + Wallet    │
                          └───────────────┘
```
