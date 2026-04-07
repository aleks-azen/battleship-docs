# Frontend–Backend Compatibility Issues

The frontend and backend were built with different API contract assumptions. This document catalogs every mismatch, grouped by severity and endpoint.

---

## Critical — Will Crash or 404

### 1. Create Game: Mode enum value `"AI"` does not exist in backend

| Side | Detail |
|------|--------|
| **Frontend sends** | `POST /games` with body `{ "mode": "AI" }` |
| **Backend expects** | `GameMode` enum: `SINGLE_PLAYER` or `MULTIPLAYER` |
| **Frontend file** | `content/game.js:51` — `GAME_MODES.AI = "AI"` |
| **Frontend call** | `HomePage.jsx:109` — `handleCreate(GAME_MODES.AI)` |
| **Backend file** | `model/Enums.kt:3-6` — `enum class GameMode { SINGLE_PLAYER, MULTIPLAYER }` |
| **Result** | Gson throws on deserialization. **Single-player games cannot be created.** |

**Fix options:**
- A) Change frontend `GAME_MODES.AI` to `"SINGLE_PLAYER"`
- B) Add `@SerializedName("AI") SINGLE_PLAYER` alias in backend enum

---

### 2. Place Ships: wrong URL path

| Side | Detail |
|------|--------|
| **Frontend sends** | `POST /games/{id}/place` |
| **Backend route** | `POST /games/{id}/ships` (regex: `/games/[^/]+/ships`) |
| **Frontend file** | `useApi.js:36` — `` `/games/${gameId}/place` `` |
| **Backend file** | `RequestRouter.kt:41,66` — `GAME_SHIPS_PATTERN` |
| **Result** | **404 Not Found.** Ships can never be placed. |

**Fix:** Change frontend URL from `/place` to `/ships`.

---

### 3. Place Ships: wrong request body key

| Side | Detail |
|------|--------|
| **Frontend sends** | `{ "placements": [...] }` |
| **Backend expects** | `{ "ships": [...] }` (`PlaceShipsRequest.ships`) |
| **Frontend file** | `useApi.js:36` — `{ placements }` |
| **Backend file** | `ApiModels.kt:20-22` — `data class PlaceShipsRequest(val ships: List<ShipPlacement>)` |
| **Result** | `ships` deserializes as `null`. **Placement validation fails or NPE.** |

**Fix:** Change frontend from `{ placements }` to `{ ships: placements }`.

---

### 4. Place Ships: nested `start` object vs flat `row`/`col`

| Side | Detail |
|------|--------|
| **Frontend sends** | `{ type, start: { row, col }, orientation }` |
| **Backend expects** | `{ type, row, col, orientation }` (flat `ShipPlacement`) |
| **Frontend file** | `GamePage.jsx:226-230` — `start: { row, col }` |
| **Backend file** | `ApiModels.kt:24-29` — `data class ShipPlacement(val type, val row, val col, val orientation)` |
| **Result** | `row` and `col` deserialize as `0` (default int). **All ships placed at (0,0).** |

**Fix:** Change frontend to send `{ type, row, col, orientation }` instead of nesting under `start`.

---

### 5. Game State: board structure — object vs 2D array

| Side | Detail |
|------|--------|
| **Backend returns** | `BoardView` object: `{ ships: [ShipView], shots: [Coordinate], hits: [Coordinate] }` |
| **Frontend expects** | 2D array: `board[row][col]` = cell state string (e.g. `"empty"`, `"hit"`, `"ship"`) |
| **Backend file** | `RequestRouter.kt:157-168`, `ApiModels.kt:54-58` |
| **Frontend file** | `useGameState.js:59-60` — `setPlayerBoard(state.playerBoard)` then used as `board?.[row]?.[col]` in `GameBoard.jsx:22-26` |
| **Result** | **TypeError** — accessing `[row][col]` on a `BoardView` object. Board rendering crashes. |

**Fix:** Add a transform layer in the frontend (or a new backend response format) that converts `BoardView` → 2D cell-state grid. Needs to:
- Place ships on grid for player board
- Mark hits, misses, sunk cells
- Hide ship positions on opponent board

---

## High — Silent Failure, Broken Game Flow

