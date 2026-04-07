# BS-13 — Frontend: Menu + Multiplayer Join Flow

## Goal
Game mode selection, multiplayer create/join, and win/loss/rematch screens.

## Pages

### HomePage
- "Play vs AI" button → POST /games {mode: SINGLE_PLAYER} → navigate to /game/{id}
- "Play vs Human" button → POST /games {mode: MULTIPLAYER} → show share link
- "Game History" link → /history

### Multiplayer Join
- After creating: show game link (URL with gameId) + "Waiting for opponent..."
- Join URL: /game/{id}/join → POST /games/{id}/join → navigate to /game/{id}
- Poll until opponent joins, then both enter placement phase

### Win/Loss Screen
- Overlay on game board: "You Win!" / "You Lose!"
- "Rematch" button → creates new game with same mode
- "Back to Menu" button → navigate to /

### History Page
- Table: date, mode, result, move count
- Simple MUI Table component

## Steps
1. Build HomePage with mode buttons
2. Implement multiplayer share link flow
3. Build join route + API call
4. Build win/loss overlay with rematch
5. Build history page with API fetch

## Blocked by
BS-11
