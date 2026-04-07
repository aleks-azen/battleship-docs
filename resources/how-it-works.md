# Battleship — How It Works

## Game Modes

### Single Player (vs AI)
The AI uses a hunt/target/destroy algorithm:
- **Hunt:** Fires on a checkerboard pattern (maximizes coverage by eliminating two-cell gaps where ships can't hide)
- **Target:** After a hit, probes all adjacent cells (up/down/left/right) to find the ship's direction
- **Destroy:** Once two hits confirm a line, follows that line until the ship is sunk
- Returns to hunt mode after sinking a ship

The AI places its ships randomly at game creation and fires immediately after the player's turn — no waiting.

### Multiplayer
Create a game and share the invite link. The second player opens the link and joins. Once two players are in, the game is locked — no additional tabs or players can join. Both players place ships independently, and when both have submitted, the game begins with a random first turn.

---

## Session & Identity

Player identity is a server-generated UUID token, stored in the browser's `sessionStorage` (scoped to the tab).

**What this means in practice:**
- **Refresh is safe.** Your token survives page reloads. You can refresh at any point — during placement, mid-game, or on the results screen — and your game state is fully restored.
- **Each tab is a separate player.** Open two tabs to test multiplayer locally. Each tab gets its own token on create/join.
- **Closing the tab loses your identity.** `sessionStorage` is cleared when the tab closes. You can't rejoin a game from a new tab.
- **Game history is tied to your session.** While you play games within a single tab, the history page shows your completed games and whether you won or lost. If you open a fresh tab, you can still view game history but you won't know which player you were.

---

## Spectator Mode

If you access a game's state endpoint without a player token (or with an invalid one), you get a spectator view:

- **Completed games:** Both boards are fully visible — ships, shots, hits. You can review the entire game after it ends.
- **In-progress games:** Both boards show shots and hits, but ship positions are hidden. You can watch the game unfold without seeing either player's fleet.
- **Placing ships:** Nothing is revealed. Spectators see empty boards until the game starts.

---

## Server-Side Validation

Every game mutation is validated before it's accepted:

- **Ship placement:** Exactly 5 ships (one of each type), all within the 10x10 grid, no overlaps. Partial placements are rejected.
- **Firing:** Must be your turn. Coordinate must be in bounds. Can't fire at the same cell twice. Game must be in progress.
- **Turn enforcement:** The server tracks whose turn it is by player number. Requests from the wrong player are rejected.
- **Anti-cheat:** Opponent ship positions are never included in API responses. The client only sees hits, misses, and sunk notifications for the opponent's board.

---

## Polling & Optimization

The frontend polls `GET /games/{id}/state` every 1.5 seconds during the opponent's turn. To minimize wasted compute:

- The response includes `updatedAt` (epoch millis)
- The client sends `?since=<timestamp>` on subsequent polls
- If nothing has changed, the server returns `304 Not Modified` — no response body, no serialization, minimal Lambda execution time
- Polling stops when it's the player's turn

---

## Data Lifecycle

Games are stored in DynamoDB with automatic TTL-based cleanup:

- **Active/incomplete games:** 24-hour TTL. If a game is abandoned mid-placement or mid-match, it's automatically deleted. No orphaned state accumulates.
- **Completed games:** 7-day TTL. Finished games persist long enough for players to review history, then clean up automatically.
- No manual cleanup, no cron jobs, no admin intervention. DynamoDB handles expiration natively.
