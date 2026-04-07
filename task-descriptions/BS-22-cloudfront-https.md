# BS-22 — Add CloudFront Distribution for HTTPS

## Goal
Serve the frontend over HTTPS via CloudFront, replacing the plain HTTP S3 website endpoint.

## Problem
The S3 website endpoint (`http://battleship-frontend-beta.s3-website-us-east-1.amazonaws.com`) is HTTP-only. This breaks:
- `navigator.clipboard` API (requires secure context)
- Future service worker / PWA capabilities
- General security best practices

## Implementation
Add a CloudFront distribution in the CDK infra project (`battleship-infra`):
- Origin: S3 website endpoint (use HTTP-only custom origin, not S3 origin — needed for SPA routing via S3 error document)
- Default behavior: redirect HTTP → HTTPS
- Error pages: 403/404 → `/index.html` with 200 status (SPA routing)
- Price class: PriceClass_100 (cheapest — US/EU only)
- No custom domain needed initially — CloudFront `*.cloudfront.net` domain is fine

Update the API Gateway CORS `allowOrigins` to include the CloudFront URL.

Update the stack outputs to expose the CloudFront URL.

## What NOT to do
- Do NOT add a custom domain or ACM certificate — keep it simple for now
- Do NOT remove the S3 website config — CloudFront uses it as origin
- Do NOT add WAF or Lambda@Edge

## Acceptance
- `cdk deploy` creates a CloudFront distribution
- Frontend is accessible via `https://{id}.cloudfront.net`
- SPA routing works (direct navigation to `/game/{id}` returns the app, not 404)
- Clipboard copy works on the HTTPS URL
- API CORS allows requests from the CloudFront origin

## Blocked by
BS-05