### 6. Game State: `status` vs `phase` (field name + enum values)

| Side | Detail |
|------|--------|
| **Backend returns** | `status`: `"PLACING_SHIPS"`, `"IN_PROGRESS"`, `"COMPLETED"` |
| **Frontend expects** | `phase`: `"waiting-for-opponent"`, `"placing-ships"`, `"firing"`, `"game-over"` |
| **Backend file** | `ApiModels.kt:46`, `Enums.kt:8-12` |
| **Frontend file** | `useGameState.js:61` — `if (state.phase) setPhase(state.phase)`, `content/game.js:23-28` |
| **Result** | `state.phase` is `undefined` → phase never updates from polling. Game stays stuck in initial phase. Additionally, even if the field name matched, the enum values differ — there is no backend status for "waiting-for-opponent" or "firing". |

**Mapping needed:**
| Backend `status` | Frontend `phase` | Notes |
|-----------------|-----------------|-------|
| `PLACING_SHIPS` (player2 not joined) | `waiting-for-opponent` | Frontend needs to check if multiplayer + player2 missing |
| `PLACING_SHIPS` (both joined) | `placing-ships` | |
| `IN_PROGRESS` | `firing` | |
| `COMPLETED` | `game-over` | |

---

### 7. Game State: `currentTurn` string vs `isMyTurn` boolean

| Side | Detail |
|------|--------|
| **Backend returns** | `currentTurn`: `"you"` or `"opponent"` |
| **Frontend expects** | `isMyTurn`: `true` or `false` |
| **Backend file** | `RequestRouter.kt:169` |
| **Frontend file** | `useGameState.js:62` — `if (state.isMyTurn !== undefined) setIsMyTurn(state.isMyTurn)` |
| **Result** | `state.isMyTurn` is `undefined` → turn state never updates. Player can never fire. |

**Fix:** Either:
- Backend returns `isMyTurn: Boolean`, or
- Frontend reads `state.currentTurn === "you"`

---

### 8. Game State: missing `updatedAt` field

| Side | Detail |
|------|--------|
| **Backend returns** | No `updatedAt` field in `GameStateResponse` |
| **Frontend expects** | `state.updatedAt` for change detection / dedup |
| **Frontend file** | `useGameState.js:56-57` — `if (state.updatedAt === updatedAtRef.current) return` |
| **Result** | `state.updatedAt` is always `undefined`, which equals the initial `updatedAtRef.current` (`null`) only on first call. After that, `undefined === undefined` → **all subsequent polls are skipped as "no change".** Only the first poll ever processes. |

**Fix:** Either:
- Add `updatedAt` to backend response, or
- Remove dedup check in frontend, or
- Use a hash/version field for change detection

---

### 9. Game State: `winnerId` format in state response

| Side | Detail |
|------|--------|
| **Backend returns** | `winnerId`: `"you"` or `"opponent"` (in game state endpoint) |
| **Frontend expects** | `winner`: field name is `winner`, values `"me"` or `"opponent"` |
| **Backend file** | `RequestRouter.kt:170` — `winnerId = ... "you" else "opponent"` |
| **Frontend file** | `useGameState.js:63` — `if (state.winner) setWinner(state.winner)`, `content/game.js:55-58` — `WINNER.ME = "me"` |
| **Result** | Field name mismatch (`winnerId` vs `winner`) AND value mismatch (`"you"` vs `"me"`). Winner is never detected from polling. |

---

### 10. Fire Response: missing board state and game phase fields

| Side | Detail |
|------|--------|
| **Backend returns** | `FireResponseWithAi`: `{ result, coordinate, sunkShip, gameOver, winnerId, aiResult }` |
| **Frontend expects** | Additional fields: `phase`, `isMyTurn`, `playerBoard`, `opponentBoard`, `aiShot`, `sunkShips` |
| **Frontend file** | `useGameState.js:112-148` — reads `result.opponentBoard`, `result.playerBoard`, `result.phase`, `result.isMyTurn`, `result.aiShot`, `result.sunkShips` |
| **Backend file** | `ApiModels.kt:67-74` |
| **Result** | After firing, frontend cannot update boards, turn state, or show AI counter-shot animation. Game gets stuck after first shot. |

