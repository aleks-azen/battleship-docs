# BS-29 — Join flow UX polish

## Goal
Improve the join experience: accept full URLs in the join input, and update the waiting banner once both players are in.

## Problems

1. **Join input only accepts game ID.** Users naturally copy the full game URL (e.g., `http://localhost:5173/game/abc-123`) but the join field expects just the ID. Should accept either format and extract the ID automatically.

2. **"Waiting for opponent to join" persists after opponent joins.** Once both players are in the game, the banner should change to "Waiting for opponent to place ships" (or go away entirely if both have placed). Currently it stays on the join-waiting message even after player 2 has joined.

## Fix

### Accept full URL in join input
- In the join input field, if the pasted value looks like a URL containing `/game/`, extract the game ID from it.
- Simple approach: `value.includes('/game/') ? value.split('/game/')[1] : value`
- Strip any trailing slashes or query params.

### Update banner after opponent joins
- Once the game has two players (detected via polling or state response), change banner from "Waiting for opponent to join..." to "Waiting for opponent to place ships..."
- Once both players have placed, transition to firing phase as normal.
- The state endpoint should provide enough info to distinguish: waiting for join vs waiting for placement vs in progress.

## Files
`pages/JoinPage.jsx` or `pages/MenuPage.jsx` (join input), `pages/GamePage.jsx` (banner logic), `hooks/useGameState.js`

## Acceptance
- Pasting a full game URL into the join field works (extracts game ID)
- Pasting just the game ID still works
- After opponent joins, banner says "Waiting for opponent to place ships"
- After both place, transitions to firing phase
- `npm run build` passes

## Blocked by
BS-19
