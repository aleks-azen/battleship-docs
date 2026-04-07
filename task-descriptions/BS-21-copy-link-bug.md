# BS-21 — Fix Copy Share Link on Non-HTTPS

## Goal
Make the "share this link with your opponent" text copyable when `navigator.clipboard` is unavailable.

## Problem
The share link button in `GamePage.jsx` uses `navigator.clipboard.writeText()`, which requires a secure context (HTTPS or localhost). On the deployed S3 website (`http://battleship-frontend-beta.s3-website-us-east-1.amazonaws.com`), this API silently fails.

Additionally, the link text isn't directly selectable — double-clicking highlights the entire button rather than just the URL text, so users can't easily copy it manually either.

## Fix
- Display the share URL as selectable text (e.g., an input or styled span) so users can select and copy manually
- Optionally attempt `navigator.clipboard.writeText()` and fall back gracefully if it fails (show the text as selected instead of showing "Copied!")
- This will be fully resolved once HTTPS is added via BS-22, but the link should still be manually copyable regardless

## Acceptance
- Share link text is selectable by double-click or click-drag
- On HTTPS (localhost or CloudFront), clipboard copy still works
- On plain HTTP, no silent failure — user can still copy the link manually
- `npm run build` passes

## Blocked by
BS-13
