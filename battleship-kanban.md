---

kanban-plugin: board

---

## Grooming

- [ ] **BS-14** End-to-end deploy + verify
	CDK deploy, upload frontend to S3, smoke test both modes.
	Blocked by: BS-08, BS-09, BS-12, BS-13 [[task-descriptions/BS-14-e2e-deploy|details]]
- [ ] **BS-22** CloudFront HTTPS distribution
	Add CloudFront in CDK for HTTPS frontend. Fixes clipboard API, security.
	Blocked by: BS-05 [[task-descriptions/BS-22-cloudfront-https|details]]


## TODO



## In Progress


## QA

- [ ] **BS-30** Rematch button broken (at least vs AI)
	Clicking Rematch after game over doesn't work. Needs browser console investigation.
	Blocked by: none [[task-descriptions/BS-30-rematch-broken|details]]
- [ ] **BS-29** Join flow UX polish
	Accept full game URL in join input. Change "Waiting for opponent to join" → "Waiting for opponent to place ships" once P2 joins.
	Blocked by: BS-19 [[task-descriptions/BS-29-join-flow-polish|details]]
- [ ] **BS-27** Spectator view for non-participants
	View any game without a token. Neutral labels ("Player 1 / Player 2", "Winner / Loser"). Read-only boards. Bug fixed.
	Blocked by: BS-24 [[task-descriptions/BS-27-spectator-view|details]]

## Complete

- [x] **BS-26** Placement screen UX polish
	Mode-aware banner ("Playing against AI" vs share link), placement instructions, "Waiting for opponent..." after confirm.
	Blocked by: BS-19 [[task-descriptions/BS-26-placement-ux-polish|details]]
- [x] **BS-28** 1-week TTL for completed games
	Set TTL to now + 7 days on game completion instead of persisting forever.
	Blocked by: none [[task-descriptions/BS-28-completed-game-ttl|details]]
- [x] **BS-10** Game history persistence + API
	Store completed games without TTL, GET /games/history endpoint. Includes BS-24 backend guard.
	Blocked by: BS-07 [[task-descriptions/BS-10-game-history|details]]
- [x] **BS-23** UI stuck after confirming ship placement
	Phase doesn't transition after placement — adapter response mismatch. Refresh fixes it.
	Blocked by: BS-19 [[task-descriptions/BS-23-placement-no-transition|details]]
- [x] **BS-21** Fix copy share link on non-HTTPS
	Share link button silently fails without HTTPS. Make URL text selectable as fallback.
	Blocked by: BS-13 [[task-descriptions/BS-21-copy-link-bug|details]]
- [x] **BS-25** Color-coded ships during firing phase
	Show each ship in its distinct placement color on player board during firing. Match fleet list palette.
	Blocked by: BS-12 [[task-descriptions/BS-25-colorful-ships-firing|details]]
- [x] **BS-24** Guard against rejoining started/completed games
	Resume with token, error without. Fix history links. Backend 409 guard + frontend token check.
	Blocked by: BS-19 [[task-descriptions/BS-24-game-rejoin-guard|details]]
- [x] **BS-08** Multiplayer game flow
	Create/join, player tokens, turn enforcement, polling-optimized state endpoint.
	Blocked by: BS-07 [[task-descriptions/BS-08-multiplayer-flow|details]]
- [x] **BS-19** Frontend API adapter layer
	Translate all 13 frontend↔backend contract mismatches in one adapter module.
	Blocked by: BS-20 [[task-descriptions/BS-19-api-adapter|details]]
- [x] **BS-20** Backend response enrichment
	Add updatedAt to state response, normalize winnerId across endpoints.
	Blocked by: none [[task-descriptions/BS-20-backend-response-fields|details]]
- [x] **BS-09** AI opponent logic
	Hunt/target/destroy algorithm. Runs server-side in fire handler.
	Blocked by: BS-07 [[task-descriptions/BS-09-ai-opponent|details]]
- [x] **BS-17** Rework ship placement controls
	Rotate button + R key, left-click to remove placed ship, fix vertical placement on right edge.
	Blocked by: BS-11 [[task-descriptions/BS-17-placement-controls|details]]
- [x] **BS-18** Fix board grid alignment
	Column headers misaligned with cells, row labels off-center. Also fix X-Player-Token error on waiting screen.
	Blocked by: BS-11 [[task-descriptions/BS-18-board-alignment-fix|details]]
- [x] **BS-13** Frontend: menu + multiplayer join flow
	Mode selection, create/join game, share game link, win/loss screen, rematch.
	Blocked by: BS-11 [[task-descriptions/BS-13-frontend-menu|details]]
- [x] **BS-07** Ship placement + firing APIs
	POST /games, /ships, /fire, GET /state. Server-side validation, anti-cheat, per-gameId locking.
	Blocked by: BS-06 [[task-descriptions/BS-07-game-apis|details]]
- [x] **BS-12** Frontend: firing phase + polling
	Shot selection, hit/miss/sunk display, short polling during opponent turn.
	Blocked by: BS-11 [[task-descriptions/BS-12-frontend-firing-ui|details]]
- [x] **BS-06** Game state model + DynamoDB persistence
	Kotlin domain models, DynamoDB Enhanced Client beans, GameService CRUD.
	Blocked by: BS-03 [[task-descriptions/BS-06-game-state-model|details]]
- [x] **BS-11** Frontend: game board + placement UI
	React + MUI grid, drag/rotate ships, placement validation feedback.
	Blocked by: BS-04 [[task-descriptions/BS-11-frontend-board-ui|details]]
- [x] **BS-05** CDK infrastructure constructs
	DynamoDB table, API Gateway REST API, Lambda (reserved concurrency 1), S3 website hosting.
	Blocked by: BS-02, BS-03 [[task-descriptions/BS-05-cdk-constructs|details]]
- [x] **BS-01** Create GitHub repos
	3 repos: battleship-infra, battleship-backend, battleship-frontend under personal GH account.
	Blocked by: none [[task-descriptions/BS-01-create-repos|details]]
- [x] **BS-02** Initialize CDK project
	TypeScript CDK with DynamoDB table, API Gateway REST, Lambda, S3 constructs.
	Blocked by: BS-01 [[task-descriptions/BS-02-init-cdk|details]]
- [x] **BS-03** Scaffold Kotlin Lambda with Guice
	Gradle project, Guice module, handler skeleton, AppConfig, build shadowJar.
	Blocked by: BS-01 [[task-descriptions/BS-03-scaffold-backend|details]]
- [x] **BS-04** Scaffold React frontend
	Vite + React + MUI, theme setup, content separation, API client stub.
	Blocked by: BS-01 [[task-descriptions/BS-04-scaffold-frontend|details]]


%% kanban:settings
```
{"kanban-plugin":"board","list-collapse":[false,false,false,false,false]}
```
%%
