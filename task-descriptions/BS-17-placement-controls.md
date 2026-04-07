# BS-17 — Rework Ship Placement Controls

## Goal
Fix placement controls so ships can be placed vertically on the right edge (and generally improve UX).

## Problem
Current rotate-on-click scheme makes it impossible to place a ship vertically on the right board edge. The interaction model is confusing.

## New Control Scheme
- **Left-click cell** — place the selected ship at that cell (unchanged)
- **Left-click a placed ship** — remove it (return to ship list as unplaced). This REPLACES the old rotate-on-click behavior.
- **Right-click** — no longer used for removal
- **Rotate button** — add a visible button to toggle H/V orientation BEFORE placing
- **R key** — keyboard shortcut to rotate (same as clicking the rotate button)
- **Hover preview** — still shows green/red preview based on current orientation

## Summary of Changes
| Action | Old | New |
|---|---|---|
| Place ship | Left-click cell | Left-click cell (same) |
| Rotate before placing | N/A (rotated on placed ship) | Rotate button or R key |
| Remove placed ship | Right-click placed ship | Left-click placed ship |
| Rotate placed ship | Left-click placed ship | Remove + re-place with new orientation |

## Steps
1. Add a Rotate button (H/V toggle) next to the ship list sidebar
2. Add R key listener to toggle orientation
3. Change left-click on placed ship from rotate → remove
4. Remove right-click handler for ship removal
5. Ensure hover preview respects current orientation from the toggle
6. Test: can place ships vertically on right edge (the original bug)
7. `npm run build` must pass

## Acceptance
- Rotate button toggles H/V orientation with visual indicator
- R key toggles orientation
- Left-click places ship, left-click placed ship removes it
- Ships can be placed vertically on columns 9-10 (right edge)
- No right-click behavior remains
- `npm run build` passes

## Blocked by
BS-11
