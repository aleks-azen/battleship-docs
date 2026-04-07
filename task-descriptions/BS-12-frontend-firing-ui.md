# BS-12 — Frontend: Firing Phase + Polling

## Goal
Implement the firing phase UI with dual boards and short polling for multiplayer.

## Layout
- **Left board:** "Your Fleet" — own ships + incoming hits/misses
- **Right board:** "Enemy Waters" — outgoing shots only (hits/misses), clickable to fire

## Firing Flow
1. Click cell on enemy board → POST /games/{id}/fire
2. Show result animation (hit splash / miss splash)
3. Update cell state (hit=red, miss=gray)
4. If sunk: highlight entire ship outline on enemy board
5. If game over: show win/loss overlay

## Polling (useGameState hook)
- Poll GET /games/{id}/state every 1.5s when it's opponent's turn
- Stop polling when it's player's turn
- Use `updatedAt` to skip redundant re-renders
- Show "Waiting for opponent..." message during opponent's turn

## AI Mode
- After firing, response includes AI's counter-shot
- Show AI shot on "Your Fleet" board with brief delay (500ms) for visual effect
- No polling needed — responses are synchronous

## Steps
1. Build dual-board layout
2. Implement fire-on-click → API call → update state
3. Build polling hook with start/stop logic
4. Add hit/miss/sunk animations
5. Build game-over overlay
6. Handle AI mode (synchronous responses)

## Blocked by
BS-11
