# BS-09 — AI Opponent Logic

## Goal
Implement moderately intelligent AI shot selection using Hunt/Target/Destroy algorithm.

## Algorithm

### Hunt Mode (default)
- Select random cell from untried cells
- Use checkerboard pattern (row + col is even) to maximize coverage
- Rationale: smallest ship is size 2, so checkerboard guarantees finding all ships

### Target Mode (after a hit on an unsunk ship)
- Probe all 4 adjacent cells (up/down/left/right) of the hit
- Skip cells that are out of bounds, already fired, or adjacent to a sunk ship

### Destroy Mode (after 2+ hits in a line on same unsunk ship)
- Determine direction (horizontal or vertical) from the hit sequence
- Continue firing in that direction until miss or boundary
- Then reverse direction from the first hit

### Return to Hunt
- After sinking a ship, clear target/destroy state
- Resume hunt mode with updated available cells

## AiService Interface
```kotlin
class AiService {
    fun placeShips(): List<PlacedShip>       // Random valid placement
    fun chooseTarget(board: Board): Coordinate  // Next shot
}
```

## Ship Placement (AI)
- For each ship (largest first): pick random start + orientation
- Retry if overlaps or out of bounds
- Max 1000 attempts then fall back to systematic placement

## State Tracking
AI state (hunt/target/destroy, hit queue) stored in game's `gameData` JSON alongside other game state. No separate DynamoDB item.

## Steps
1. Implement random ship placement with validation
2. Implement hunt mode with checkerboard optimization
3. Implement target mode (adjacent probing)
4. Implement destroy mode (line following)
5. Unit test each mode transition
6. Test that AI eventually sinks all ships on any board configuration

## Acceptance
- AI places ships in valid positions every time
- AI never fires at the same cell twice
- AI transitions correctly between hunt → target → destroy → hunt
- AI wins against a static board within a reasonable number of shots (~60-70 for a 10x10 board)

## Blocked by
BS-07
