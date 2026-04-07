# LD-02 — DynamoDB Local via Docker Compose

## Goal
Run DynamoDB locally so the backend works without hitting AWS.

## Implementation

### docker-compose.yml
Create `docker-compose.yml` in `battleship-backend/` root:
- Service: `dynamodb-local`
- Image: `amazon/dynamodb-local:latest`
- Port: `8000:8000`
- Command: `-jar DynamoDBLocal.jar -sharedDb`

### AppConfig / BattleshipModule changes
Add support for a `DYNAMODB_ENDPOINT` env var in `BattleshipModule.kt`:
- If `DYNAMODB_ENDPOINT` is set, configure the DynamoDB client with `endpointOverride(URI(endpoint))`
- If not set, use default AWS SDK behavior (real AWS)
- This is the ONLY conditional logic needed — everything else stays the same

### Table creation script
Create `create-table.sh` in `battleship-backend/`:
```bash
aws dynamodb create-table \
  --endpoint-url http://localhost:8000 \
  --table-name battleship-games-beta \
  --attribute-definitions AttributeName=gameId,AttributeType=S \
  --key-schema AttributeName=gameId,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --no-cli-pager
```

### Environment
The `GAMES_TABLE` env var must also be set. For local dev: `GAMES_TABLE=battleship-games-beta`

## What NOT to do
- Do NOT add DynamoDB Local as a Java/Gradle dependency
- Do NOT use testcontainers — this is for manual local dev, not tests
- Do NOT modify GameService or GameRecord

## Acceptance
- `docker compose up -d` starts DynamoDB Local on port 8000
- `./create-table.sh` creates the table without errors
- Backend with `DYNAMODB_ENDPOINT=http://localhost:8000 GAMES_TABLE=battleship-games-beta ./gradlew run` can read/write to local DynamoDB
