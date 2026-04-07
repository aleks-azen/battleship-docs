# BS-28 — 1-week TTL for completed games

## Goal
Completed games should expire after 1 week instead of persisting indefinitely.

## Problem
BS-10 removed the TTL on completed games so they persist forever. This will accumulate stale data in DynamoDB over time. A 1-week window is plenty for reviewing history and spectating.

## Fix
When a game transitions to `COMPLETED`, set its TTL to `now + 7 days` (epoch seconds) instead of removing it.

- In-progress games keep their existing TTL (24h or whatever is currently set).
- Completed games get a fresh 7-day TTL from the moment the game ends.

## Files
`service/GameService.kt` or wherever TTL is set on completion, `model/persistence/GameRecord.kt`

## Acceptance
- Completed games have a TTL of ~7 days from completion
- In-progress game TTL unchanged
- `./gradlew test` passes

## Blocked by
None
