# Game Design: Hyperliquid Liar's Dice

## Core Concept

**Liar's Dice meets Perpetual Futures Trading**

Every bid you make isn't just a claim about dice—it's a real leveraged position on Hyperliquid that can liquidate you. Bluffing becomes financially consequential. Reading opponents means reading both their psychology AND their risk tolerance.

## Traditional Liar's Dice (Baseline)

### Rules
1. Each player has 5 dice under a cup (hidden from others)
2. Players take turns bidding on the TOTAL number of dice showing a specific face across ALL cups
3. Example: "Three 4s" means "I claim there are at least three dice showing 4 across all players' cups"
4. Each bid must be higher than the last:
   - Increase quantity: "Three 4s" → "Four 4s"
   - Increase face value (same quantity): "Three 4s" → "Three 5s"
   - Increase both: "Three 4s" → "Four 5s"
5. On your turn, you can either:
   - **Raise:** Make a higher bid
   - **Challenge:** Call BS on the previous bid
6. When challenged, all dice are revealed:
   - If the bid was TRUE (enough dice exist), challenger loses a die
   - If the bid was FALSE (not enough dice), bidder loses a die
7. Players eliminated when they lose all dice
8. Last player standing wins

### Key Skills
- **Probability:** Calculate odds based on your dice
- **Psychology:** Read opponents' confidence
- **Bluffing:** Make bids you can't support
- **Risk management:** Know when to fold

## Hyperliquid Integration (The Innovation)

### Bid → Position Mapping

Every bid opens a real perpetual futures position on Hyperliquid:

**Mapping System:**

| Die Face | Asset | Rationale |
|----------|-------|-----------|
| 1 | BTC | #1 crypto |
| 2 | ETH | #2 crypto |
| 3 | SOL | High volatility |
| 4 | ARB | L2 (medium risk) |
| 5 | AVAX | Alt (higher risk) |
| 6 | OP | Alt (highest risk) |

**Position Direction:**

- **Raising (increase quantity or face)** = **LONG** position
  - "Three 4s" → "Four 4s" = 4x ARB long
  - "Three 4s" → "Three 5s" = 3x AVAX long
  
- **Matching/Playing safe** = **SHORT** position
  - Previous bid "Three 4s", you bid "Three 5s" (minimal raise) = 3x AVAX short

**Position Size:**

- **Base:** $100 per die claimed (configurable)
- **Leverage:** 10x (can be adjusted per table)
- **Example:** "Four 4s" bid = $400 * 10x = $4,000 ARB position

**When Positions Open/Close:**

1. **Position Opens:** When you make a bid
2. **Position Stays Open:** Until you're challenged OR game ends
3. **Position Closes:** 
   - Someone challenges your bid
   - You win the round
   - You get eliminated
   - 1-hour timeout (auto-close with current P&L)

### Economic Dynamics

**Scenario 1: Accurate Bid + Good Trade**
```
You bid: "Three 4s" (3x ARB long @ $1.20)
Reality: Four 4s exist (you were RIGHT)
Market: ARB pumps to $1.25 (+4.2%)

Result:
- Win dice round ✅
- Trading profit: $420 (+4.2% on $10k position)
- Total gain: Pot + $420
```

**Scenario 2: Bluff + Lucky Trade**
```
You bid: "Five 6s" (5x OP long @ $2.50)
Reality: Only three 6s exist (you were WRONG)
Market: OP pumps to $2.65 (+6%)

Result:
- Lose dice round ❌ (lose 1 die)
- Trading profit: $750 (+6% on $12.5k position)
- Total: -1 die but +$750

You're eliminated from dice but made money!
```

**Scenario 3: Good Read + Bad Trade**
```
You challenge opponent's "Four 5s" bid
Reality: Only three 5s exist (they were WRONG, you win!)
Market: AVAX dumped -8% while position was open

Result:
- Win dice challenge ✅ (opponent loses die)
- Their trading loss: -$800
- Your trading gain: $0 (you didn't have position)

They lose dice AND money.
```

**Scenario 4: Liquidation**
```
You bid: "Six 3s" (6x SOL long @ $100)
Position size: $6,000 * 10x = $60,000
Reality: Actually only five 3s
Market: SOL dumps -12%

Result:
- Liquidated ⚠️ (position closed, collateral lost)
- Auto-lose the dice round (forced elimination)
- Lose: $6,000 (your collateral)

Catastrophic failure.
```

### Game Modes

#### Mode 1: Classic (Low Stakes)
- Entry: $10
- Position size: 10x entry ($100 per position)
- Leverage: 5x
- Target: Casual players, learning

#### Mode 2: Degen (High Stakes)
- Entry: $100
- Position size: 10x entry ($1,000 per position)
- Leverage: 20x
- Target: Risk-seekers, whales

#### Mode 3: Tournament (Competitive)
- Entry: $50
- Position size: 20x entry ($1,000 per position)
- Leverage: 10x
- Prize pool: Winner-takes-all
- Target: Competitive players

### New Strategic Dimensions

**Traditional Liar's Dice Strategy:**
- Bluff when you have weak dice
- Challenge when probability is low
- Play conservatively to survive

