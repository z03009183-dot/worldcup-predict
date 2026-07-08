# Demo Video Script - World Cup Prediction Market

## Duration: 5 minutes

## Opening (0:00 - 0:30)

**[Screen: Project Title]**

"Hi, I'm Zelan Albani, and today I'll show you the World Cup Prediction Market - a fully decentralized prediction market platform built on Solana."

**[Screen: Architecture Diagram]**

"This project integrates TxLINE real-time sports data with Solana smart contracts to create a trustless betting platform for World Cup matches."

## Part 1: Smart Contract (0:30 - 1:30)

**[Screen: Code Editor - lib.rs]**

"The smart contract is built with Anchor framework and has 5 main instructions:"

1. **Initialize** - Sets up the protocol state
2. **Create Market** - Creates prediction markets for matches
3. **Buy Shares** - Uses constant product AMM for pricing
4. **Settle Market** - Resolves markets using TxLINE data
5. **Claim Winnings** - Distributes USDC to winners

**[Screen: AMM Formula]**

"We use a constant product AMM where x * y = k, ensuring fair pricing based on supply and demand."

## Part 2: Frontend Demo (1:30 - 3:00)

**[Screen: Home Page]**

"The frontend is built with Next.js 14 and TailwindCSS, featuring a dark theme optimized for sports betting."

**[Screen: Markets Page]**

"Here you can browse all active markets, see live scores via TxLINE SSE streaming, and check current odds."

**[Screen: Market Detail]**

"Clicking on a market shows detailed information, live updates, and the option to buy shares."

**[Screen: Buy Shares Demo]**

"Let me demonstrate buying shares. I'll select 'Brazil to Win' and purchase 100 shares. The AMM automatically adjusts the price based on current demand."

**[Screen: Portfolio Page]**

"The portfolio page tracks all your positions, showing potential winnings and current market value."

## Part 3: TxLINE Integration (3:00 - 4:00)

**[Screen: SSE Connection]**

"TxLINE provides real-time match data through Server-Sent Events. Watch as scores update live without refreshing the page."

**[Screen: Merkle Proof]**

"For settlement, we use TxLINE's Merkle proof API to cryptographically verify match results on-chain."

## Part 4: Security & Trustlessness (4:00 - 4:30)

**[Screen: PDA Accounts]**

"All funds are stored in Program Derived Addresses, controlled by the smart contract - no central authority."

**[Screen: On-chain Settlement]**

"Settlement happens entirely on-chain using TxLINE's cryptographic proofs, ensuring fair and transparent results."

## Closing (4:30 - 5:00)

**[Screen: Summary]**

"To summarize, we've built a fully decentralized prediction market that:"

- Uses TxLINE for real-time data
- Implements AMM for fair pricing
- Settles trustlessly on Solana
- Provides instant finality

**[Screen: Links]**

"Check out the code on GitHub and try the live demo. Thank you!"

## Recording Checklist

- [ ] Screen recording software ready (OBS Studio)
- [ ] Browser with demo app running
- [ ] Code editor with smart contract open
- [ ] Terminal for commands
- [ ] Microphone tested
- [ ] Backup recording in case of issues

## Tips for Recording

1. **Speak clearly** and at a moderate pace
2. **Highlight key features** as you demonstrate them
3. **Show real interactions** - don't just talk, demo!
4. **Keep it concise** - 5 minutes max
5. **End with a call to action** - check GitHub, try demo

## Backup Plan

If live demo fails:
1. Use pre-recorded clips
2. Show screenshots instead
3. Focus on code walkthrough
4. Emphasize architecture and design decisions
