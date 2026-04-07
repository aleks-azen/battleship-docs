# BS-20 — Backend Response Enrichment

## Goal
Add missing fields to backend responses and fix the winnerId inconsistency.

## Changes

### GameStateResponse — add `updatedAt`
**File:** `model/ApiModels.kt` (GameStateResponse)
- Add `updatedAt: Long` field
**File:** `router/RequestRouter.kt` (buildGameStateResponse)
- Include `game.updatedAt` in response

### Fire endpoint — normalize winnerId
**File:** `service/FiringService.kt` line 78
- Currently returns `"player$playerNumber"` — inconsistent with state endpoint's `"you"/"opponent"`
- Change to return player number (`1` or `2`) as an Int
**File:** `router/RequestRouter.kt` (fire handler)
- Convert player number to `"you"/"opponent"` using the requesting player's token (same pattern as state endpoint)

### Place ships endpoint — enrich response
**File:** `router/RequestRouter.kt` (placeShips handler)
- Currently returns `{ gameId, status }`
- Add `status` field (already there) — frontend adapter will call GET /state anyway, so this is optional
- No other changes needed — adapter handles the rest

## Acceptance
- `GET /state` response includes `updatedAt` timestamp
- Fire response `winnerId` uses `"you"/"opponent"` (consistent with state endpoint)
- No breaking changes to existing response fields
