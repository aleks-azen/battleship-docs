# BS-02 — Initialize CDK Project

## Goal
Set up the CDK TypeScript project with all construct skeletons.

## Location
`/home/aleks/linuxws/battleShip/battleship-infra/`

## Structure
```
battleship-infra/
├── bin/app.ts                    → Entry point
├── lib/
│   ├── constants.ts              → Stage config (account, region, suffix)
│   ├── battleship-stack.ts       → Composition layer
│   └── constructs/
│       ├── game-storage.ts       → DynamoDB table
│       ├── game-api.ts           → API Gateway REST + Lambda
│       └── frontend-hosting.ts   → S3 + CloudFront
├── test/
│   └── battleship-stack.test.ts
├── cdk.json
├── tsconfig.json
├── package.json
└── .claude/rules/cdk.md
```

## CDK Patterns (from equitas)
- Construct composition: each construct owns one bounded concern
- Stage config in constants.ts (not hardcoded)
- Resource naming: `battleship-{resource}-{stageSuffix}`
- Use CDK grant methods where available
- Explicit LogGroup with stage-driven retention

## Steps
1. `npx cdk init app --language typescript`
2. Create constants.ts with beta stage config
3. Create empty construct files
4. Set up .claude/rules/cdk.md
5. Verify: `npx cdk synth` produces valid CloudFormation
6. Commit and push

## Acceptance
- `npx cdk synth` succeeds
- Construct skeletons exist for all 3 concerns
- Rules file in place

## Blocked by
BS-01
