# BS-27 — Spectator view for non-participant game viewing

## Goal
Allow anyone to view a completed (or in-progress) game they didn't participate in, with neutral labels instead of "your/enemy" perspective.

## Problem
Currently, navigating to `/game/{id}` without a valid player token shows nothing useful (BS-24 will block with "not a participant"). But users should be able to view games — e.g., from the history page — even if they weren't a player.

## Expected Behavior

### Backend
- `GET /games/{id}/state` should work without `X-Player-Token` (or with an unrecognized token) for completed games.
- When no valid token is provided, return both boards from a neutral perspective (player 1 = "Player 1", player 2 = "Player 2").
- For in-progress games without a token, return limited info (no hidden ship positions — only public knowledge like shots/hits).

### Frontend
- If viewing without a token, render the game in **spectator mode**.
- Labels: "Player 1 (Winner)" / "Player 2 (Loser)" instead of "Your Fleet" / "Enemy Fleet".
- Board should be read-only — no firing, no placement UI.
- For completed games: show both boards fully revealed (all ship positions visible).
- For in-progress games: show only public info (shots and hits, not undetected ship positions).

### History page integration
- History page entries should link to `/game/{id}` for viewing.
- Landing on the page without a token triggers spectator mode automatically.

## Files
`router/RequestRouter.kt` (backend — allow tokenless state request), `pages/GamePage.jsx`, `hooks/useGameState.js`

## Acceptance
- Completed game viewable without token — shows both boards with all ships revealed
- Labels say "Player 1 (Winner)" / "Player 2 (Loser)", not "Your Fleet" / "Enemy Fleet"
- In-progress game viewable without token — hides unrevealed ship positions
- No interactive controls (fire button, placement) in spectator mode
- `./gradlew test` passes, `npm run build` passes

## Blocked by
BS-24
