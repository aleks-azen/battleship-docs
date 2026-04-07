# BS-05 — CDK Infrastructure Constructs

## Goal
Implement all CDK constructs to deploy the full Battleship stack.

## Constructs

### GameStorage (game-storage.ts)
- DynamoDB table: `battleship-games-{stage}`
- Partition key: `gameId` (STRING)
- TTL attribute: `ttl`
- Billing: PAY_PER_REQUEST
- Removal policy: DESTROY (beta)

### GameApi (game-api.ts)
- API Gateway REST API: `battleship-api-{stage}`
- Lambda function: Java 21 runtime
- Code: `../battleship-backend/build/libs/battleship-backend-all.jar`
- Handler: `co.amazensolutions.battleship.handler.BattleshipHandler::handleRequest`
- Memory: 512 MB, Timeout: 30s
- **Reserved concurrency: 1** (`reservedConcurrentExecutions: 1`) — serializes all requests through a single instance for concurrency safety (ADR-09)
- Environment: `GAMES_TABLE` (from GameStorage)
- Proxy integration: `{proxy+}` catches all routes
- CORS enabled for S3 website origin

### FrontendHosting (frontend-hosting.ts)
- S3 bucket: `battleship-frontend-{stage}` (website hosting enabled)
- Public read access via bucket policy
- Index document: index.html
- Error document: index.html (SPA routing)
- No CloudFront — HTTP-only is fine for a demo

## IAM
- Lambda gets `dynamodb:*` on games table (use grant methods)
- S3 bucket has public-read policy for website hosting

## Steps
1. Implement GameStorage construct
2. Implement GameApi construct with Lambda + APIGW
3. Implement FrontendHosting construct with S3 website hosting (no CloudFront)
4. Wire all constructs in BattleshipStack
5. Add stack outputs (API URL, S3 website URL, bucket name)
6. `npx cdk synth` to validate
7. Commit and push

## Acceptance
- `npx cdk synth` produces valid CloudFormation with all resources
- Lambda has `reservedConcurrentExecutions: 1`
- Stack outputs include API URL and S3 website URL

## Blocked by
BS-02 (CDK skeleton), BS-03 (backend JAR path)
