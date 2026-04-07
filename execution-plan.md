2# Battleship — Execution Plan

## Parallel Agent Assignment

4 agents running concurrently. Tasks designed for minimal cross-agent dependencies.

---

## Wave 1: Foundation (all 4 agents, parallel)

After BS-01 (repo creation — done by coordinator), all 4 agents launch simultaneously:

| Agent | Task | Repo | Notes |
|---|---|---|---|
| Agent 1 | **BS-02** Init CDK project | battleship-infra | Skeleton constructs only, no Lambda wiring yet |
| Agent 2 | **BS-03** Scaffold Kotlin backend | battleship-backend | Handler + Guice + models + empty services |
| Agent 3 | **BS-04** Scaffold React frontend | battleship-frontend | Vite + MUI + routing + GameBoard component |
| Agent 4 | **BS-06** Game state model + persistence | battleship-backend | Domain models, DynamoDB beans, GameService CRUD |

**Dependency note:** Agents 2 and 4 both write to battleship-backend. Agent 2 sets up project structure; Agent 4 implements models and persistence. **Agent 4 must wait for Agent 2 to finish** OR they can use separate git worktrees.

**Alternative Wave 1 assignment (no conflicts):**

| Agent | Task | Repo |
|---|---|---|
| Agent 1 | **BS-02** Init CDK project | battleship-infra |
| Agent 2 | **BS-03 + BS-06** Scaffold backend + game state model | battleship-backend |
| Agent 3 | **BS-04** Scaffold React frontend | battleship-frontend |
| Agent 4 | **BS-07** Game APIs (placement + firing logic) | battleship-backend (worktree) |

---

## Wave 2: Core Logic (all 4 agents, parallel)

| Agent | Task | Repo | Notes |
|---|---|---|---|
| Agent 1 | **BS-05** CDK constructs (DDB, APIGW, Lambda, S3, CF) | battleship-infra | Wires Lambda JAR path, env vars |
| Agent 2 | **BS-07 + BS-08** Game APIs + multiplayer flow | battleship-backend | All REST endpoints, player tokens, turn logic |
| Agent 3 | **BS-11** Frontend: game board + placement UI | battleship-frontend | Ship placement with rotate, validate, confirm |
| Agent 4 | **BS-09** AI opponent logic | battleship-backend (worktree) | Hunt/target/destroy in AiService |

---

## Wave 3: Integration (all 4 agents, parallel)

| Agent | Task | Repo | Notes |
|---|---|---|---|
| Agent 1 | **BS-14** CDK deploy + smoke test | battleship-infra | Deploy stack, verify endpoints |
| Agent 2 | **BS-10** Game history persistence + API | battleship-backend | History table/GSI, GET /games/history |
| Agent 3 | **BS-12** Frontend: firing phase + polling | battleship-frontend | Shot UI, polling hook, hit/miss display |
| Agent 4 | **BS-13** Frontend: menu + multiplayer join | battleship-frontend (worktree) | Mode select, create/join, win screen |

---

## Wave 4: Polish + Deploy

| Agent | Task | Notes |
|---|---|---|
| Agent 1 | Frontend deploy to S3 + CloudFront invalidation | |
| Agent 2 | End-to-end smoke test both modes | |
| Agent 3 | Writeup document (approach, decisions, AI usage) | |
| Agent 4 | Final code review + cleanup | |

---

## Dependency Graph

```
BS-01 (repos)
  ├── BS-02 (CDK) ──────────────────── BS-05 (constructs) ── BS-14 (deploy)
  ├── BS-03 (backend scaffold) ─┬──── BS-06 (models) ─── BS-07 (APIs) ─┬── BS-08 (multiplayer)
  │                             │                                       └── BS-09 (AI)
  │                             └──── BS-10 (history)
  └── BS-04 (frontend scaffold) ──── BS-11 (board UI) ─┬── BS-12 (firing)
                                                        └── BS-13 (menu)
```

## Critical Path
BS-01 → BS-03 → BS-06 → BS-07 → BS-08 → BS-14

Frontend and CDK run in parallel with backend — they only converge at final deploy.
