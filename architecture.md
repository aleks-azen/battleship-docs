# Battleship — Architecture

## Overview

Turn-based Battleship game with single-player (vs AI) and real-time multiplayer modes. Deployed on AWS free tier as a serverless application.

---

## Tech Stack

| Layer | Technology | Notes |
|---|---|---|
| Infrastructure as Code | TypeScript (CDK) | Defines all AWS resources |
| Business logic (Lambda) | Kotlin (JVM 21) | Raw handler pattern, Guice DI |
| Frontend | React + MUI + Vite | Static site on S3 (website hosting) |
| Storage | DynamoDB | Game state + history, PAY_PER_REQUEST |

---

## Repo Structure

Three separate repositories (following amazenSolutions pattern):

| Repo | Contents |
|---|---|
| `battleship-infra` | CDK TypeScript — all AWS resources |
| `battleship-backend` | Kotlin Lambda — game logic, API handlers |
| `battleship-frontend` | React + MUI + Vite — static SPA |

Each repo has its own `.claude/rules/` for AI-assisted development conventions.

---

## AWS Architecture

```
         S3 Bucket              API Gateway
    (website hosting)              (REST)
     http://...s3-               https://...
     website...com              execute-api...
              \                   /
               \                 /
                \               /
                 \             /
                  \           /
                   Lambda
                 (Kotlin JVM)
                      |
                  DynamoDB
               (game state +
                 history)
```

**All services within AWS free tier:**
- API Gateway REST: 1M requests/month (12 months)
- Lambda: 1M requests + 400K GB-seconds/month
- DynamoDB: 25 GB + 25 RCU/WCU (always free)
- S3: 5GB storage, 20K GET/2K PUT per month (always free)

---

## API Design

Single Lambda behind API Gateway REST. Routes handled by a thin dispatcher.

| Method | Path | Purpose |
|---|---|---|
| POST | /games | Create a new game (returns gameId + player token) |
| POST | /games/{id}/join | Join an existing multiplayer game |
| POST | /games/{id}/ships | Submit ship placements (validated server-side) |
| POST | /games/{id}/fire | Fire at coordinate (returns hit/miss/sunk) |
| GET | /games/{id}/state | Get current game state (filtered per player) |
| GET | /games/history | List completed games with outcomes |

---

## Real-Time Multiplayer: Short Polling

**Decision:** Short polling over WebSocket API.

**Why:** API Gateway WebSocket API is NOT in the AWS free tier ($1/M connection-minutes). For a turn-based game where updates happen every 10-30 seconds, polling every 1-2s is perfectly adequate and stays within free tier.

**How it works:**
- Frontend polls `GET /games/{id}/state` every 1.5s during opponent's turn
- Response includes `lastActionTimestamp` — client skips processing if unchanged
- Polling stops when it's the player's turn (no wasted calls)
- ETag/conditional requests to minimize Lambda compute on no-change polls

---

## DynamoDB Schema

### Games Table

| Attribute | Type | Key | Purpose |
|---|---|---|---|
| gameId | S | PK | UUID game identifier |
| gameData | S | — | JSON blob: ship positions, shots, turn state |
| status | S | — | PLACING_SHIPS, IN_PROGRESS, COMPLETED |
| mode | S | — | SINGLE_PLAYER, MULTIPLAYER |
| player1Token | S | — | Auth token for player 1 |
| player2Token | S | — | Auth token for player 2 (or "AI") |
| winner | S | — | Player who won (null if in progress) |
| createdAt | N | — | Epoch millis |
| updatedAt | N | — | Epoch millis (used for polling optimization) |
| ttl | N | — | Auto-expire after 24h (active games) |

### History Query

Completed games get a 7-day TTL (extended from the 24h active-game TTL on completion). Long enough for history review, short enough to avoid unbounded storage growth. History query uses a scan with `status=COMPLETED` filter — acceptable at demo scale. For production, migrate to either a GSI on `status` (PK) + `createdAt` (SK), or a separate history table to avoid hot-key issues on the main table.

