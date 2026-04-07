# BS-30 — Rematch button doesn't work (at least vs AI)

## Problem
After completing a game vs AI, clicking Rematch does not work as expected. Reported on the deployed (remote) build.

## Current Rematch Flow
`GamePage.jsx:261-273`:
1. Reads stored mode from sessionStorage (`battleship-mode-{oldGameId}`)
2. Falls back to `GAME_MODES.AI` if not found
3. Calls `api.createGame(mode)` via the adapter (translates `'AI'` → `'SINGLE_PLAYER'`)
4. Stores new token and mode in sessionStorage
5. Navigates to `/game/{newGameId}`

## Root Cause
Error: `Cannot read properties of undefined (reading 'hits')`

The rematch successfully creates a new game and navigates to `/game/{newGameId}`. But the new game page crashes — it polls game state for the fresh game and tries to access `board.hits` on a board object that is undefined (no ships placed yet). The frontend doesn't guard against undefined/null board data in the early game state.

This likely also affects navigating directly to a freshly created game before placement.

## Files
`pages/GamePage.jsx` (handleRematch), `hooks/useGameState.js`, `hooks/apiAdapter.js`

## Acceptance
- Clicking Rematch after an AI game creates a new game and navigates to placement
- Clicking Rematch after a multiplayer game creates a new game with correct mode
- New game starts fresh — no state leaking from previous game
- `npm run build` passes

## Blocked by
None
