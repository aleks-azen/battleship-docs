# BS-10 — Game History Persistence + API

## Goal
Store completed games without TTL and provide a query API.

## Approach
When game status → COMPLETED:
- Remove TTL attribute (game persists indefinitely)
- Store final state including all moves, winner, timestamps

GET /games/history returns list of completed games with:
- gameId, mode, winner, createdAt, move count, duration

## DynamoDB Query
Scan with `status=COMPLETED` filter, sorted by `createdAt` descending.

## Steps
1. Remove TTL attribute on game completion
2. Implement history endpoint in handler
3. Test with completed game fixtures

## Blocked by
BS-07
