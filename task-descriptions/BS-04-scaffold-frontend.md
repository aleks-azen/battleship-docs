# BS-04 — Scaffold React Frontend

## Goal
Set up the React + MUI + Vite frontend following amazenSolutions patterns.

## Location
`/home/aleks/linuxws/battleShip/battleship-frontend/`

## Structure
```
battleship-frontend/
├── src/
│   ├── components/          → Reusable UI components
│   │   ├── GameBoard.jsx    → 10x10 grid renderer
│   │   ├── Ship.jsx         → Ship display/drag component
│   │   ├── Cell.jsx         → Individual grid cell
│   │   ├── StatusBar.jsx    → Game status messages
│   │   └── Nav.jsx          → Top navigation
│   ├── pages/
│   │   ├── HomePage.jsx     → Mode selection (vs AI / multiplayer)
│   │   ├── GamePage.jsx     → Main game view (placement + firing)
│   │   └── HistoryPage.jsx  → Completed games list
│   ├── hooks/
│   │   ├── useGameState.js  → Game state management + polling
│   │   └── useApi.js        → API client wrapper
│   ├── content/
│   │   └── game.js          → Ship definitions, labels, messages
│   ├── App.jsx              → Router
│   ├── main.jsx             → Entry + MUI ThemeProvider
│   └── theme.js             → MUI theme config
├── public/
│   └── images/
├── vite.config.js
├── package.json
└── .claude/rules/frontend.md
```

## Key Patterns (from amazenSolutions)
- MUI sx prop styling (no separate CSS)
- Content separation in `content/` files
- Native `fetch()` for API calls (no axios)
- React Router DOM for routing
- Responsive via MUI breakpoints

## Steps
1. `npm create vite@latest -- --template react`
2. Install MUI + emotion + react-router-dom
3. Set up theme.js
4. Create App.jsx with routes (/, /game/:id, /history)
5. Create stub page components
6. Create GameBoard component (10x10 grid of Cells)
7. Create API client hook with base URL config
8. Set up .claude/rules/frontend.md
9. Verify: `npm run dev` serves the app
10. Commit and push

## Acceptance
- `npm run dev` shows the app with navigation
- GameBoard renders a 10x10 grid
- API client points to configurable base URL
- Rules file in place

## Blocked by
BS-01
