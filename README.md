# Hyperliquid Liar's Dice 🎲⚡

**On-chain Liar's Dice where every bid is a real Hyperliquid perpetual position.**

Bluff with money on the line. Challenge calls with real trades. Win by reading the table AND the market.

## The Concept

Traditional liar's dice meets perpetual futures trading. Each bid you make opens a real position on Hyperliquid. When someone challenges, positions close and P&L determines the winner.

**Core Innovation:** Your bluffs aren't just words—they're leveraged positions that can liquidate you.

## Game Mechanics

### Traditional Liar's Dice Rules

1. Each player has 5 dice under a cup (hidden)
2. Players bid on total dice across all cups
3. Bids must increase (quantity or face value)
4. Challenge the previous bid or raise
5. Reveal dice - if bid was true, challenger loses; if false, bidder loses

### Hyperliquid Integration

**Every bid = A real perpetual position**

- **Bid "Three 4s"** → Open 3x ETH long @ current price
- **Bid "Four 5s"** → Open 4x BTC short @ current price  
- **Challenge** → Force close all positions, P&L settles the round

**Position Mapping:**
- Face value (1-6) → Asset (BTC, ETH, SOL, ARB, AVAX, OP)
- Quantity → Position size multiplier
- Bid direction (raise vs match) → Long vs Short

**Financial Dynamics:**
- Good bluff + market moves your way = double win
- Bad bluff + market moves against = double loss
- Accurate read + good timing = profit
- Wrong call + bad entry = liquidation

## Quick Start

```bash
# Clone
git clone https://github.com/unju-ai/hyperliquid-liars-dice.git
cd hyperliquid-liars-dice

# Install
bun install

# Run locally (testnet)
bun dev

# Open http://localhost:3000
```

## Repository Structure

```
hyperliquid-liars-dice/
├── docs/                   # Game design & architecture
│   ├── GAME_DESIGN.md     # Full mechanics
│   ├── ARCHITECTURE.md    # Technical architecture
│   └── ECONOMICS.md       # Token economics & payouts
├── contracts/             # Solidity smart contracts
│   ├── src/
│   │   ├── LiarsDice.sol  # Core game logic
│   │   ├── DiceVRF.sol    # Verifiable randomness
│   │   └── HLIntegration.sol # Hyperliquid interface
│   └── test/
├── frontend/              # Next.js + React Three Fiber
│   ├── app/
│   ├── components/
│   └── lib/
├── backend/               # Bun + WebSocket server
│   ├── src/
│   │   ├── game/          # Game state management
│   │   ├── hyperliquid/   # Trading integration
│   │   └── ws/            # Real-time multiplayer
│   └── prisma/
└── README.md
```

## Features

### Phase 1: MVP (2 weeks)
- [x] Game design document
- [ ] Smart contracts (game logic + VRF)
- [ ] Frontend (3D dice + UI)
- [ ] Backend (WebSocket + matchmaking)
- [ ] Hyperliquid testnet integration
- [ ] 2-player games

### Phase 2: Full Release (4 weeks)
- [ ] Multi-player (up to 6 players)
- [ ] Leaderboards & stats
- [ ] Custom game modes
- [ ] Mobile responsive
- [ ] Mainnet deployment
- [ ] Audits (contracts + security)

### Phase 3: Advanced (8 weeks)
- [ ] Tournaments
- [ ] Spectator mode
- [ ] Replay system
- [ ] AI opponents (trained on game data)
- [ ] Advanced trading strategies
- [ ] Social features

## Stack

**Smart Contracts:**
- Solidity 0.8.20+
- Foundry (testing & deployment)
- Chainlink VRF (randomness)
- Hyperliquid SDK

**Frontend:**
- Next.js 14 (App Router)
- React Three Fiber (3D dice)
- wagmi + viem (wallet connection)
- TailwindCSS + shadcn/ui

**Backend:**
- Bun runtime
- WebSocket (real-time)
- PostgreSQL + Prisma
- Hyperliquid API

## How to Play

1. **Join Game** - Connect wallet, find table
2. **Ante Up** - Deposit collateral (starts at $10)
3. **Roll Dice** - Hidden roll (VRF on-chain)
4. **Make Bids** - Each bid opens a Hyperliquid position
5. **Challenge** - Call BS and force position close
6. **Settlement** - Winner takes both: dice round + trading profits

## Economics

**Entry Fee:** $10-$100 (player choice)  
**House Cut:** 2% of pot  
**Position Size:** 10x entry fee (leveraged)  
**Liquidation:** If position liquidates, you auto-lose round

**Example Round:**
```
Entry: $50
Position size: $500 (10x)

Player A bids "Three 4s" (3x BTC long @ $50k)
Player B challenges

Outcome:
- Dice reveal: Only two 4s → Player A lied
- BTC pumped to $51k → Player A's position +$30
- Result: Player B wins dice round but Player A keeps trading profit
- Settlement: Player B gets $50 pot, Player A gets $30 P&L

Net: Player B +$50, Player A -$20
```

## Contributing

This is an experiment in game theory meets DeFi. Contributions welcome!

1. Fork the repo
2. Create feature branch (`git checkout -b feature/amazing`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push branch (`git push origin feature/amazing`)
5. Open Pull Request

## License

MIT

## Disclaimer

**This is gambling.** You can lose money. Positions can liquidate. Markets are volatile. Trade responsibly.

Not financial advice. Use at your own risk. Built for educational purposes and degenerate fun.

## Links

- [Game Design Doc](docs/GAME_DESIGN.md)
- [Architecture](docs/ARCHITECTURE.md)
- [Hyperliquid](https://hyperliquid.xyz)
- [unju.ai](https://unju.ai)

---

**Built with 🎲 by [unju.ai](https://github.com/unju-ai)**
