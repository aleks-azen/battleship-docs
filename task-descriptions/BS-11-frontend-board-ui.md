# BS-11 — Frontend: Game Board + Ship Placement UI

## Goal
Build the core game board component and ship placement interaction.

## Components
- **GameBoard** — 10x10 grid using MUI Grid/Table, labeled A-J rows, 1-10 columns
- **Cell** — Individual cell, colored by state (empty, ship, hit, miss, sunk)
- **ShipPlacer** — Sidebar showing unplaced ships, click to select, click board to place
- **RotateButton** — Toggle horizontal/vertical orientation

## Placement Controls
- **Click cell** — place the selected ship at that cell
- **Click placed ship** — rotate it (H↔V) in place, revalidate position
- **Right-click placed ship** — remove it, return to ship list
- No drag-and-drop. No external libraries.

## Placement Flow
1. Player sees empty board + ship list sidebar
2. Select ship from list → highlight it, show orientation indicator (H/V)
3. Hover over board → show ship preview (green=valid, red=invalid)
4. Left-click to place → validate client-side, show ship on board, auto-select next unplaced ship
5. Left-click an already-placed ship → rotate in place (revalidate; if invalid after rotate, revert + flash red)
6. Right-click an already-placed ship → remove it, return to ship list as unplaced
7. After all 5 placed → "Confirm" button → POST /games/{id}/ships
8. Server validates → transition to firing phase (or show error)

## Styling
- MUI sx prop, no external CSS
- Ship colors: distinct per ship type
- Hit: red marker, Miss: white/gray dot, Sunk: darker red + ship outline

## Steps
1. Build Cell component with state-based coloring
2. Build GameBoard as 10x10 grid of Cells
3. Build ship selection sidebar
4. Implement placement preview on hover
5. Client-side placement validation (bounds, overlap)
6. Confirm placement → API call
7. Responsive for mobile (smaller cells)

## Blocked by
BS-04
