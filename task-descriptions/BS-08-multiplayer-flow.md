# BS-08 — Multiplayer Game Flow

## Goal
Wire create/join game flow for two human players with turn enforcement and polling-optimized state.

## Key Behaviors
- POST /games with mode=MULTIPLAYER creates game, returns player1Token
- POST /games/{id}/join assigns player2, returns player2Token
- Both players submit ships independently via POST /games/{id}/ships
- When both have placed: status → IN_PROGRESS, random first turn
- GET /games/{id}/state returns `updatedAt` — client skips re-render if unchanged
- Turn switches after each fire; opponent's poll picks up the new state

## Polling Optimization
- Response includes `updatedAt` timestamp
- Client sends `If-Modified-Since` or query param `?since=<ts>`
- Lambda returns 304 Not Modified if no changes → minimal compute

## Player Token Validation
- Every mutating endpoint requires `X-Player-Token` header
- Token matched against game's player1Token or player2Token
- Unauthorized → 403

## Steps
1. Implement join endpoint in handler
2. Add player token validation middleware/helper
3. Implement "both players placed" transition
4. Add `updatedAt` check for polling optimization
5. Test full 2-player flow end-to-end with mock

## Blocked by
BS-07
