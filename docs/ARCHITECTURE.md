# Architecture: Hyperliquid Liar's Dice

## System Overview

```
┌─────────────────────────────────────────────────────────┐
│                    Frontend (Next.js)                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  3D Dice     │  │  Game UI     │  │  Positions   │  │
│  │  (Three.js)  │  │  (React)     │  │  (Real-time) │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└───────────┬──────────────────┬──────────────────┬───────┘
            │                  │                  │
            │ WebSocket        │ Wallet           │ Market Data
            │                  │ (wagmi)          │
┌───────────▼──────────────────▼──────────────────▼───────┐
│                  Backend (Bun + WS)                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  Game State  │  │  Matchmaking │  │  Hyperliquid │  │
│  │  Manager     │  │  & Rooms     │  │  Integration │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└───────────┬──────────────────┬──────────────────┬───────┘
            │                  │                  │
            │ DB               │ Smart Contract   │ Trading API
            │                  │                  │
┌───────────▼──────────┐  ┌────▼─────────┐  ┌───▼──────────┐
│   PostgreSQL         │  │  Smart       │  │  Hyperliquid │
│   + Prisma           │  │  Contracts   │  │  Exchange    │
│                      │  │  (Arbitrum)  │  │              │
└──────────────────────┘  └──────────────┘  └──────────────┘
```

## Smart Contracts (On-Chain)

### 1. LiarsDice.sol (Core Game Logic)

**Purpose:** Handle game state, bids, challenges, dice rolls

```solidity
contract LiarsDice {
    struct Game {
        address[] players;
        uint256 pot;
        uint256 ante;
        GameState state;
        uint256 currentPlayer;
        Bid lastBid;
    }
    
    struct Player {
        address wallet;
        uint8 diceCount;
        bytes32 diceCommit; // Commitment for hidden dice
        uint8[5] diceValues; // Revealed after challenge
    }
    
    struct Bid {
        uint8 quantity;
        uint8 faceValue;
        address bidder;
        uint256 timestamp;
    }
    
    // Core functions
    function createGame(uint256 ante, uint8 maxPlayers) external returns (uint256 gameId);
    function joinGame(uint256 gameId) external payable;
    function commitDice(uint256 gameId, bytes32 commitment) external;
    function makeBid(uint256 gameId, uint8 quantity, uint8 faceValue) external;
    function challenge(uint256 gameId) external;
    function revealDice(uint256 gameId, uint8[5] memory dice, bytes32 salt) external;
    function resolveCh allenge(uint256 gameId) external;
    function claimWinnings(uint256 gameId) external;
}
```

**Key Features:**
- Commit-reveal scheme for dice (prevent front-running)
- Bid validation (must be higher than previous)
- Challenge resolution (count dice across all players)
- Pot management (entry fees + house cut)
- Time-based forfeit (30s timeout per turn)

### 2. DiceVRF.sol (Verifiable Randomness)

**Purpose:** Generate provably random dice using Chainlink VRF

```solidity
contract DiceVRF {
    // Chainlink VRF integration
    function requestRandomDice(address player) external returns (uint256 requestId);
    function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override;
    function getDiceFromRandom(uint256 randomness) internal pure returns (uint8[5] memory);
}
```

**Alternative:** If VRF gas costs too high, use commit-reveal with block hash:
```solidity
function commitDice(bytes32 hash) external; // hash = keccak256(dice + salt)
function revealDice(uint8[5] dice, bytes32 salt) external; // Verify hash matches
```

### 3. HLPositionTracker.sol (Position References)

**Purpose:** Track Hyperliquid position metadata on-chain

```solidity
contract HLPositionTracker {
    struct Position {
        address player;
        uint256 gameId;
        uint8 bidQuantity;
        uint8 bidFaceValue;
        bool isLong;
        uint256 notionalSize;
        uint256 openedAt;
        bool closed;
    }
    
    mapping(uint256 => Position) public positions;
    mapping(address => uint256[]) public playerPositions;
    
    // Called by backend when position opened
    function recordPositionOpened(
        address player,
        uint256 gameId,
        uint8 quantity,
        uint8 faceValue,
        bool isLong,
        uint256 size
    ) external onlyBackend returns (uint256 positionId);
    
    // Called when position closed (challenge or timeout)
    function recordPositionClosed(uint256 positionId, int256 pnl) external onlyBackend;
}
```

**Note:** Actual Hyperliquid trading happens off-chain via API. Contract only tracks metadata for provenance.

## Backend (Bun + WebSocket)

### Architecture

```
backend/
├── src/
│   ├── server.ts              # Main server entry
│   ├── ws/
│   │   ├── GameSocket.ts      # WebSocket handler
│   │   └── RoomManager.ts     # Room/matchmaking
│   ├── game/
│   │   ├── GameState.ts       # State machine
│   │   ├── BidValidator.ts    # Bid logic
│   │   └── DiceResolver.ts    # Challenge resolution
│   ├── hyperliquid/
│   │   ├── HLClient.ts        # API client
│   │   ├── PositionManager.ts # Position lifecycle
│   │   └── PnLTracker.ts      # Real-time P&L
│   ├── db/
│   │   └── prisma.ts          # Database client
│   └── types/
│       └── game.ts            # Shared types
├── prisma/
│   └── schema.prisma          # Database schema
└── package.json
```

