# Local Dev Setup — Agent Prompt

You are setting up local development for a Battleship web game so the developer can run the full stack locally and test as other agents make changes.

## Repos
- **Backend**: `/home/aleks/linuxws/battleshipcode/battleship-backend` — Kotlin Lambda with Gradle, Guice DI
- **Frontend**: `/home/aleks/linuxws/battleshipcode/battleship-frontend` — React + MUI + Vite

## Tasks
Complete these in order (each builds on the previous):

1. **LD-01** — Convert `LocalRunner.kt` to a local HTTP server on port 3000 using JDK's `HttpServer`. No new dependencies. See [[LD-01-local-http-server]] for full spec.
2. **LD-02** — Add `docker-compose.yml` for DynamoDB Local + table creation script. Add `DYNAMODB_ENDPOINT` env var support to `BattleshipModule.kt`. See [[LD-02-dynamodb-local]] for full spec.
3. **LD-03** — Configure Vite proxy so `/games/**` routes to `localhost:3000`. Update `useApi.js` default. See [[LD-03-vite-proxy]] for full spec.
4. **LD-04** — Create `dev.sh` startup script in the backend repo. See [[LD-04-dev-script]] for full spec.

## Rules
- Read the task description file BEFORE starting each task
- Read existing code BEFORE modifying it
- Do NOT add unnecessary dependencies
- Do NOT modify service classes, handler logic, or request routing — only touch `LocalRunner.kt`, `BattleshipModule.kt`, `vite.config.js`, `useApi.js`, and new files (`docker-compose.yml`, `create-table.sh`, `dev.sh`)
- Commit each task separately with a clear message
- After completing all 4 tasks, verify by running `./gradlew run` compiles and starts (it may fail on DynamoDB connection if Docker isn't running — that's fine, just confirm it starts the HTTP server)

## End State
- `./dev.sh` in backend repo starts DynamoDB Local + backend on port 3000
- `npm run dev` in frontend repo starts Vite on port 5173 with API proxy to backend
- Developer can open `http://localhost:5173` and interact with the full stack locally
