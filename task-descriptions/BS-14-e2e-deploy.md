# BS-14 — End-to-End Deploy + Verify

## Goal
Deploy the full stack and verify both game modes work.

## Deploy Steps
1. Build backend: `cd battleship-backend && ./gradlew shadowJar`
2. Build frontend: `cd battleship-frontend && npm run build`
3. CDK deploy: `cd battleship-infra && npx cdk deploy --profile amazen-beta`
4. Upload frontend: `aws s3 sync dist/ s3://battleship-frontend-beta/ --profile amazen-beta`

## Smoke Tests
- [ ] Single-player: create game → place ships → fire → AI responds → play to completion
- [ ] Multiplayer: create game → share link → second browser joins → both place ships → alternate firing → game completes
- [ ] Page refresh mid-game → state restored
- [ ] Game history shows completed games
- [ ] Invalid requests return proper errors
- [ ] Opponent ships never visible in network tab

## Blocked by
BS-09 (BS-07, BS-12, BS-13 complete)
