# BS-26 — Placement screen UX polish

## Goal
Make the placement screen context-aware (single-player vs multiplayer) and add placement instructions.

## Problems

1. **Single-player shows join link.** When playing vs AI, the banner says "Waiting for opponent to join..." with a share link. Should say "Playing against AI" with no link.

2. **No placement instructions.** New players have no guidance on how to place ships.

3. **No feedback after confirming placement (multiplayer).** After successful placement in multiplayer, the button should indicate you're waiting for the opponent to finish placing.

## Fix

### Banner — mode-aware text
- **Single-player:** Show "Playing against AI" at the top. No share link, no "Waiting for opponent to join".
- **Multiplayer:** Keep current behavior (share link + waiting message) until opponent joins.

### Placement instructions
Add a short instruction block to the right of the placement area (below or near the fleet list):

1. Press **R** or click the rotate button to rotate your ship.
2. Click the board to place. Click a placed ship to remove it.
3. Hit **Confirm Placement** when done.

### Confirm button state (multiplayer)
- After successful placement response, disable the button and change text to **"Waiting for opponent..."**.
- In single-player, proceed directly to firing phase (no waiting message needed).

## Files
`pages/GamePage.jsx` or placement component, `hooks/useGameState.js`

## Acceptance
- Single-player placement shows "Playing against AI", no share link
- Instructions visible next to fleet list during placement
- After confirming in multiplayer, button says "Waiting for opponent..."
- `npm run build` passes

## Blocked by
BS-19
