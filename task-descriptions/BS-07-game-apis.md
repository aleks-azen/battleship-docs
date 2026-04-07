# BS-07 — Ship Placement + Firing APIs

## Goal
Implement all game API endpoints with server-side validation.

## Endpoints

### POST /games
Create a new game. Returns gameId + player1Token.
- Body: `{ "mode": "SINGLE_PLAYER" | "MULTIPLAYER" }`
- Response: `{ "gameId": "...", "playerToken": "...", "playerNumber": 1 }`
- For SINGLE_PLAYER: AI places ships immediately (random valid placement)

### POST /games/{id}/join
Join a multiplayer game. Returns player2Token.
- Headers: none (open join)
- Response: `{ "playerToken": "...", "playerNumber": 2 }`
- Fails if game already has 2 players or is not MULTIPLAYER

### POST /games/{id}/ships
Submit ship placements.
- Headers: `X-Player-Token: <token>`
- Body: `{ "ships": [{ "type": "CARRIER", "start": {"row": 0, "col": 0}, "orientation": "HORIZONTAL" }, ...] }`
- Validates: all 5 ships present, no overlaps, within bounds, correct sizes
- When both players have placed: status → IN_PROGRESS, random first turn

### POST /games/{id}/fire
Fire at a coordinate.
- Headers: `X-Player-Token: <token>`
- Body: `{ "row": 3, "col": 5 }`
- Validates: player's turn, coordinate in bounds, not already fired
- Response: `{ "result": "HIT" | "MISS" | "SUNK", "shipType": "CARRIER" (if sunk), "gameOver": false, "winner": null }`
- If SINGLE_PLAYER and not game over: AI fires immediately, include AI result in response

### GET /games/{id}/state
Get filtered game state for a player.
- Headers: `X-Player-Token: <token>`
- Response includes:
  - Own board: ships + incoming shots (hits/misses)
  - Opponent board: only outgoing shots (hits/misses), NO ship positions
  - currentTurn, status, sunk ships list
  - `updatedAt` for polling optimization

## Validation Rules (anti-cheat)

### Ship Placement
- Exactly 5 ships: Carrier(5), Battleship(4), Cruiser(3), Submarine(3), Destroyer(2)
- All within 0-9 bounds
- No overlaps
- One of each type

### Firing
- Must be player's turn
- Coordinate 0-9 bounds
- Cannot fire at same coordinate twice
- Game must be IN_PROGRESS

## Concurrency / Locking

Lambda reserved concurrency is 1 (set in CDK). As an additional safety measure, the router must use a `ConcurrentHashMap<String, ReentrantLock>` keyed by `gameId`. Acquire the lock before dispatching any mutating request (`POST /fire`, `/ships`, `/join`). Read-only requests (`GET /state`, `/history`) do NOT need the lock. This avoids DynamoDB conditional write complexity for demo scope (ADR-09).

## Handler Routing
```kotlin
fun handleRequest(input: APIGatewayProxyRequestEvent, context: Context): APIGatewayProxyResponseEvent {
    val method = input.httpMethod
    val path = input.path
    return when {
        method == "POST" && path == "/games" -> createGame(input)
        method == "POST" && path.matches("/games/.+/join") -> withGameLock(input) { joinGame(it) }
        method == "POST" && path.matches("/games/.+/ships") -> withGameLock(input) { placeShips(it) }
        method == "POST" && path.matches("/games/.+/fire") -> withGameLock(input) { fire(it) }
        method == "GET" && path.matches("/games/.+/state") -> getState(input)
        method == "GET" && path == "/games/history" -> getHistory(input)
        else -> notFound()
    }
}
```

## Steps
1. Implement PlacementService with validation
2. Implement FiringService with hit/miss/sunk logic
3. Wire handler routing to services
4. Unit test all validation rules
5. Unit test firing logic including sunk detection and win condition
6. Test edge cases: fire same cell twice, fire out of turn, invalid placement

## Acceptance
- All endpoints return correct responses
- Invalid requests return 400 with descriptive error
- Opponent ship positions never leaked in GET /state
- Win condition detected when all ships sunk
- Mutating endpoints (`/join`, `/ships`, `/fire`) acquire a per-gameId lock before processing

## Blocked by
BS-06
