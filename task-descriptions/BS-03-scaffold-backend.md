# BS-03 — Scaffold Kotlin Lambda with Guice

## Goal
Set up the Kotlin Lambda project with Guice DI, following equitas patterns with Guice addition.

## Location
`/home/aleks/linuxws/battleShip/battleship-backend/`

## Structure
```
battleship-backend/
├── src/main/kotlin/co/amazensolutions/battleship/
│   ├── handler/
│   │   └── BattleshipHandler.kt         → Thin Lambda entrypoint, dispatches by route
│   ├── service/
│   │   ├── GameService.kt               → Game creation, state management
│   │   ├── PlacementService.kt          → Ship placement validation
│   │   ├── FiringService.kt             → Shot processing, hit/miss/sunk
│   │   └── AiService.kt                 → AI opponent logic
│   ├── model/
│   │   ├── Game.kt                      → Domain model
│   │   ├── Ship.kt, Board.kt, Cell.kt  → Game entities
│   │   ├── ApiRequest.kt, ApiResponse.kt
│   │   └── persistence/GameRecord.kt    → DynamoDB bean
│   ├── config/
│   │   ├── AppConfig.kt                 → Env var loading
│   │   └── BattleshipModule.kt          → Guice module
│   └── LocalRunner.kt                   → Local test harness
├── src/test/kotlin/...
├── events/                              → Sample Lambda events
├── build.gradle.kts
├── settings.gradle.kts
├── .claude/rules/
│   ├── kotlin-lambda.md
│   └── kotlin-testing.md
└── CLAUDE.md
```

## Key Patterns
- **Handler:** Single handler that parses API Gateway proxy event, routes by HTTP method + path
- **Guice module:** Binds AppConfig, DynamoDB clients, all services
- **Handler constructor:** `@Inject constructor(private val gameService: GameService, ...)`
- **No-arg constructor** for Lambda runtime: creates Guice injector, gets handler instance
- **DynamoDB Enhanced Client** with `@DynamoDbBean` annotations
- **AWS SDK for Java v2** (not Kotlin SDK)
- **Kotlin noarg plugin** for DynamoDB beans

## Gradle Dependencies
- Kotlin 2.3.x, JVM 21
- AWS SDK v2 BOM (latest stable)
- aws-lambda-java-core, aws-lambda-java-events
- com.google.inject:guice
- Gson for JSON serialization
- Shadow plugin (gradleup)
- Test: MockK, JUnit 5

## Steps
1. Create Gradle project with build.gradle.kts
2. Set up Guice module with AppConfig + client bindings
3. Create handler skeleton that routes API Gateway proxy events
4. Create empty service classes
5. Create domain models (Game, Board, Ship, Cell)
6. Create DynamoDB bean (GameRecord)
7. Set up .claude/rules/ and CLAUDE.md
8. Verify: `./gradlew shadowJar` produces fat JAR
9. Commit and push

## Acceptance
- `./gradlew shadowJar` succeeds
- Guice injector wires all services
- Handler routes to correct service method by path
- Rules files in place

## Blocked by
BS-01
