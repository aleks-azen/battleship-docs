# BS-29 — Join flow UX polish

## Goal
Improve the join experience: accept full URLs in the join input, and update the waiting banner once both players are in.

## Problems

1. **Join input only accepts game ID.** Users naturally copy the full game URL (e.g., `http://localhost:5173/game/abc-123`) but the join field expects just the ID. Should accept either format and extract the ID automatically.

2. **"Waiting for opponent to join" persists after opponent joins.** Once both players are in the game, the banner should change to "Waiting for opponent to place ships" (or go away entirely if both have placed). Currently it stays on "Waiting for opponent to join" even after P2 has joined — it only changes once you confirm your own placement. The banner update should happen as soon as P2 joins, independent of whether P1 has confirmed placement. This means the frontend needs to detect the join event via polling (e.g., a `player2Joined` field in the state response) and update the banner immediately.

## Fix

### Accept full URL in join input
- In the join input field, if the pasted value looks like a URL containing `/game/`, extract the game ID from it.
- Simple approach: `value.includes('/game/') ? value.split('/game/')[1] : value`
- Strip any trailing slashes or query params.

### Update banner after opponent joins
- The banner must react to P2 joining **immediately via polling**, not only after P1 confirms placement.
- Backend needs to expose `player2Joined` (or equivalent) in the state response so the frontend can distinguish "waiting for join" vs "waiting for placement".
- Once P2 joins → banner changes to "Waiting for opponent to place ships..."
- Once both players have placed → transition to firing phase as normal.
- Current bug: the banner only changes after P1 confirms placement, because that's the first time the frontend re-evaluates game state. Polling during placement must also check for the join event.

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
