# LD-04 — Local Dev Startup Script

## Goal
Single script to start the full local backend stack.

## Implementation

Create `dev.sh` in `battleship-backend/`:

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "Starting DynamoDB Local..."
docker compose up -d

echo "Waiting for DynamoDB Local..."
until aws dynamodb list-tables --endpoint-url http://localhost:8000 --no-cli-pager >/dev/null 2>&1; do
  sleep 1
done

echo "Creating table (if not exists)..."
aws dynamodb create-table \
  --endpoint-url http://localhost:8000 \
  --table-name battleship-games-beta \
  --attribute-definitions AttributeName=gameId,AttributeType=S \
  --key-schema AttributeName=gameId,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --no-cli-pager 2>/dev/null || echo "Table already exists"

echo "Starting backend on http://localhost:3000"
DYNAMODB_ENDPOINT=http://localhost:8000 GAMES_TABLE=battleship-games-beta ./gradlew run
```

### Notes
- Frontend is started separately: `cd battleship-frontend && npm run dev`
- Script is meant to be run from the `battleship-backend/` directory
- Ctrl+C stops the backend; DynamoDB Local stays running in Docker (use `docker compose down` to stop)

## What NOT to do
- Do NOT add process managers (concurrently, foreman, etc.)
- Do NOT auto-start the frontend — keep them separate
- Do NOT add this to gradle tasks

## Acceptance
- `chmod +x dev.sh && ./dev.sh` starts DynamoDB Local + backend
- Backend is reachable at `http://localhost:3000`
- Script is idempotent — running twice doesn't error on table creation
