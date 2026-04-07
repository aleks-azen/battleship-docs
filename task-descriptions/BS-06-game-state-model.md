# BS-06 — Game State Model + DynamoDB Persistence

## Goal
Define domain models and DynamoDB persistence layer for game state.

## Domain Models

### Game.kt
```kotlin
data class Game(
    val gameId: String,
    val mode: GameMode,          // SINGLE_PLAYER, MULTIPLAYER
    val status: GameStatus,      // PLACING_SHIPS, IN_PROGRESS, COMPLETED
    val player1: PlayerState,
    val player2: PlayerState,    // AI or human
    val currentTurn: Int,        // 1 or 2
    val player1Token: String,
    val player2Token: String,
    val winner: Int?,            // null, 1, or 2
    val createdAt: Long,
    val updatedAt: Long
)
```

### Board.kt
```kotlin
data class Board(
    val ships: List<PlacedShip>,
    val shots: Set<Coordinate>,     // all shots received
    val hits: Set<Coordinate>,      // subset of shots that hit
)
```

### Ship.kt
```kotlin
enum class ShipType(val size: Int) {
    CARRIER(5), BATTLESHIP(4), CRUISER(3), SUBMARINE(3), DESTROYER(2)
}

data class PlacedShip(
    val type: ShipType,
    val start: Coordinate,
    val orientation: Orientation  // HORIZONTAL, VERTICAL
) {
    fun occupiedCells(): List<Coordinate> { ... }
    fun isSunk(hits: Set<Coordinate>): Boolean { ... }
}
```

### PlayerState.kt
```kotlin
data class PlayerState(
    val board: Board,
    val shipsPlaced: Boolean
)
```

## DynamoDB Bean

### GameRecord.kt
Single item per game. Board/ship state serialized as JSON string within `gameData`.

```kotlin
@DynamoDbBean
class GameRecord {
    @get:DynamoDbPartitionKey
    var gameId: String = ""
    var gameData: String = ""       // JSON serialized Game
    var status: String = ""         // For GSI queries
    var mode: String = ""
    var createdAt: Long = 0
    var updatedAt: Long = 0
    var ttl: Long = 0
}
```

## GameService

```kotlin
class GameService @Inject constructor(
    private val dynamoClient: DynamoDbEnhancedClient,
    private val config: AppConfig
) {
    fun createGame(mode: GameMode): Game
    fun getGame(gameId: String): Game?
    fun saveGame(game: Game)
    fun listCompletedGames(): List<Game>
}
```

## Steps
1. Create all domain model classes
2. Create GameRecord DynamoDB bean
3. Implement GameService with CRUD operations
4. Write unit tests for model logic (occupiedCells, isSunk)
5. Write unit tests for GameService with mocked DynamoDB

## Acceptance
- All model classes compile
- GameService CRUD operations work with mocked DynamoDB
- Ship.occupiedCells() returns correct coordinates
- Ship.isSunk() detects sunk ships

## Blocked by
BS-03
