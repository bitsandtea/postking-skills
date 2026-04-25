---
name: onboarding
description: First-time PostKing setup — register or log in, onboard a brand from a website URL, connect social accounts, and generate your first post.
icon: https://postking.app/icons/onboarding.svg
categories: [productivity, marketing]
free: true
version: 0.1.0
---

# Onboarding

The zero-to-first-post skill. Gets a new user authenticated, with a brand profile, connected socials, and a scheduled post in under 5 minutes.

## How to use

> "I'm new. Set up PostKing for my site https://acme.com."

Flow:

1. **Auth**
   - New user: `register` with email+password — PostKing emails a magic link.
   - Existing user: `login_start` → `login_complete` (device flow).
2. **Brand**
   - `onboard_brand` with your website URL. PostKing crawls, analyzes audience, generates 10 themes. Typically 30–90s.
   - Or `create_brand` manually with name + tone + audience.
3. **Sanity check**
   - `me` — confirm scope, credits, free-tier remaining.
   - `list_brands` — confirm the new brand is there.
4. **Social**
   - `check_social_accounts` — see what's connected.
   - `generate_connect_link` with a `platform` param (`linkedin`, `x`, `instagram`, `threads`, `facebook`) to autostart OAuth.
5. **First post**
   - `generate_post` with a platform and optional theme.
   - `approve_post` with a `scheduledAt` to lock it in.
   - `get_calendar` to confirm.

## Expected output

- Authenticated session with an API key saved.
- Populated brand profile (tone, audience, 10 themes).
- 1+ connected social platform.
- At least one scheduled post.

## Errors to expect

- `UNAUTHORIZED` — re-run `login_start` or `register`.
- `TRIAL_EXPIRED` — upgrade via returned `checkoutUrl`.
- `VALIDATION` — invalid email/password format.
