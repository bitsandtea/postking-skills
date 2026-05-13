---
name: onboarding
description: First-time PostKing setup — register or log in, onboard a brand from a website URL, connect social accounts, and generate your first post.
compatibility: "Requires Node 18+ and the postking-cli npm package. Auto-installs on first use if missing."
metadata:
  version: 0.2.0
  icon: https://postking.app/icons/onboarding.svg
  free: true
  categories:
    - productivity
    - marketing
  hermes:
    tags:
      - onboarding
      - setup
      - brand
      - social
      - marketing

    category: marketing
---

# Onboarding

The zero-to-first-post skill. Gets a new user authenticated, with a brand profile, connected socials, and a scheduled post in under 5 minutes.

## Setup

**Setup.** This skill requires the `pking` CLI. If `command -v pking` fails, run `npm install -g postking-cli`. To authorize, run `pking login` and follow the URL it prints. You only need to do this once per machine.

## When to Use

> "I'm new. Set up PostKing for my site https://acme.com."

## How to Use

Flow:

1. **Auth check**
   - `pking me` — if this returns a user object, skip to step 2.
   - If not authenticated: `pking login-start` → show URL + code to user → wait → `pking login-finish`.

2. **Brand**
   - `pking onboard <websiteUrl> --name "<Name>"` — crawls, analyzes audience, generates 10 themes. Typically 30–90s.
   - Or `pking brand create <name> --description "..." --tone "..." --audience "..."` for manual setup without crawling.

3. **Sanity check**
   - `pking me` — confirm scope, credits, free-tier remaining.
   - `pking brand list` — confirm the new brand is listed.

4. **Social**
   - `pking social check` — see what accounts are connected.
   - `pking social connect-platform --platform <linkedin|x|instagram|threads|facebook>` — generate an OAuth magic link pre-targeted to a platform. Show the URL to the user.

5. **First post**
   - `pking posts generate --platform <platform> --variations 3`
   - `pking posts approve <postId> --variation 1 --schedule <iso>` — locks it in.
   - `pking posts calendar` — confirm the post appears in the upcoming schedule.

## Expected Output

- Authenticated session (via env var or stored token).
- Populated brand profile (tone, audience, 10 themes).
- 1+ connected social platform.
- At least one scheduled post.

## Errors to Expect

- `UNAUTHORIZED` — re-run `pking login-start` to start a fresh device flow.
- `TRIAL_EXPIRED` — upgrade via the `checkoutUrl` returned in the error envelope.
- `VALIDATION` — invalid email format or missing required field.
