# BS-23 — UI stuck after confirming ship placement

## Goal
After clicking "Confirm Placement", transition the UI to the correct phase (firing or waiting for opponent).

## Problem
Player 1 places all ships and clicks "Confirm Placement". The backend accepts the placement and transitions the game to `IN_PROGRESS`, but the frontend stays on the placement screen showing "Waiting for opponent to join..." with the share link still visible.

Clicking "Confirm Placement" again produces: `Cannot place ships in game with status IN_PROGRESS`.

Refreshing the page shows the correct screen (firing phase).

## Root Cause
The placement API (`POST /games/{id}/ships`) returns `{ gameId, status }` but the frontend's `submitPlacements` in `useGameState.js:156-165` expects `{ phase, playerBoard, isMyTurn }`. Since `result.phase` is undefined, `setPhase()` is never called and the UI stays in the placement phase.

This is part of the broader frontend↔backend contract mismatch (see `compatibility-issues.md` issue #13). The API adapter layer (BS-19) should handle translating the response, but the phase transition after placement must also be verified end-to-end.

## Acceptance
- After clicking "Confirm Placement", UI transitions to firing phase (or waiting for opponent's placement in multiplayer)
- "Waiting for opponent" and share link disappear once the game is in progress
- No stale state on screen — phase matches backend status
- Double-clicking Confirm Placement doesn't produce an error

## Blocked by
BS-19
