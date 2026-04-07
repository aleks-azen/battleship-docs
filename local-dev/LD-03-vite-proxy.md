# LD-03 — Configure Vite Dev Proxy

## Goal
Proxy API requests from the Vite dev server to the local backend, eliminating CORS issues during local dev.

## Context
- `src/hooks/useApi.js` currently defaults `API_BASE` to `http://localhost:3000`
- This causes CORS issues because the frontend runs on `localhost:5173`
- Fix: proxy through Vite so all requests originate from the same host

## Implementation

### vite.config.js
Add a `server.proxy` config:
```js
export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      '/games': {
        target: 'http://localhost:3000',
        changeOrigin: true,
      },
    },
  },
})
```

### useApi.js
Change the default `API_BASE` to empty string for dev (requests go to same origin, Vite proxies them):
```js
const API_BASE = import.meta.env.VITE_API_URL || '';
```

When `VITE_API_URL` is set (production deploy), it uses the real API Gateway URL. When unset (local dev), requests go to `/games/...` which Vite proxies to `localhost:3000`.

## What NOT to do
- Do NOT install any proxy middleware packages
- Do NOT modify any component or hook other than `useApi.js`
- Do NOT change the production behavior — `VITE_API_URL` must still work when set

## Acceptance
- `npm run dev` starts on port 5173
- Browser requests to `http://localhost:5173/games` are proxied to `http://localhost:3000/games`
- No CORS errors in browser console during local dev
- Production build still uses `VITE_API_URL` env var
