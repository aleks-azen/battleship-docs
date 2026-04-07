# BS-18 — Fix Board Grid Alignment

## Goal
Fix the game board so column headers, row labels, and cells are properly aligned.

## Problem
See screenshot: `/mnt/d/Users/aleks/Pictures/Screenshots/147msedge_AE5enJblOq.png`

Issues visible:
1. Column headers (1-10) are not aligned with the grid cells below them — they're shifted/offset
2. Row labels (A-J) don't line up centered with their corresponding rows
3. The overall grid alignment is off — headers and cells use different sizing/spacing

## Secondary Issue
There's an "X-Player-Token header is required" error banner showing at the top of the page during the "Waiting for opponent to join" state. This likely means a GET /state poll is firing before the player token is properly set, or it's polling before needed. Fix this if it's in the frontend code (don't suppress the error — fix the root cause so the request isn't made without the token).

## Fix
- Ensure column headers share the same grid layout as the cell rows (same column widths)
- Ensure row labels are vertically centered with their row
- Use a consistent grid structure — e.g., a single CSS grid or table where headers are part of the same layout as cells, not a separate element
- Test at different viewport sizes to ensure alignment holds

## Acceptance
- Column numbers 1-10 are centered above their respective columns
- Row letters A-J are centered beside their respective rows
- Grid looks clean and aligned at desktop and mobile sizes
- No "X-Player-Token" error during normal game flow
- `npm run build` passes

## Blocked by
BS-11
