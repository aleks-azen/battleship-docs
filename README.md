# Battleship — Project Documentation

Turn-based Battleship game with single-player AI and real-time multiplayer. AWS serverless (Lambda + DynamoDB + S3), built in one session using a multi-agent AI workflow.

**Code:**
| Repo | Stack |
|------|-------|
| [battleship-infra](https://github.com/aleks-azen/battleship-infra) | CDK TypeScript — all AWS resources |
| [battleship-backend](https://github.com/aleks-azen/battleship-backend) | Kotlin Lambda — game logic, API |
| [battleship-frontend](https://github.com/aleks-azen/battleship-frontend) | React + MUI + Vite — SPA |

---

## Start Here

These three documents tell the full story. Read them in this order:

1. **[Approach & AI Usage](resources/approach-writeup.md)** — How we built it: the multi-agent workflow, architecture decisions, why this is production-ready and not a prototype. **This is the main deliverable.**

2. **[How It Works](resources/how-it-works.md)** — What the game does: AI algorithm, session identity, spectator mode, server-side validation, polling optimization, data lifecycle.

3. **[Architecture](architecture.md)** — Technical deep dive: API design, DynamoDB schema, anti-cheat threat model, concurrency model, ADR log.

---

## Supporting Material

For those who want to go deeper into the process:

| Document                                        | What it shows                                                                                                |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| [Audit Log](resources/audit-log.md)             | Timeline of the build with screenshots — planning, parallel agents, bug fixes, polish                        |
| [Compatibility Issues](compatibility-issues.md) | The 13 API contract mismatches found between frontend and backend, and how they were systematically resolved |
| [Coordinator Prompt](coordinator-prompt.md)     | The actual prompt used to orchestrate 3 worker agents across repos                                           |
| [Execution Plan](execution-plan.md)             | 4-wave parallelization plan with dependency graph                                                            |
| [Known Issues](resources/known-issues.md)       | HTTP-only deployment (clipboard API requires HTTPS)                                                          |
| [task-descriptions/](task-descriptions/)        | 27 individual task specs with acceptance criteria — what each agent received                                 |
| [local-dev/](local-dev/)                        | Local development setup: DynamoDB Local, JDK HTTP server, Vite proxy                                         |

---

## Quick Stats

- **20+ tasks** completed across 3 repos
- **5 agents** running concurrently at peak (coordinator, 3 workers, brainstorm/debug)
- **13 integration bugs** found and resolved systematically via adapter pattern
- **9 architectural decisions** documented with rationale
- **0 manual infrastructure** — fully IaC via CDK, one-command deploy
