# BS-24 — Game session guards (backend + frontend)

## Goal
Make game joining and navigation behave as users expect — no surprises, no broken states.

## Principle
Follow the principle of least surprise. A user should never land on a screen that doesn't make sense for the current game state. Rejoining should feel seamless. Being blocked should feel clear.

## Current Problems

1. **Join link works unlimited times.** `POST /games/{id}/join` hands the player2Token to anyone who calls it during placement. A second person opening the same join link gets the token too. Confusing — they think they joined, but they're sharing a seat.

2. **Completed/in-progress games show a blank board.** Navigating to `/game/{id}` without a session token (e.g., from history page) shows the placement screen with "Waiting for opponent to join..." — makes no sense for a finished game.

3. **DELETE endpoint exists but shouldn't.** Unauthenticated, unused by frontend, games expire via TTL.

## Expected User Experience

### Joining a game
| Scenario | Expected Behavior |
|----------|-------------------|
| First person opens join link | Gets player 2 token, enters game. Normal flow. |
| Same person reopens join link (has token in session) | Frontend detects existing token, skips join API call, navigates straight to game. Seamless resume. |
| Different person opens join link after P2 already joined | Backend returns 409: "This game already has two players." Join page shows error + "Back to Menu" button. |
| Someone opens join link for a completed game | Backend returns 409: "Game is not accepting new players." Clear error. |

### Navigating to a game
| Scenario | Expected Behavior |
|----------|-------------------|
| Has token in session | Game resumes at current phase (placement, firing, or game over screen). |
| No token in session | Shows message: "You are not a participant in this game" + "Back to Menu". Does NOT show a blank placement board. |
| Clicking completed game in history | Either shows final game summary inline, or doesn't link to the game page at all. No interactive board for finished games. |

## Implementation

### Backend

**Track player 2 joining — add `player2Joined` flag:**
- Add `player2Joined: Boolean = false` to `Game` data class
- Persist through `GameRecord`
- On `POST /games/{id}/join`: after existing status check, add `check(!game.player2Joined) { "This game already has two players" }`. Then save game with `player2Joined = true` before returning the token.

**Remove DELETE endpoint:**
- Remove the `DELETE` route and `deleteGame()` method from `RequestRouter`
- Games expire via DynamoDB TTL — manual deletion isn't needed

**Files:** `model/Game.kt`, `model/persistence/GameRecord.kt`, `router/RequestRouter.kt`

### Frontend

**Join page — skip API call if already joined:**
- `JoinPage.jsx`: Before calling `api.joinGame()`, check if a token already exists in sessionStorage for this gameId. If yes, skip the API call and navigate directly to `/game/{id}`.

**Game page — require token:**
- `GamePage.jsx` or `useGameState.js`: On mount, if no token in sessionStorage, show "You are not a participant in this game" with a "Back to Menu" button. Don't render the board.

**History page — don't link to interactive game:**
- `HistoryPage.jsx`: Remove clickable navigation to `/game/{id}`. Show game details inline (winner, mode, move count, date). A read-only game replay view can be added later.

**Files:** `pages/JoinPage.jsx`, `pages/GamePage.jsx` or `hooks/useGameState.js`, `pages/HistoryPage.jsx`

## Acceptance
- First join succeeds, second join from different session returns 409 with clear message
- Reopening join link with existing session token resumes seamlessly
- `/game/{id}` without token shows "not a participant" message, not a blank board
- History page doesn't dump users into an interactive game page for finished games
- `./gradlew test` passes, `npm run build` passes

## Blocked by
BS-19