**Hyperliquid Liar's Dice Strategy:**
- Bluff when market conditions favor your position
- Challenge based on BOTH dice probability AND market direction
- Aggressive play if you have edge on market timing
- Conservative if market is choppy/uncertain
- Can intentionally lose dice rounds to profit from positions

**Example Strategic Play:**

Scenario: BTC just broke resistance, momentum strong
```
You have: [3, 3, 5, 5, 6]
Current bid: "Two 1s" (2x BTC)

Traditional strategy: Bid "Three 3s" (you have two)
Hyperliquid strategy: Bid "Four 1s" (BTC long)

Why? 
- You're bluffing (probably not four 1s)
- But BTC momentum → position likely profitable
- If challenged and lose, you still profit from trade
- Forces opponent into awkward position:
  - Challenge = expose you but miss BTC pump
  - Raise = they open bigger position at worse entry
```

### Risk Management Rules

**Position Limits:**
- Max position per bid: $10,000 (configurable)
- Max total exposure per player: $50,000
- Auto-liquidation triggers forced elimination

**Time Limits:**
- 30 seconds to make bid
- 1-hour max position duration (auto-close)
- 5-minute round timer

**Safety Rails:**
- Can't bid if insufficient collateral
- Warning before liquidation risk (< 5% margin)
- Can forfeit round to close positions early

## Win Conditions

### Primary Win: Last Player Standing
- Traditional liar's dice victory
- Keep all your dice while others lose theirs
- Prize: **Entire pot + your trading P&L**

### Secondary Win: Best Trader
- If all players eliminated (simultaneous liquidations)
- Player with highest net P&L wins
- Prize: **Consolation (20% of pot) + your trading P&L**

### Bonus: Perfect Game
- Win without losing a single die
- All positions profitable
- Prize: **Pot + P&L + 50% bonus**

## Technical Requirements

### On-Chain (Smart Contract)
- Provably random dice rolls (VRF)
- Bid validation
- Challenge resolution
- Position tracking (references, not execution)
- Pot management

### Off-Chain (Backend + Hyperliquid)
- Real-time bid management
- Hyperliquid API integration
- Position execution
- P&L tracking
- Liquidation monitoring

### Frontend
- 3D dice visualization
- Position display (real-time P&L)
- Market data feeds
- Probability calculator
- Social features (chat, emotes)

## Edge Cases & Rules

### Liquidation
- If any position liquidates, player is immediately eliminated
- Liquidated collateral goes to pot
- Remaining players continue

### Disconnection
- 30-second timeout
- Auto-fold if disconnected
- Positions remain open (risky!)

### Simultaneous Liquidation
- All remaining players liquidated at once
- Game ends
- Pot split proportionally by survival time

### Market Closure
- If Hyperliquid market closes mid-game
- That asset's bids become "classic" (no position)
- Game continues

### Ties
- Multiple players eliminated simultaneously
- Check trading P&L as tiebreaker
- If still tied, split pot

## Progression & Meta

### Leaderboards
- Most pots won
- Highest net P&L
- Best bluff success rate
- Most liquidations caused
- Perfect games

### Achievements
- First blood (eliminate first player)
- Degenerate (win with all liquidations)
- Prophet (win with perfect probability calls)
- Market wizard (win with 90%+ profitable positions)
- Survivor (outlast 5+ eliminations)

### Skill Rating (ELO-style)
- Factors:
  - Win rate
  - Average pot size
  - Trading P&L consistency
  - Bluff success rate

## Future Expansions

### Wild Cards
- Special dice faces (jokers, doubles)
- Affect position mechanics

### Power-Ups
- Use trading profits to buy:
  - Extra die
  - Position size boost
  - Liquidation insurance
  - Market data reveals

### Team Mode
- 2v2 or 3v3
- Shared pot, individual positions
- Can coordinate bids

### AI Opponents
- Trained on game data
- Different risk profiles
- Practice mode

## Balancing Considerations

**Too Random?**
- Dice are still central (70% of outcome)
- Trading adds skill ceiling, not luck
- Good players can exploit market edge

**Too Complex?**
- Tutorial mode (classic dice first)
- AI hints for beginners
- Simplified "auto-trade" mode

**Too Risky?**
- Start with low-stakes tables
- Optional insurance (costs % of pot)
- Conservative default leverage (5x)

## Why This Works

1. **Familiar Core:** Everyone knows liar's dice
2. **Novel Twist:** Positions add financial stakes
3. **Skill Expression:** Both psychological and technical
4. **Viral Potential:** "I won $500 by lying" stories
5. **Sustainable:** House cut + trading fees
6. **Composable:** Can add more mechanics later

## Design Principles

- **Clarity:** Easy to understand what happens
- **Feedback:** Real-time P&L display
- **Risk/Reward:** Clear tradeoffs
- **Skill Ceiling:** Deep strategy for pros
- **Fun First:** Even losing can be entertaining
- **No Grinding:** Games are fast (5-10 min)

---

**Next:** See [ARCHITECTURE.md](ARCHITECTURE.md) for technical implementation.
