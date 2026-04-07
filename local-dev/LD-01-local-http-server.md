# LD-01 — Convert LocalRunner.kt to Local HTTP Server

## Goal
Replace the single-event JSON file invocation in `LocalRunner.kt` with a running HTTP server so the backend can be tested locally via `./gradlew run`.

## Context
- File: `src/main/kotlin/co/amazensolutions/battleship/LocalRunner.kt`
- Gradle `application` plugin is already configured with `mainClass = LocalRunnerKt`
- `LocalContext` class already exists in the same file — keep it
- The shadow JAR already excludes `LocalRunner*` and `LocalContext*` — no build changes needed

## Implementation
Use `com.sun.net.httpserver.HttpServer` (built into JDK, no new dependencies).

### Requirements
- Listen on port 3000 (or `PORT` env var)
- For each incoming HTTP request, construct an `APIGatewayProxyRequestEvent` with:
  - `httpMethod` from request method
  - `path` from request URI path
  - `headers` from request headers (including `X-Player-Token`)
  - `queryStringParameters` from query string
  - `body` from request body (POST requests)
  - `pathParameters` — extract `{id}` from paths like `/games/{id}/fire`
- Pass the event to `BattleshipHandler().handleRequest(event, LocalContext())`
- Map the `APIGatewayProxyResponseEvent` back to HTTP:
  - Status code
  - Response headers (including CORS)
  - Response body
- Handle `OPTIONS` preflight requests (CORS)
- Print startup message: `Local server running on http://localhost:3000`

### What NOT to do
- Do NOT add any new dependencies to build.gradle.kts
- Do NOT modify BattleshipHandler, RequestRouter, or any service class
- Do NOT create a framework wrapper (no Ktor, no Spring, no Javalin)

## Acceptance
- `./gradlew run` starts a server on port 3000
- `curl http://localhost:3000/games -X POST -d '{"mode":"SINGLE_PLAYER"}'` returns a valid response (or a clean error if DynamoDB isn't configured yet)
- Server logs each request method + path to stdout
