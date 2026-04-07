# BS-19 — Frontend API Adapter Layer

## Goal
Add an `apiAdapter.js` module between `useApi.js` and `useGameState.js` that translates between frontend and backend API contracts. All mismatches fixed in one place.

## Outgoing Request Fixes (frontend → backend)

### Create Game (issue #1, #12)
- Map `GAME_MODES.AI` → `"SINGLE_PLAYER"` before sending to `POST /games`

### Place Ships (issues #2, #3, #4)
- Fix URL: `/place` → `/ships`
- Fix body key: `{ placements }` → `{ ships: placements }`
- Flatten shape: `{ type, start: { row, col }, orientation }` → `{ type, row, col, orientation }`

## Incoming Response Fixes (backend → frontend)

### Game State (issues #5, #6, #7, #8, #9)
- Convert `BoardView` object → 2D cell-state grid:
  - Player board: mark ship cells, hit cells, miss cells
  - Opponent board: only hits and misses (no ship positions)
- Map `status` → `phase`:
  - `PLACING_SHIPS` (no player2) → `"waiting-for-opponent"`
  - `PLACING_SHIPS` (both joined) → `"placing-ships"`
  - `IN_PROGRESS` → `"firing"`
  - `COMPLETED` → `"game-over"`
- Map `currentTurn: "you"` → `isMyTurn: true`, `"opponent"` → `false`
- Add `updatedAt` passthrough (backend needs to include it — see BS-20)
- Map `winnerId: "you"` → `winner: "me"`, `"opponent"` → `"opponent"`

### Fire Response (issues #10, #11)
- Map `winnerId: "player1"/"player2"` → `winner: "me"/"opponent"` (use player context)
- After firing, call `GET /state` to get updated boards and return enriched response with:
  - `opponentBoard`, `playerBoard` (as 2D grids)
  - `phase`, `isMyTurn`
  - `aiShot` coordinate (from `aiResult.coordinate`)
  - `sunkShips` list (derived from board ships where `sunk: true`)

### Place Ships Response (issue #13)
- After placing, call `GET /state` to get current phase and board, return enriched response

## Structure
```
useApi.js          → raw HTTP calls (unchanged)
apiAdapter.js      → transforms requests/responses
useGameState.js    → consumes adapted responses (unchanged)
```

`useGameState.js` calls adapter functions instead of `useApi` directly. Adapter calls `useApi` under the hood.

## Acceptance
- Single-player game can be created (mode maps correctly)
- Ships can be placed (URL, body key, shape all correct)
- Board renders without TypeError (2D grid)
- Turn indicator works (isMyTurn updates)
- Game completes with winner detected
- All 13 compatibility issues resolved