### Key Components

#### 1. GameState.ts (State Machine)

```typescript
export class GameState {
  id: string;
  players: Player[];
  currentPlayer: number;
  lastBid: Bid | null;
  pot: number;
  phase: 'WAITING' | 'ROLLING' | 'BIDDING' | 'CHALLENGING' | 'FINISHED';
  
  constructor(gameId: string, ante: number) {
    this.id = gameId;
    this.pot = 0;
    this.phase = 'WAITING';
  }
  
  addPlayer(player: Player): void;
  startGame(): void;
  makeBid(player: Address, quantity: number, face: number): void;
  challenge(challenger: Address): Promise<ChallengeResult>;
  eliminatePlayer(player: Address): void;
  endGame(): WinResult;
}
```

#### 2. HLClient.ts (Hyperliquid Integration)

```typescript
export class HyperliquidClient {
  private sdk: HyperliquidSDK;
  
  // Open position based on bid
  async openPosition(
    player: Address,
    bid: Bid,
    leverage: number
  ): Promise<Position> {
    const asset = FACE_TO_ASSET[bid.faceValue];
    const size = bid.quantity * 100; // $100 per die
    const isLong = this.determineDirection(bid);
    
    const result = await this.sdk.placeOrder({
      asset,
      is_buy: isLong,
      sz: size,
      limit_px: undefined, // Market order
      reduce_only: false
    });
    
    return {
      id: result.oid,
      player,
      asset,
      size,
      isLong,
      entryPrice: result.px,
      openedAt: Date.now()
    };
  }
  
  // Close position (on challenge or timeout)
  async closePosition(positionId: string): Promise<PnL> {
    const result = await this.sdk.cancelAndPlace({
      oid: positionId,
      sz: 0 // Close entire position
    });
    
    return {
      realized: result.pnl,
      fees: result.fees
    };
  }
  
  // Get real-time P&L
  async getPositionPnL(positionId: string): Promise<number> {
    const position = await this.sdk.getPositionInfo(positionId);
    return position.unrealizedPnl;
  }
}
```

#### 3. WebSocket Protocol

**Client → Server Messages:**

```typescript
type ClientMessage =
  | { type: 'JOIN_GAME'; gameId: string }
  | { type: 'BID'; quantity: number; faceValue: number }
  | { type: 'CHALLENGE' }
  | { type: 'REVEAL_DICE'; dice: number[]; salt: string };
```

**Server → Client Messages:**

```typescript
type ServerMessage =
  | { type: 'GAME_STATE'; state: GameState }
  | { type: 'PLAYER_JOINED'; player: Player }
  | { type: 'BID_MADE'; bid: Bid; position: Position }
  | { type: 'CHALLENGE_STARTED' }
  | { type: 'CHALLENGE_RESOLVED'; winner: Address; loser: Address }
  | { type: 'POSITION_PNL'; positionId: string; pnl: number }
  | { type: 'PLAYER_ELIMINATED'; player: Address }
  | { type: 'GAME_ENDED'; winner: Address; pot: number };
```

### Database Schema (Prisma)

```prisma
model Game {
  id          String   @id @default(cuid())
  ante        Int
  pot         Int
  maxPlayers  Int
  status      GameStatus
  createdAt   DateTime @default(now())
  endedAt     DateTime?
  winnerId    String?
  
  players     GamePlayer[]
  bids        Bid[]
  positions   Position[]
}

enum GameStatus {
  WAITING
  ACTIVE
  FINISHED
}

model GamePlayer {
  id          String   @id @default(cuid())
  gameId      String
  game        Game     @relation(fields: [gameId], references: [id])
  address     String
  diceCount   Int      @default(5)
  dice        Int[]    // After reveal
  eliminated  Boolean  @default(false)
  joinedAt    DateTime @default(now())
}

model Bid {
  id          String   @id @default(cuid())
  gameId      String
  game        Game     @relation(fields: [gameId], references: [id])
  player      String
  quantity    Int
  faceValue   Int
  positionId  String?
  createdAt   DateTime @default(now())
}

model Position {
  id          String   @id @default(cuid())
  gameId      String
  game        Game     @relation(fields: [gameId], references: [id])
  player      String
  hlOrderId   String   // Hyperliquid order ID
  asset       String
  size        Float
  isLong      Boolean
  entryPrice  Float
  exitPrice   Float?
  pnl         Float?
  openedAt    DateTime @default(now())
  closedAt    DateTime?
}
```

## Frontend (Next.js + React)

### Architecture

