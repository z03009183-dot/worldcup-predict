# Technical Documentation

## World Cup Prediction Market — Solana Program & System Design

---

## Table of Contents

1. [System Architecture Overview](#1-system-architecture-overview)
2. [Smart Contract Instructions](#2-smart-contract-instructions)
3. [PDA Derivation Scheme](#3-pda-derivation-scheme)
4. [AMM Formula (Constant Product)](#4-amm-formula-constant-product)
5. [TxLINE Integration Flow](#5-txline-integration-flow)
6. [On-Chain Validation with Merkle Proofs](#6-on-chain-validation-with-merkle-proofs)
7. [Security Considerations](#7-security-considerations)

---

## 1. System Architecture Overview

The prediction market consists of three main layers:

### Presentation Layer (Next.js Frontend)
- Server-rendered React pages with TailwindCSS
- Solana wallet adapter for transaction signing
- SSE client for real-time TxLINE data streaming
- Market state polling via Solana RPC subscriptions

### Application Layer (Anchor Smart Contract)
- On-chain program deployed at `Dz92ko2VQTzJKrLzm5weYtpkyuYBPWGR5ynk8WjmpsFX`
- Manages market lifecycle: creation → trading → settlement → claiming
- Implements constant product AMM for share pricing
- Verifies Merkle proofs for trustless result validation

### Data Layer (TxLINE API)
- REST API for historical match data
- SSE endpoints for real-time score/event streaming
- Merkle tree generation for cryptographic result proofs
- Free tier available for World Cup data

```
User Browser
    │
    ├──▶ [Next.js App] ◀──SSE──▶ [TxLINE API]
    │         │
    │    Sign & Send Tx
    │         │
    │         ▼
    │    [Solana RPC Node]
    │         │
    │         ▼
    │    [prediction_market.so]
    │         │
    │    ┌────┴────┐
    │    │ Program  │
    │    │ State PDAs│
    │    └─────────┘
    │
    └──▶ [Solana Explorer] (view transactions)
```

---

## 2. Smart Contract Instructions

### 2.1 `initialize`

**Purpose:** One-time setup of global program state.

**Accounts:**
| Account | Type | Description |
|---------|------|-------------|
| `global_state` | PDA (writable) | Singleton PDA storing program authority |
| `authority` | Signer | Deployer/admin who becomes program authority |
| `system_program` | Program | Solana System Program |

**Logic:**
1. Derive `global_state` PDA from seeds `[b"global_state"]`
2. Initialize account with authority pubkey
3. Set market counter to 0

**Error Conditions:**
- `AlreadyInitialized` — if `global_state` already exists

---

### 2.2 `create_market`

**Purpose:** Create a new prediction market for a specific match.

**Accounts:**
| Account | Type | Description |
|---------|------|-------------|
| `market` | PDA (writable) | New market account, seeded by match_id |
| `market_vault` | PDA (writable) | SOL vault for this market |
| `global_state` | PDA | Program global state |
| `authority` | Signer | Market creator (must be program authority) |
| `system_program` | Program | Solana System Program |

**Parameters:**
```rust
pub fn create_market(
    ctx: Context<CreateMarket>,
    match_id: String,        // Unique match identifier from TxLINE
    home_team: String,       // Home team name
    away_team: String,       // Away team name
    match_start_time: i64,   // Unix timestamp of match start
    initial_liquidity: u64,  // Initial SOL deposited as liquidity (lamports)
) -> Result<()>
```

**Logic:**
1. Derive market PDA from `match_id`
2. Initialize market account with match metadata
3. Derive vault PDA and transfer `initial_liquidity` SOL into it
4. Set AMM reserves: `yes_shares = initial_liquidity`, `no_shares = initial_liquidity`
5. Set market status to `Open`

---

### 2.3 `buy_shares`

**Purpose:** Purchase outcome shares (Yes/No) at the current AMM price.

**Accounts:**
| Account | Type | Description |
|---------|------|-------------|
| `market` | PDA (writable) | Market account to trade on |
| `market_vault` | PDA (writable) | Market's SOL vault |
| `buyer_position` | PDA (writable) | Buyer's position account for this market |
| `buyer` | Signer | User buying shares |
| `system_program` | Program | Solana System Program |

**Parameters:**
```rust
pub fn buy_shares(
    ctx: Context<BuyShares>,
    outcome: Outcome,   // Yes or No
    sol_amount: u64,    // SOL to spend (lamports)
) -> Result<()>
```

**Logic:**
1. Validate market is `Open` and match hasn't started
2. Calculate share price using AMM formula (see Section 4)
3. Transfer `sol_amount` from buyer to `market_vault`
4. Mint `shares_received` to buyer's position account
5. Update AMM reserves
6. Emit `SharesPurchased` event

**Pricing Example:**
- If 60% of liquidity is on YES, YES shares cost more (higher probability)
- Buying YES shares shifts the price curve, making YES more expensive for next buyer

---

### 2.4 `settle_market`

**Purpose:** Resolve a market by recording the match outcome on-chain.

**Accounts:**
| Account | Type | Description |
|---------|------|-------------|
| `market` | PDA (writable) | Market to settle |
| `global_state` | PDA | Program global state |
| `authority` | Signer | Program authority (settlement oracle) |

**Parameters:**
```rust
pub fn settle_market(
    ctx: Context<SettleMarket>,
    outcome: Outcome,              // Winning outcome (Yes/No)
    result_hash: [u8; 32],         // Merkle root from TxLINE
    merkle_proof: Vec<[u8; 32]>,   // Proof path
    leaf_data: Vec<u8>,            // Match result leaf node
) -> Result<()>
```

**Logic:**
1. Validate market is `Open` and match has ended (timestamp check)
2. Verify Merkle proof against `result_hash` (see Section 6)
3. Set `market.outcome = Some(outcome)` (winning outcome)
4. Set `market.status = Settled`
5. Calculate total winning shares vs total losing shares
6. Record payout ratio in market account
7. Emit `MarketSettled` event

---

### 2.5 `claim_winnings`

**Purpose:** Distribute SOL to holders of winning shares.

**Accounts:**
| Account | Type | Description |
|---------|------|-------------|
| `market` | PDA (writable) | Settled market |
| `market_vault` | PDA (writable) | Market's SOL vault |
| `buyer_position` | PDA (writable) | Claimer's position account |
| `claimer` | Signer | User claiming winnings |
| `system_program` | Program | Solana System Program |

**Parameters:**
```rust
pub fn claim_winnings(
    ctx: Context<ClaimWinnings>,
) -> Result<()>
```

**Logic:**
1. Validate market is `Settled`
2. Validate claimer holds winning shares (not already claimed)
3. Calculate payout: `winnings = (winning_shares / total_winning_shares) * vault_balance`
4. Transfer `winnings` from `market_vault` to `claimer`
5. Mark position as `Claimed`
6. Emit `WinningsClaimed` event

---

## 3. PDA Derivation Scheme

All accounts are Program Derived Addresses (PDAs), ensuring deterministic addresses and preventing key management issues.

### Global State PDA
```
seeds = [b"global_state"]
program = prediction_market
→ Single instance per deployment
```

### Market PDA
```
seeds = [b"market", match_id.as_bytes()]
program = prediction_market
→ One per unique match_id (e.g., "fifa_wc_2026_match_042")
```

### Market Vault PDA
```
seeds = [b"vault", market.key().as_ref()]
program = prediction_market
→ SOL escrow account for each market
```

### Buyer Position PDA
```
seeds = [b"position", market.key().as_ref(), buyer.key().as_ref()]
program = prediction_market
→ One per user per market, tracks shares owned
```

### PDA Diagram
```
prediction_market (Program)
│
├── GlobalState PDA
│   seeds: ["global_state"]
│
├── Market PDA ──────────── (match_id: "wc2026_brasil_vs_argentina")
│   ├── Vault PDA
│   │   seeds: ["vault", market_pubkey]
│   │
│   ├── Position PDA ────── (user: Alice)
│   │   seeds: ["position", market_pubkey, alice_pubkey]
│   │
│   └── Position PDA ────── (user: Bob)
│       seeds: ["position", market_pubkey, bob_pubkey]
│
└── Market PDA ──────────── (match_id: "wc2026_france_vs_germany")
    ├── Vault PDA
    ├── Position PDA (Alice)
    └── Position PDA (Bob)
```

---

## 4. AMM Formula (Constant Product)

The market uses a **constant product market maker** (x * y = k), similar to Uniswap v2.

### Core Invariant

```
yes_reserve * no_reserve = k (constant)
```

Where:
- `yes_reserve` = virtual reserve for YES outcome
- `no_reserve` = virtual reserve for NO outcome
- `k` = constant (set at market creation, changes only with trades)

### Price Calculation

The price of a YES share represents the implied probability:

```
p_yes = no_reserve / (yes_reserve + no_reserve)
p_no  = yes_reserve / (yes_reserve + no_reserve)
```

Note: `p_yes + p_no = 1.0` (prices always sum to 1, minus spread)

### Share Purchase Calculation

When buying `ΔS` worth of SOL for outcome `yes`:

```
new_yes_reserve = yes_reserve - shares_out
new_no_reserve  = k / new_yes_reserve
cost_in_sol     = new_no_reserve - no_reserve
```

Solving for `shares_out` given `cost_in_sol`:

```
shares_out = yes_reserve - (k / (no_reserve + cost_in_sol))
```

Where `cost_in_sol` is the SOL amount (minus protocol fee).

### Slippage

Large purchases cause price impact:
- Buying YES makes YES more expensive (and NO cheaper)
- The larger the trade relative to reserves, the worse the price

### Fee Structure

```
effective_cost = sol_amount * (1 - fee_rate)
fee_amount     = sol_amount * fee_rate
```

Default `fee_rate = 0.01` (1%), collected by the protocol.

### Example

```
Initial state: yes_reserve = 10 SOL, no_reserve = 10 SOL, k = 100
Implied probability: 50/50

User buys 2 SOL of YES shares:
  effective_cost = 2 * 0.99 = 1.98 SOL
  new_no_reserve = 10 + 1.98 = 11.98
  new_yes_reserve = 100 / 11.98 ≈ 8.347
  shares_out = 10 - 8.347 ≈ 1.653 YES shares

New implied probability: YES ≈ 58.8%, NO ≈ 41.2%
```

---

## 5. TxLINE Integration Flow

### Step 1: Authentication

```
POST https://txline.txodds.com/api/auth/token
Content-Type: application/json

{
  "api_key": "<TXLINE_API_KEY>"
}

→ Response:
{
  "access_token": "eyJ...",
  "expires_in": 3600,
  "token_type": "Bearer"
}
```

### Step 2: Subscribe to Match

```
GET https://txline.txodds.com/api/matches/{match_id}/subscribe
Authorization: Bearer <access_token>

→ Response:
{
  "subscription_id": "sub_abc123",
  "sse_url": "/api/stream/sub_abc123",
  "match_id": "wc2026_match_042"
}
```

### Step 3: SSE Stream

```
GET https://txline.txodds.com/api/stream/{subscription_id}
Accept: text/event-stream

→ SSE Events:

event: score_update
data: {"match_id":"wc2026_match_042","home":1,"away":0,"minute":23,"timestamp":1686240000}

event: odds_update
data: {"match_id":"wc2026_match_042","home_win":1.45,"draw":4.20,"away_win":6.80}

event: match_end
data: {"match_id":"wc2026_match_042","final_score":{"home":2,"away":1},"status":"finished"}
```

### Step 4: Request Result Proof

```
GET https://txline.txodds.com/api/matches/{match_id}/proof
Authorization: Bearer <access_token>

→ Response:
{
  "match_id": "wc2026_match_042",
  "result": {"home": 2, "away": 1, "winner": "home"},
  "merkle_root": "0xa1b2c3d4...",
  "merkle_proof": ["0x1111...", "0x2222...", "0x3333..."],
  "leaf_hash": "0xdeadbeef...",
  "proof_timestamp": 1686243600
}
```

### Step 5: Submit to Smart Contract

The `merkle_root`, `merkle_proof`, and `leaf_hash` are passed to the `settle_market` instruction as on-chain verification data.

---

## 6. On-Chain Validation with Merkle Proofs

### Why Merkle Proofs?

The smart contract cannot directly call the TxLINE API. Instead, the authority submits the match result along with a cryptographic proof that the data originated from TxLINE's published Merkle tree.

### Merkle Tree Structure

TxLINE maintains a Merkle tree of all match results:

```
                Root (published on-chain / IPFS)
               /                              \
          Hash(AB)                          Hash(CD)
         /        \                       /        \
    Hash(A)     Hash(B)             Hash(C)     Hash(D)
       |           |                   |           |
   Match_001   Match_002           Match_003   Match_004
```

### Proof Verification (On-Chain)

```rust
fn verify_merkle_proof(
    leaf_hash: [u8; 32],
    proof: &[[u8; 32]],
    root: [u8; 32],
    leaf_data: &[u8],
) -> bool {
    // 1. Compute leaf hash from leaf_data
    let computed_leaf = hash::sha256(leaf_data);
    require!(computed_leaf == leaf_hash, InvalidLeaf);

    // 2. Walk up the Merkle tree using proof path
    let mut current = leaf_hash;
    for sibling in proof.iter() {
        if current <= *sibling {
            current = hash::sha256(&[current, *sibling].concat());
        } else {
            current = hash::sha256(&[*sibling, current].concat());
        }
    }

    // 3. Compare with published root
    current == root
}
```

### Leaf Data Format

```json
{
  "match_id": "wc2026_match_042",
  "home_team": "Brazil",
  "away_team": "Argentina",
  "home_score": 2,
  "away_score": 1,
  "match_status": "finished",
  "timestamp": 1686243600
}
```

This JSON is serialized to bytes, SHA-256 hashed to produce the leaf, then verified against the Merkle root.

### Trust Model

- **TxLINE publishes the Merkle root** to a verifiable location (IPFS, Solana account, or their public API)
- **Anyone can verify** that a specific match result is included in the tree
- **The authority cannot forge results** — they must provide valid proofs
- **If the proof is invalid**, `settle_market` reverts

---

## 7. Security Considerations

### 7.1 Access Control

| Action | Who Can Do It |
|--------|--------------|
| `initialize` | First caller only (one-time) |
| `create_market` | Program authority only |
| `buy_shares` | Any user |
| `settle_market` | Program authority only |
| `claim_winnings` | Position holder only |

### 7.2 Authority Trust Assumption

The program authority is a trusted role that can:
- Create markets with arbitrary parameters
- Submit settlement results (but must provide valid Merkle proofs)
- Cannot steal funds from vaults (settlement doesn't move funds)
- Cannot modify user positions

**Mitigation for authority trust:**
- Merkle proofs constrain what results can be submitted
- Multi-sig authority (recommended for production)
- Future: transition to oracle-based settlement (e.g., Switchboard, Pyth)

### 7.3 Reentrancy

Solana's programming model is not susceptible to reentrancy attacks in the same way as Ethereum, because accounts must be passed explicitly and execution is serialized within a transaction.

However, the program still follows best practices:
- State is updated before transfers (checks-effects-interactions)
- All account mutations happen atomically within a single instruction

### 7.4 Integer Overflow

- All arithmetic uses checked operations (`checked_add`, `checked_sub`, `checked_mul`, `checked_div`)
- AMM calculations use `u128` intermediates to prevent overflow in multiplication
- Division by zero is explicitly guarded

### 7.5 Front-Running

- Match start time is enforced: no trades after match begins
- Settlement requires valid Merkle proof (not just authority signature)
- Consider adding commit-reveal scheme for large bets in production

### 7.6 Account Validation

All instructions validate:
- Account ownership (must be owned by the program)
- PDA derivation (seeds match expected values)
- Signer status (only authorized signers)
- Account discriminator (prevents type confusion)

### 7.7 Upgrade Authority

- The program should be deployed with an upgrade authority for bug fixes
- Consider removing upgrade authority (immutable) for production
- Use a multi-sig as upgrade authority if retained

### 7.8 Denial of Service

- Market creation requires `initial_liquidity > 0` to prevent spam
- Position accounts are PDAs (no account creation race conditions)
- Compute unit limits prevent infinite loops

---

## Appendix: Account Sizes

| Account | Size (bytes) | Description |
|---------|-------------|-------------|
| `GlobalState` | 48 | Authority (32) + counter (8) + discriminator (8) |
| `Market` | 512 | Match data, reserves, status, outcome, metadata |
| `MarketVault` | 0 | Owned by PDA, holds SOL (rent-exempt minimum) |
| `BuyerPosition` | 128 | Shares (yes/no), claimed status, metadata |

---

## Appendix: Error Codes

| Code | Name | Description |
|------|------|-------------|
| 6000 | `MarketNotOpen` | Market is not in Open status |
| 6001 | `MarketNotSettled` | Market has not been settled yet |
| 6002 | `MatchAlreadyStarted` | Cannot trade after match begins |
| 6003 | `MatchNotEnded` | Cannot settle before match ends |
| 6004 | `InvalidMerkleProof` | Merkle proof verification failed |
| 6005 | `AlreadyClaimed` | User already claimed winnings |
| 6006 | `InsufficientShares` | User has no winning shares |
| 6007 | `AlreadyInitialized` | Program already initialized |
| 6008 | `InvalidOutcome` | Outcome does not match winning side |
| 6009 | `ArithmeticOverflow` | Calculation overflow |
