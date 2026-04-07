# Known Issues

## HTTP-only deployment — clipboard copy broken

S3 website hosting serves over HTTP, not HTTPS (ADR-07 — no CloudFront). The browser's Clipboard API (`navigator.clipboard.writeText()`) requires a secure context (HTTPS or localhost). The "copy invite link" button on the multiplayer waiting screen silently fails.

**Workaround:** Manually select and copy the link text.

**Fix:** Add CloudFront with ACM certificate for HTTPS. This was deferred to avoid CDK complexity (OAC, cache invalidation, 15-min distribution creation) for the initial deploy. The CDK constructs are stage-parameterized, so adding a CloudFront distribution is additive — no existing construct changes needed.