```
frontend/
├── app/
│   ├── layout.tsx             # Root layout
│   ├── page.tsx               # Home/lobby
│   ├── game/[id]/page.tsx     # Game room
│   └── api/
│       └── ws/route.ts        # WebSocket endpoint
├── components/
│   ├── Dice3D.tsx             # React Three Fiber dice
│   ├── GameTable.tsx          # Game UI
│   ├── BidPanel.tsx           # Bid controls
│   ├── PositionCard.tsx       # Position display
│   └── PlayerList.tsx         # Player avatars
├── lib/
│   ├── game-socket.ts         # WebSocket client
│   ├── hyperliquid.ts         # Market data
│   └── wagmi.ts               # Wallet config
└── hooks/
    ├── useGame.ts             # Game state hook
    ├── usePositions.ts        # Position tracking
    └── useMarketData.ts       # Price feeds
```

### Key Components

#### 1. Dice3D.tsx (3D Visualization)

```typescript
import { Canvas } from '@react-three/fiber';
import { Physics, RigidBody } from '@react-three/rapier';

export function Dice3D({ dice, rolling }: { dice: number[]; rolling: boolean }) {
  return (
    <Canvas>
      <Physics>
        {dice.map((value, i) => (
          <RigidBody key={i}>
            <Die value={value} position={[i * 2, 10, 0]} rolling={rolling} />
          </RigidBody>
        ))}
      </Physics>
    </Canvas>
  );
}
```

#### 2. GameTable.tsx (Main UI)

```typescript
export function GameTable({ gameId }: { gameId: string }) {
  const { state, bid, challenge } = useGame(gameId);
  const { positions, pnls } = usePositions(gameId);
  
  return (
    <div className="game-table">
      <PlayerList players={state.players} />
      <Dice3D dice={myDice} rolling={state.phase === 'ROLLING'} />
      <BidPanel onBid={bid} canChallenge={!!state.lastBid} onChallenge={challenge} />
      <PositionList positions={positions} pnls={pnls} />
      <PotDisplay amount={state.pot} />
    </div>
  );
}
```

## Data Flow

### 1. Start Game Flow

```
Frontend                Backend                 Contract                Hyperliquid
   │                      │                        │                        │
   │──createGame()───────>│                        │                        │
   │                      │──createGame()────────>│                        │
   │                      │<─────gameId───────────│                        │
   │<────gameId───────────│                        │                        │
   │                      │                        │                        │
   │──joinGame()──────────>│                        │                        │
   │                      │──joinGame()──────────>│                        │
   │                      │<─────joined───────────│                        │
   │<────joined───────────│                        │                        │
```

### 2. Bid Flow

```
Frontend                Backend                 Contract                Hyperliquid
   │                      │                        │                        │
   │──makeBid()───────────>│                        │                        │
   │                      │──validateBid()────────>│                        │
   │                      │<─────valid─────────────│                        │
   │                      │──openPosition()───────────────────────────────>│
   │                      │<────────positionId─────────────────────────────│
   │                      │──recordPosition()─────>│                        │
   │<────bidAccepted──────│                        │                        │
   │<────positionOpened───│                        │                        │
```

### 3. Challenge Flow

```
Frontend                Backend                 Contract                Hyperliquid
   │                      │                        │                        │
   │──challenge()─────────>│                        │                        │
   │                      │──challenge()──────────>│                        │
   │                      │──revealDice()─────────>│                        │
   │                      │<─────result────────────│                        │
   │                      │──closePositions()─────────────────────────────>│
   │                      │<─────pnl───────────────────────────────────────│
   │<────result───────────│                        │                        │
   │<────settlement───────│                        │                        │
```

## Security Considerations

### Smart Contract
- Reentrancy guards on payable functions
- Time-lock for emergency pause
- Rate limiting on game creation
- Proper access control (only backend can record positions)

### Backend
- WebSocket authentication (signed messages)
- Rate limiting (anti-spam)
- Input validation (bid ranges, player counts)
- Hyperliquid API key rotation
- Position size limits (prevent manipulation)

### Frontend
- Wallet signature verification
- XSS protection (sanitize inputs)
- CSRF tokens
- Rate limiting on actions

## Deployment

### Testnet (Phase 1)
- Contracts: Arbitrum Sepolia
- Hyperliquid: Testnet API
- Backend: Railway/Render
- Frontend: Vercel

### Mainnet (Phase 2)
- Contracts: Arbitrum One
- Hyperliquid: Mainnet API
- Backend: Dedicated VPS (low latency)
- Frontend: Vercel + CDN

## Monitoring

### Metrics to Track
- Active games
- Total volume (positions)
- Liquidation rate
- Average game duration
- Player retention
- P&L distribution

### Alerts
- Hyperliquid API failures
- Smart contract reverts
- WebSocket disconnections
- Abnormal liquidation rate
- Large position sizes (whale detection)

## Future Optimizations

- Layer 2 smart contracts (reduce gas)
- WebSocket clustering (scale to 1000s of games)
- Hyperliquid position caching (reduce API calls)
- CDN for market data (reduce latency)
- Redis for game state (faster reads)

---

**Next:** See implementation in `/contracts`, `/backend`, `/frontend`