---

## Anti-Cheat Design

**Threat model:** Players can inspect network traffic, modify requests, or poll opponent state.

| Threat | Mitigation |
|---|---|
| See opponent's ships | Server never returns opponent ship positions — only hit/miss/sunk results. `GET /state` filters response per player token. |
| Fire out of turn | Server validates `currentTurn` matches player token before accepting fire. |
| Place ships illegally | Server validates all placement rules (bounds, overlap, ship count/sizes) before accepting. |
| Replay/forge requests | Player tokens are UUIDs generated server-side, returned once at game creation/join. |
| Rapid-fire polling | API Gateway throttling (free tier default: 10K req/s burst). |
| Modify game state directly | DynamoDB not exposed — all writes go through Lambda validation. |

**Principle:** The server is the single source of truth. The client is a view layer only. Every mutation is validated server-side before persistence.

---

## AI Opponent

Moderately intelligent shot selection (not purely random):

1. **Hunt mode:** Random shots on a checkerboard pattern (optimizes coverage)
2. **Target mode:** After a hit, probe adjacent cells (up/down/left/right)
3. **Destroy mode:** Once direction is confirmed (2+ hits in a line), follow the line until miss or sunk
4. **Return to hunt** after sinking a ship

This is the standard "Hunt/Target" algorithm — effective enough to be challenging without being unfair.

---

## Scalability Considerations

**Maximum board size:** Game state is a JSON blob in a single DynamoDB item (400KB limit). A standard 10×10 board serializes to ~2KB. The architecture supports boards up to ~140×140 before hitting the item size limit (two full boards of shots at ~10 bytes each). Beyond that, you'd need a fundamentally different storage model (partitioned state, streaming reads) — a different system entirely.

**Concurrency model:** Lambda reserved concurrency is set to 1. All requests are serialized through a single Lambda instance, and the handler uses a simple in-memory lock on `gameId` to guarantee mutual exclusion for game state mutations. This eliminates race conditions (e.g., two simultaneous fire requests reading the same turn state) without adding DynamoDB conditional write complexity. The tradeoff: concurrent requests to *different* games also serialize, and a burst of requests gets throttled (HTTP 429). This is acceptable for a demo with a small number of concurrent games. For production scale, you'd replace this with DynamoDB conditional updates (optimistic locking on a version attribute).

**AI algorithm:** O(1) amortized for hunt mode, O(k) for target/destroy where k = hits on current ship. Trivial at any supported board size.

---

## Architectural Decisions Log

| # | Decision | Rationale | Date |
|---|---|---|---|
| ADR-01 | Short polling over WebSocket | WebSocket API not in free tier; turn-based game tolerates 1-2s latency | 2026-04-06 |
| ADR-02 | Raw Lambda handlers over Ktor | Faster cold starts (~300ms vs 2-3s); only ~5 endpoints | 2026-04-06 |
| ADR-03 | Guice DI over manual injection | User preference; adds structure for service wiring despite small surface | 2026-04-06 |
| ADR-04 | Separate repos over monorepo | Consistent with team conventions; cleaner CI separation | 2026-04-06 |
| ADR-05 | DynamoDB over RDS/Aurora | Free tier, no VPC needed, TTL for auto-cleanup, PAY_PER_REQUEST | 2026-04-06 |
| ADR-06 | Single Lambda over multiple | Small API surface; one handler dispatches to service layer | 2026-04-06 |
| ADR-07 | S3 website hosting over CloudFront | CloudFront adds CDK complexity (OAC, cache invalidation, 15min deploy) for marginal benefit on a demo. HTTP-only is fine. | 2026-04-06 |
| ADR-08 | Reserved concurrency 1 + in-memory lock over DynamoDB conditional writes | Demo scope, small player count. Serializes all requests through one Lambda instance with gameId-level locking. Avoids conditional write complexity. Trade: concurrent games also serialize; bursts get 429s. | 2026-04-06 |
