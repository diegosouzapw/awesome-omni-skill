---
name: add-game
description: Scaffold a new game for the Ancient Games platform. Use when the user says "add a game", "create a new game", "implement [game name]", or similar. Guides implementation of all required backend and frontend pieces.
argument-hint: <game-id> "<Display Name>" "<emoji>"
---

You are helping add a new game to the Ancient Games platform. This is a full-stack TypeScript monorepo (npm workspaces) with:

- `shared/` — types and game manifests
- `backend/` — Node.js + Express + Socket.io + game logic
- `frontend/` — React 18 + Vite + Tailwind CSS

## Architecture: Game Isolation

Each game is self-contained in its own folder. The platform uses registries and manifests so that **adding a new game never requires modifying shared UI components** (Home.tsx, MoveLog, etc.). Changes are limited to:

1. **Engine-level changes** (shared types, backend engine, backend registry) — committed first
2. **Game-specific resources** (board, rules, controls, score, manifest entry) — committed second

## Arguments

Parse `$ARGUMENTS` as: `<game-id> "<Display Name>" "<emoji>"`

- `game-id`: kebab-case identifier (e.g. `fox-and-geese`)
- `Display Name`: human-readable title (e.g. `Fox & Geese`)
- `emoji`: single emoji for the game picker (e.g. `🦊`)

If any argument is missing, ask the user before proceeding.

## Step 0: Understand the game

Before writing any code, if the game rules are not obvious or well-known, ask the user to describe:

1. Board layout and number of positions
2. Number of pieces per player
3. Dice mechanic (or whether it's dice-free — use `rollDice()` returning `1` always)
4. Win condition
5. Any special squares, captures, or multi-phase mechanics

---

## COMMIT 1: Engine & Shared Types

This commit adds the game engine and all shared type changes. No frontend changes.

### Step 1: Shared types — add GameType and GameManifest entry

Edit `shared/types/game.ts`:

**1a. Add to the `GameType` union:**

```typescript
export type GameType = ... | 'GAME_ID';
```

**1b. Add entry to `GAME_MANIFESTS`:**

```typescript
'GAME_ID': {
  type: 'GAME_ID',
  title: 'DISPLAY NAME',
  emoji: 'EMOJI',
  description: '2 players',  // or appropriate description
  playerColors: ['#COLOR1', '#COLOR2'],  // choose distinct colors for each player
  supportsHistory: true,      // set true if the game has a move log
  // supportsAnimation: true, // only if implementing piece animation
  // disabled: true,          // if not yet playable
  // aiGenerated: true,       // if AI-designed game
},
```

The manifest drives: Home.tsx game picker, MoveLog player colors, title display everywhere, and animation gating. No manual changes to those files needed.

### Step 2: Backend — create the game engine

Create `backend/src/games/GAME_ID/GAMECLASSGame.ts`:

```typescript
import { GameEngine } from '../GameEngine';
import { BoardState, Move, Player, PiecePosition } from '@ancient-games/shared';

export class GAMECLASSGame extends GameEngine {
  gameType = 'GAME_ID' as const;
  playerCount = 2;

  private readonly PIECES_PER_PLAYER = N;

  initializeBoard(): BoardState {
    const pieces: PiecePosition[] = [];
    for (let player = 0; player < 2; player++) {
      for (let i = 0; i < this.PIECES_PER_PLAYER; i++) {
        pieces.push({ playerNumber: player, pieceIndex: i, position: -1 });
      }
    }
    return {
      pieces,
      currentTurn: Math.floor(Math.random() * 2),
      diceRoll: null,
      lastMove: null,
    };
  }

  rollDice(): number {
    // Standard d6: return Math.ceil(Math.random() * 6);
    // Binary dice (0–4): sum of 4 coin flips
    // No dice (Morris-style): return 1;
    return Math.ceil(Math.random() * 6);
  }

  validateMove(board: BoardState, move: Move, player: Player): boolean {
    const piece = board.pieces.find(
      (p) => p.playerNumber === player.playerNumber && p.pieceIndex === move.pieceIndex,
    );
    if (!piece) return false;
    if (board.diceRoll === null) return false;
    // TODO: implement game-specific validation
    return false;
  }

  applyMove(board: BoardState, move: Move): BoardState {
    const newPieces = board.pieces.map((p) => ({ ...p }));
    const pieceIdx = newPieces.findIndex(
      (p) => p.playerNumber === board.currentTurn && p.pieceIndex === move.pieceIndex,
    );
    if (pieceIdx === -1) return board;

    newPieces[pieceIdx] = { ...newPieces[pieceIdx], position: move.to };

    // IMPORTANT: applyMove must always:
    //   1. Advance currentTurn (unless extra-turn rule applies)
    //   2. Set diceRoll: null
    const extraTurn = false;
    return {
      ...board,
      pieces: newPieces,
      currentTurn: extraTurn ? board.currentTurn : (board.currentTurn + 1) % 2,
      diceRoll: null,
      lastMove: move,
    };
  }

  checkWinCondition(board: BoardState): number | null {
    for (let playerNumber = 0; playerNumber < 2; playerNumber++) {
      const playerPieces = board.pieces.filter((p) => p.playerNumber === playerNumber);
      if (playerPieces.every((p) => p.position === 99)) return playerNumber;
    }
    return null;
  }

  getValidMoves(board: BoardState, playerNumber: number, diceRoll: number): Move[] {
    const moves: Move[] = [];
    const playerPieces = board.pieces.filter(
      (p) => p.playerNumber === playerNumber && p.position !== 99,
    );
    for (const piece of playerPieces) {
      // TODO: compute legal destinations from piece.position + diceRoll
    }
    return moves;
  }

  canMove(board: BoardState, playerNumber: number, diceRoll: number): boolean {
    return this.getValidMoves(board, playerNumber, diceRoll).length > 0;
  }

  isCaptureMove(board: BoardState, move: Move): boolean {
    // Return true if this move captures an opponent piece by landing on it.
    // Return false if game has no capture-by-landing mechanic (e.g. Morris).
    return false;
  }
}
```

### Key rules for applyMove:

- Always set `diceRoll: null` — the server checks this to know a move was applied
- Always advance `currentTurn` to `(currentTurn + 1) % 2`, unless the game has an extra-turn mechanic
- Return a new `BoardState` object (spread `...board`, then override fields) — never mutate in place

### Step 2b: Backend — add to Mongoose schema enum

Edit `backend/src/models/Session.ts`. The `gameType` field has a hardcoded enum that MongoDB validates against — if you skip this step, session creation will fail with "not a valid enum value":

```typescript
gameType: { type: String, enum: [..., 'GAME_ID'], required: true },
```

### Step 3: Backend — register in GameRegistry

Edit `backend/src/games/GameRegistry.ts`:

```typescript
import { GAMECLASSGame } from './GAME_ID/GAMECLASSGame';
// Add to the Map:
['GAME_ID', new GAMECLASSGame() as GameEngine],
```

### Step 3b: Backend — write game engine tests

Create `backend/src/games/GAME_ID/GAMECLASSGame.test.ts` (colocated with the engine):

```typescript
import { describe, it, expect } from 'vitest';
import { GAMECLASSGame } from './GAMECLASSGame';
import { Move, Player } from '@ancient-games/shared';

const game = new GAMECLASSGame();

function makePlayer(playerNumber: number): Player {
  return { id: 'p', displayName: 'P', socketId: 's', ready: true, playerNumber, status: 'active' };
}

describe('GAMECLASSGame', () => {
  describe('initializeBoard', () => {
    it('creates the correct number of pieces', () => {
      const board = game.initializeBoard();
      expect(board.pieces).toHaveLength(EXPECTED_TOTAL);
      expect(board.pieces.filter((p) => p.playerNumber === 0)).toHaveLength(EXPECTED_PER_PLAYER);
    });

    it('starts with null diceRoll', () => {
      expect(game.initializeBoard().diceRoll).toBeNull();
    });

    it('currentTurn is 0 or 1', () => {
      expect([0, 1]).toContain(game.initializeBoard().currentTurn);
    });
  });

  describe('rollDice', () => {
    it('returns values within expected range', () => {
      const results = new Set<number>();
      for (let i = 0; i < 200; i++) results.add(game.rollDice());
      expect(Math.min(...results)).toBeGreaterThanOrEqual(MIN_ROLL);
      expect(Math.max(...results)).toBeLessThanOrEqual(MAX_ROLL);
    });
  });

  describe('validateMove', () => {
    it('rejects move when diceRoll is null', () => {
      const board = game.initializeBoard();
      const move: Move = { playerId: '', pieceIndex: 0, from: -1, to: 0 };
      expect(game.validateMove(board, move, makePlayer(board.currentTurn))).toBe(false);
    });
    // TODO: add game-specific validation tests
  });

  describe('applyMove', () => {
    // TODO: test piece movement, diceRoll cleared, currentTurn advances, captures
  });

  describe('checkWinCondition', () => {
    it('returns null at game start', () => {
      expect(game.checkWinCondition(game.initializeBoard())).toBeNull();
    });
    // TODO: test win detection
  });

  describe('getValidMoves', () => {
    it('returns moves from initial position', () => {
      const board = game.initializeBoard();
      board.diceRoll = TYPICAL_ROLL;
      const moves = game.getValidMoves(board, board.currentTurn, TYPICAL_ROLL);
      expect(moves.length).toBeGreaterThan(0);
    });
  });
});
```

Replace `EXPECTED_TOTAL`, `EXPECTED_PER_PLAYER`, `MIN_ROLL`, `MAX_ROLL`, `TYPICAL_ROLL` with actual values. Fill in all TODO sections with concrete tests.

### Step 3c: Verify and commit

```bash
npm run build --workspace=shared
npm run build --workspace=backend
npm test  # or: cd backend && npx vitest run
```

**Commit message:** `feat: add DISPLAY NAME game engine`

This commit touches: `shared/types/game.ts`, `backend/src/games/GAME_ID/`, `backend/src/games/GameRegistry.ts`, `backend/src/models/Session.ts`

---

## COMMIT 2: Game-Specific Frontend Resources

This commit adds all frontend pieces. It should NOT modify any shared platform files (GameRoom.tsx board dispatch, Home.tsx, GameControls.tsx dispatcher, etc.) — only add new files to the game folder and register in lookup records.

### Step 4: Frontend — create the board component

Create `frontend/src/components/games/GAME_ID/GAMECLASSBoard.tsx`:

```typescript
import { Session, GameState } from '@ancient-games/shared';
import { socketService } from '../../../services/socket';

interface GAMECLASSBoardProps {
  session: Session;
  gameState: GameState;
  playerId: string;
  isMyTurn: boolean;
  animatingPiece?: { playerNumber: number; pieceIndex: number } | null;
}

export default function GAMECLASSBoard({
  session,
  gameState,
  playerId,
  isMyTurn,
}: GAMECLASSBoardProps) {
  const { board } = gameState;

  function handleRollDice() {
    if (!isMyTurn || board.diceRoll !== null) return;
    socketService.getSocket()?.emit('game:roll-dice', {
      sessionCode: session.sessionCode,
      playerId,
    });
  }

  function handleMove(pieceIndex: number, from: number, to: number) {
    if (!isMyTurn || board.diceRoll === null) return;
    socketService.getSocket()?.emit('game:move', {
      sessionCode: session.sessionCode,
      playerId,
      move: { playerId, pieceIndex, from, to, diceRoll: board.diceRoll },
    });
  }

  // TODO: render the board, pieces, and controls
  return (
    <div className="flex flex-col items-center gap-4 p-4">
      {isMyTurn && board.diceRoll === null && !gameState.finished && (
        <button onClick={handleRollDice} className="btn btn-primary px-6 py-3 text-lg font-semibold">
          Roll Dice
        </button>
      )}
      {board.diceRoll !== null && (
        <div className="text-2xl font-bold" style={{ color: '#E8C870' }}>
          Roll: {board.diceRoll}
        </div>
      )}
      <div className="text-gray-400 text-sm">[Board rendering not yet implemented]</div>
    </div>
  );
}
```

Board rendering notes:

- Use SVG or CSS grid — look at `UrBoard.tsx` for SVG patterns, `MorrisBoard.tsx` for grid patterns
- **SVG must be responsive on mobile**: Use `viewBox` attribute and `width="100%"` with `style={{ maxWidth: SVG_W }}` instead of fixed `width={SVG_W}`. This ensures the board scales down on narrow viewports while staying centered via parent `items-center` flex layout.
  ```tsx
  const SVG_W = 412; // your computed width
  <svg viewBox={`0 0 ${SVG_W} ${SVG_H}`} width="100%" style={{ maxWidth: SVG_W, ... }}>
  ```
- Pieces are in `board.pieces`, filtered by `playerNumber` and `position`
- Use `session.sessionCode` (not `session.code`) when emitting socket events
- `game:move` requires top-level `playerId` in the payload

### Step 5: Frontend — create rules component

Create `frontend/src/components/games/GAME_ID/GAMECLASSRules.tsx`:

```tsx
import { Section } from '../../GameRules';

export default function GAMECLASSRules() {
  return (
    <>
      <div className="text-center pb-1">
        <div className="text-2xl mb-1">EMOJI</div>
        <p className="font-bold" style={{ color: '#F0D090' }}>
          DISPLAY NAME
        </p>
        <p className="text-xs mt-1" style={{ color: '#7A6A50' }}>
          Brief description
        </p>
      </div>
      <Section title="Objective">How to win.</Section>
      <Section title="Movement">How pieces move.</Section>
      <Section title="Special Rules">Any special mechanics.</Section>
    </>
  );
}
```

### Step 5b: Frontend — export a piece preview component

Export a `<GAMECLASSPiecePreview>` component from the board file:

```tsx
// In GAMECLASSBoard.tsx, add near the top (after imports, before the default export):
export function GAMECLASSPiecePreview({ playerNumber, size = 20 }: { playerNumber: 0 | 1; size?: number }) {
  // Render a small SVG of the player's piece at the given size.
  // Player 0 gets their piece, Player 1 gets theirs.
  // If the game has no persistent piece identity (e.g. RPS), return null.
  const color = playerNumber === 0 ? '#PLAYER0_COLOR' : '#PLAYER1_COLOR';
  return (
    <svg viewBox="0 0 20 20" width={size} height={size}>
      <circle cx="10" cy="10" r="8" fill={color} />
    </svg>
  );
}
```

Then register it in `frontend/src/components/games/GamePiecePreview.tsx` — add a `case 'GAME_ID':` to the switch statement:

```tsx
case 'GAME_ID':
  return <GAMECLASSPiecePreview playerNumber={playerNumber} size={size} />;
```

And add the import at the top of `GamePiecePreview.tsx`:

```tsx
import { GAMECLASSPiecePreview } from './GAME_ID/GAMECLASSBoard';
```

### Step 6: Frontend — create score info (optional)

Create `frontend/src/components/games/GAME_ID/gameIdScoreInfo.ts` if the game has meaningful score display:

```ts
import { PiecePosition } from '@ancient-games/shared';

export function getScoreInfo(pieces: PiecePosition[], seatIndex: number): string | null {
  const finished = pieces.filter((p) => p.playerNumber === seatIndex && p.position === 99).length;
  const onBoard = pieces.filter(
    (p) => p.playerNumber === seatIndex && p.position >= 0 && p.position < 99,
  ).length;
  return `${onBoard} on board \u00B7 ${finished} finished`;
}
```

### Step 7: Frontend — create controls (optional)

Create `frontend/src/components/games/GAME_ID/GAMECLASSControls.tsx` if the game needs custom dice/controls UI beyond what the board component provides. Import `GameControlsProps` from `../../GameControls`.

### Step 8: Register in frontend lookup records

These are the **only** shared files that need editing — adding entries to lookup records:

**8a. `frontend/src/components/GameRoom.tsx`** — add to `boardComponents` record:

```typescript
'GAME_ID': lazy(() => import('./games/GAME_ID/GAMECLASSBoard')),
```

**8b. `frontend/src/components/GameRules.tsx`** — add to `rulesComponents` record:

```typescript
'GAME_ID': lazy(() => import('./games/GAME_ID/GAMECLASSRules')),
```

**8c. `frontend/src/utils/gameScoreInfo.ts`** — add import and registry entry (if score info created):

```typescript
import { getScoreInfo as gameIdScore } from '../components/games/GAME_ID/gameIdScoreInfo';
// In registry:
'GAME_ID': gameIdScore,
```

**8d. `frontend/src/components/GameControls.tsx`** — add to `controlComponents` record (if controls created):

```typescript
'GAME_ID': lazy(() => import('./games/GAME_ID/GAMECLASSControls')),
```

**8e. `frontend/src/components/lobby/SessionLobby.tsx`** — replace `GAME_NAMES` usage with `getGameTitle`:

Note: SessionLobby still has a local `GAME_NAMES` record. Replace it with `getGameTitle` from `@ancient-games/shared`, or add the new entry to the existing record until that cleanup is done:

```typescript
GAME_ID: 'DISPLAY NAME',
```

### Step 8f: Verify and commit

```bash
npm run build
npm run lint 2>&1 | head -20  # fix errors if any
```

**Commit message:** `feat: add DISPLAY NAME frontend (board, rules, controls)`

---

## Files NOT touched when adding a game

Thanks to the manifest and registry architecture, these files need **no changes**:

- `Home.tsx` — reads from `GAME_MANIFESTS` automatically
- `MoveLog.tsx` — reads player colors from manifest
- `gameHandlers.ts` — uses `GameRegistry` and engine methods (including `isCaptureMove`)
- `AnimationOverlay.tsx` — only activated for games with `supportsAnimation: true`

## Common pitfalls

1. **`applyMove` must set `diceRoll: null`** — the server checks this to know a move was applied
2. **`applyMove` must advance `currentTurn`** — or the same player moves forever
3. **`validateMove` reads `board.diceRoll`**, not the move's `diceRoll` — the server stores the roll on `board` before calling validate
4. **Position 99 = finished**, not "captured" — filter `!== 99` when computing available pieces
5. **Morris exception**: `diceRoll` is repurposed as a phase indicator. Only do this if your game needs multi-phase turns.
6. **Mongoose enum must be updated** — `backend/src/models/Session.ts` has a separate hardcoded `enum` array. MongoDB will reject session creation with "not a valid enum value" until this is updated.
7. **`session.sessionCode` not `session.code`** — use `session.sessionCode` in socket events
8. **`game:move` requires top-level `playerId`** — payload is `{ sessionCode, playerId, move }`
9. **`isCaptureMove` must be implemented** — even if just returning `false`. The server calls this on every move to determine capture status.
10. **SVG board must use `viewBox` + responsive `width`** — Don't use fixed `width={SVG_W}`. Instead use `viewBox={`0 0 ${SVG_W} ${SVG_H}`}` with `width="100%"` and `style={{ maxWidth: SVG_W, ... }}`. This ensures the board centers on mobile and scales properly without horizontal overflow.

## Checklist

After implementing, verify:

**Commit 1 (Engine):**

- [ ] `shared/types/game.ts` — `GameType` union updated
- [ ] `shared/types/game.ts` — `GAME_MANIFESTS` entry added (with title, emoji, description, colors)
- [ ] `backend/src/models/Session.ts` — Mongoose `gameType` enum updated
- [ ] `backend/src/games/GAME_ID/GAMECLASSGame.ts` — engine created with `isCaptureMove`
- [ ] `backend/src/games/GameRegistry.ts` — engine registered
- [ ] `backend/src/games/GAME_ID/GAMECLASSGame.test.ts` — tests written and passing

**Commit 2 (Frontend):**

- [ ] `frontend/src/components/games/GAME_ID/GAMECLASSBoard.tsx` — board created (default export)
- [ ] `frontend/src/components/games/GAME_ID/GAMECLASSBoard.tsx` — `GAMECLASSPiecePreview` exported
- [ ] `frontend/src/components/games/GamePiecePreview.tsx` — new game registered in switch
- [ ] `frontend/src/components/games/GAME_ID/GAMECLASSRules.tsx` — rules created (default export)
- [ ] `frontend/src/components/games/GAME_ID/gameIdScoreInfo.ts` — score info (if applicable)
- [ ] `frontend/src/components/games/GAME_ID/GAMECLASSControls.tsx` — controls (if applicable)
- [ ] `GameRoom.tsx` — `boardComponents` record entry added
- [ ] `GameRules.tsx` — `rulesComponents` record entry added
- [ ] `gameScoreInfo.ts` — registry entry added (if applicable)
- [ ] `GameControls.tsx` — `controlComponents` record entry added (if applicable)
- [ ] `SessionLobby.tsx` — `GAME_NAMES` entry added
- [ ] `npm run build` passes with no TypeScript errors
