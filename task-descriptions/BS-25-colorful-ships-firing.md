# BS-25 — Color-coded ships on player board during firing phase

## Goal
Show each ship in its distinct color on the player's board during the firing phase, matching the fleet list colors from placement.

## Problem
During ship placement, each ship type has a distinct color (Carrier = navy, Battleship = teal, Cruiser = purple, Submarine = yellow, Destroyer = red). But during the firing phase, the player's board shows all ships in a single uniform color, making it hard to tell which ship is which at a glance.

## Fix
- Carry the ship type information through to the firing phase board rendering
- Use the same color palette from the fleet list / placement phase
- The opponent's board should NOT show ship colors (ships are hidden until sunk)
- Sunk ships on either board can optionally show their color

## Acceptance
- During firing phase, player's board shows each ship in its placement color
- Colors match the fleet list sidebar from placement
- Opponent board still hides ship positions
- Hit/miss/sunk markers remain clearly visible on top of colored ships
- `npm run build` passes

## Blocked by
BS-12
