# Battleship — Approach & AI Usage

## The Spike

The goal was to demonstrate two things simultaneously: **sound architecture decisions** and **AI-accelerated delivery**. Not one or the other — both, reinforcing each other.

The key insight: AI agents are only as good as the instructions they receive. Time invested in architecture, task descriptions, and rules files pays back exponentially when you're running 3-5 agents in parallel. Bad instructions at scale just produce bad code faster.

---

## Agentic Coding Loop

We run a multi-agent workflow with defined roles:

**Coordinator agent** — receives tasks from the Kanban board, delegates to workers (one per repo, never two on the same repo), runs a QA loop (`/simplify` + `/cr`), and reports back for user sign-off before advancing. The coordinator prompt is versioned in the vault alongside the task descriptions.

**Worker agents** — each owns one repo at a time. They read the task description, read the `.claude/rules/` for their repo, implement, self-review, and report back. Fresh context per task — no stale assumptions carrying over.

**Project status agent** — the user's direct collaborator for architecture decisions, edge case analysis, task refinement, Kanban management, and documentation. Maintains the audit log and writeups. Never writes implementation code.

**DevOps agent** — runs in parallel with implementation agents, zero file conflicts. Stands up the local dev environment, handles deployment, and runs verification. Closes the feedback loop: agents implement, we verify locally, file bugs, agents fix.

The loop: **plan → delegate → implement → QA → verify locally → file bugs → fix → repeat**. At peak, 6 agents running concurrently.

---

## Architecture Choices Worth Highlighting

### CDK Construct Composition
Infrastructure is split into bounded constructs (`GameStorage`, `GameApi`, `FrontendHosting`), wired together in a composition layer (`BattleshipStack`). Stage configuration is externalized in `constants.ts`. Adding a production account is one new entry in constants + `cdk deploy --context stage=prod`. No construct changes needed.

### Concurrency Model (ADR-09)
Rather than adding DynamoDB conditional writes for a demo, we set Lambda reserved concurrency to 1 and use an in-memory `ReentrantLock` per gameId. All requests serialize through a single instance. This is a deliberate tradeoff: simple, correct, and the architecture doc explains exactly what you'd change for production scale (optimistic locking on a version attribute).

### Anti-Cheat by Design
The server is the single source of truth. Opponent ship positions are never returned in API responses — the client only sees hits, misses, and sunk notifications. Player tokens are server-generated UUIDs, validated on every mutating request. These aren't afterthoughts bolted on — they're baked into the API contract from day one.

### Short Polling Over WebSocket (ADR-01)
API Gateway WebSocket API isn't in the free tier. For a turn-based game where updates happen every 10-30 seconds, polling every 1.5s is perfectly adequate. The response includes `updatedAt` for dedup — client skips processing if nothing changed.

---

## Task Bootstrapping & Delegation

Before any implementation code was written, we:
1. **Explored 3 reference projects** in parallel to establish conventions (CDK patterns, Kotlin Lambda patterns, React+MUI patterns)
2. **Made 9 architectural decisions** through structured tradeoff analysis, each documented as an ADR
3. **Decomposed the project into 20 tasks** with explicit dependency chains, acceptance criteria, and file-level implementation specs
4. **Wrote `.claude/rules/` files** for each repo — conventions that every agent follows without being told per-task
5. **Audited all rules** before implementation, finding and fixing 14 issues that would have propagated across agents

The task descriptions are detailed enough that a fresh-context agent can pick one up and implement it correctly without asking questions. When the frontend and backend agents built incompatible API contracts (13 mismatches found), we didn't debug one at a time — we cataloged all 13, created an adapter layer ticket, and assigned it. Systematic, not reactive.

The Kanban board is the single source of truth. Bugs get filed as tickets with the same structure as features. Agents pick them up the same way. There's no context-switching cost between "feature work" and "bug fixing" — it's all just tasks.

---

## Production-Ready, Not a Prototype

This isn't a demo that happens to work. It's a production application that happens to be a demo.

- **Data lifecycle is handled.** Active games auto-expire via DynamoDB TTL after 24 hours — no orphaned state accumulating. Completed games get a 7-day TTL — long enough for history review, then cleaned up automatically. No manual intervention, no cron jobs.
- **Infrastructure is stage-aware.** CDK constructs are parameterized by stage. Adding a production account is one entry in `constants.ts` and a deploy command. Same constructs, same code, different config.
- **Developer onboarding is immediate.** `./dev.sh` starts DynamoDB Local + the backend. `npm run dev` starts the frontend with API proxying. Full local stack in under a minute. No AWS credentials needed for development.
- **Operational concerns are addressed.** Server-side validation on every mutation. Anti-cheat filtering on every response. Reserved concurrency for safe request serialization. Structured logging. The architecture doc captures what's implemented and what the production path looks like for each tradeoff (e.g., ADR-09 explains the move from in-memory locking to DynamoDB conditional writes).
- **Bug discovery is systematic.** When 13 API contract mismatches were found between frontend and backend, they were cataloged in one document, turned into two tickets (adapter layer + backend enrichment), and assigned to agents. No ad-hoc debugging sessions — just tasks on a board.

## Code Quality Without Reading Code

I did not read implementation code during the build. The agents wrote it, the QA loop (`/simplify` + `/cr`) reviewed it, and I verified behavior through the local stack. After everything was complete, I did a final pass over the three repos and was genuinely satisfied with the result — the code is clean, well-structured, and easy to follow for a project of this scope. That's not an accident; it's the `.claude/rules/` files doing their job. When every agent follows the same conventions (package layout, naming, error handling, CORS headers), the output is consistent even across independent sessions.

---

## What This Demonstrates

- **Architecture scales with agents.** Good upfront design (rules, task specs, dependency chains) means agents produce consistent, compatible code across repos. Bad upfront design means you spend all your time fixing integration bugs.
- **Local dev closes the loop.** Standing up a local stack in parallel with implementation means bugs are caught in minutes, not after deploy. The debugging agent's work had zero merge conflicts with implementation agents because the touchpoints were mapped in advance.
- **AI doesn't replace judgment.** Every architectural decision, every scope cut, every "is this actually a problem?" conversation happened with a human. The agents execute; the human decides.
- **The output is deployable software, not a throwaway demo.** TTL-based cleanup, stage-parameterized infra, local dev tooling, documented ADRs — this is how you'd hand a project to a team and have them ship it to production.