**What's needed in fire response (or via follow-up poll):**
- Updated opponent board (to show hit/miss)
- Updated player board (after AI fires back)
- Current turn indicator
- AI shot coordinate for animation
- Cumulative sunk ship lists

---

### 11. Fire Response: `winnerId` format

| Side | Detail |
|------|--------|
| **Backend returns** | `winnerId`: `"player1"` or `"player2"` (from `FiringService.kt:78`) |
| **Frontend expects** | `winner`: `"me"` or `"opponent"` |
| **Backend file** | `FiringService.kt:78` — `winnerId = "player$playerNumber"` |
| **Frontend file** | `useGameState.js:122` — `if (result.winner) setWinner(result.winner)` |
| **Result** | Field name wrong (`winnerId` vs `winner`), value wrong (`"player1"` vs `"me"`). Note: the game *state* endpoint uses `"you"`/`"opponent"`, but the *fire* endpoint uses `"player1"`/`"player2"` — these are inconsistent even within the backend. |

---

### 12. Create Game response: mode value mismatch for AI detection

| Side | Detail |
|------|--------|
| **Backend returns** | `mode: "SINGLE_PLAYER"` |
| **Frontend checks** | `state.mode === GAME_MODES.AI` which is `"AI"` |
| **Frontend file** | `useGameState.js:68` |
| **Result** | `isAiMode` is never set to `true` from polling. Multiplayer polling logic runs for single-player games. |

---

### 13. Place Ships response: missing fields

| Side | Detail |
|------|--------|
| **Backend returns** | `{ gameId, status }` |
| **Frontend expects** | `{ phase, playerBoard, isMyTurn }` |
| **Backend file** | `RequestRouter.kt:184` — `mapOf("gameId" to game.gameId, "status" to game.status)` |
| **Frontend file** | `useGameState.js:159-161` |
| **Result** | After confirming placement, frontend can't transition to firing phase or display the server-confirmed board. |

---

## Summary

| # | Endpoint | Issue | Severity |
|---|----------|-------|----------|
| 1 | POST /games | Mode `"AI"` vs `SINGLE_PLAYER` | **Crash** |
| 2 | POST /games/{id}/ships | URL: `/place` vs `/ships` | **Crash (404)** |
| 3 | POST /games/{id}/ships | Body key: `placements` vs `ships` | **Crash** |
| 4 | POST /games/{id}/ships | Shape: `start: {row,col}` vs flat | **Crash** |
| 5 | GET /games/{id}/state | Board: object vs 2D array | **Crash** |
| 6 | GET /games/{id}/state | Field: `status` vs `phase` + enum mismatch | Silent fail |
| 7 | GET /games/{id}/state | Field: `currentTurn` vs `isMyTurn` + type mismatch | Silent fail |
| 8 | GET /games/{id}/state | Missing `updatedAt` — dedup breaks | Silent fail |
| 9 | GET /games/{id}/state | `winnerId` vs `winner` + `"you"` vs `"me"` | Silent fail |
| 10 | POST /games/{id}/fire | Response missing boards, phase, turn, aiShot | Silent fail |
| 11 | POST /games/{id}/fire | `winnerId: "player1"` vs `winner: "me"` | Silent fail |
| 12 | POST /games (response) | Mode `"SINGLE_PLAYER"` vs `"AI"` for detection | Silent fail |
| 13 | POST /games/{id}/ships | Response missing phase, board, turn | Silent fail |

**5 crash-level issues, 8 silent-failure issues. No endpoint works correctly end-to-end except `POST /games` for multiplayer and `POST /games/{id}/join`.**

---

## Recommended Fix Strategy

The cleanest approach is to **add an adapter layer on the frontend** — a single `apiAdapter.js` module that:

1. Translates outgoing requests (fix URLs, rename fields, flatten shapes, map enums)
2. Transforms incoming responses (convert `BoardView` → 2D grid, map `status` → `phase`, `currentTurn` → `isMyTurn`, etc.)

This avoids modifying the backend (which serves the Lambda contract correctly) and keeps all translation logic in one place. The adapter sits between `useApi.js` and `useGameState.js`.

Alternatively, the backend's `LocalRunner` or `RequestRouter` could return an enriched local-dev response format, but that diverges the local and deployed behavior.
