---
name: onboarding
description: First-time PostKing setup — register or log in, onboard a brand from a website URL, connect social accounts, and generate your first post.
version: 0.2.0
compatibility: "Requires Node 18+ and the postking-cli npm package. Auto-installs on first use if missing."
metadata:
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

The zero-to-first-post skill. Gets a new user authenticated, with a brand profile, reviewed themes, connected socials, and a scheduled post — paced one stage at a time so the user stays in control.

## Setup

**Setup.** This skill requires the `pking` CLI. If `command -v pking` fails, run `npm i -g postking-cli`. To authorize, run `pking login` and follow the URL it prints. You only need to do this once per machine.

## When to Use

> "I'm new. Set up PostKing for my site https://acme.com."

## Pacing rules

Onboarding is an explicit multi-step flow. Hermes (the agent driving this skill) MUST obey these rules:

1. **One stage per turn.** Run the commands for the current stage, present the result, then stop and wait for the user. Never chain into the next stage in the same turn, even if everything succeeded.
2. **Never reuse remembered values without confirmation.** Brand names, website URLs, social handles, theme choices, and post topics must be reconfirmed in the current conversation. Do not pull them from prior sessions, long-term memory, or earlier context without surfacing the value and asking the user to confirm.
3. **Fail loudly, never silently retry.** If a stage fails, stop immediately, surface the error message verbatim, and ask the user how to proceed. Do not silently rerun the same command or skip ahead.
4. **Zero brands = onboarding is required.** When the user has zero brands, treat onboarding as required, not optional — never proceed to other stages without a brand.

## How to Use

### Stage 1 — Auth

- `pking me` — if this returns a user object, the user is already authenticated; inspect the `Brands:` count and `Active brand:` fields in the output to pick the right branch below.
- Branch on the `pking me` brand state:
  - **`Brands: 0`** (or `Active brand: none` with no brands listed): the agent MUST stop and explicitly onboard the user before doing anything else. Frame it warmly — tell the user PostKing needs a brand to do anything useful, and offer to set one up now. Then ask two questions in plain prose, in this order: (1) "What's the name of the brand you want to set up?", (2) "What's the website URL we should analyse for it?" If the UI supports turn-by-turn questions, ask one per turn; wait for the user's actual answers in the current conversation. Do NOT reuse a brand name or URL the agent remembers from a prior conversation or long-term memory, even if it seems plausible — if a prior-context value exists, surface it as a suggestion only (e.g. "Last time you mentioned `acme.com` — still the right one, or different?") and wait for explicit confirmation. Only after both answers are confirmed does the agent proceed to Stage 2 with those values.
  - **`Brands: ≥1` and `Active brand: none`**: do NOT auto-onboard. Run `pking brand list` to surface existing brands and ask the user which one to make active (`pking brand set <id>`), or whether they want to add a new one. "Add another brand" is the only path that proceeds to Stage 2; selecting an existing brand skips Stage 2 and Stage 3 — run `pking brand info` to confirm the audience analysis is already in place.
  - **`Brands: ≥1` and `Active brand: <id>`**: skip Stage 2 and go straight to Stage 3 (brand reveal) so the user still gets the "wow" view of their existing brand's audience.
- If not authenticated: `pking login-start` → show the URL and code to the user → wait for them to confirm they've completed the browser step → `pking login-finish`.
- Pause here for user confirmation before continuing.

### Stage 2 — Onboard the brand

- Before invoking `pking onboard`, you MUST ask the user — in the current conversation — for the website URL and brand name. Do NOT reuse a URL or brand name remembered from any prior conversation, prior session, or long-term memory. Always confirm with the user in this conversation, even if a plausible value seems known from earlier context. If only one brand context is plausibly known, still surface it and ask "Use <url> as <name>, or give me a different site?" rather than running the command silently.
- `pking onboard <websiteUrl> --name "<Name>"` — crawls, analyzes audience, generates 10 themes. Typically 30–90s. The command also prints a first-pass reveal of what was learned.
- Or `pking brand create <name> --description "..." --tone "..." --audience "..."` for manual setup without crawling.
- Pause here for user confirmation before continuing.

### Stage 3 — Brand reveal

- Run `pking brand info` to pull the full audience + positioning analysis (audience demographics, psyche, pain points, awareness levels, market dynamics, product context) from the agent v1 audience endpoint.
- Do NOT dump raw command output. Read it and present **3–5 "wow" insights** in plain prose — for example: the top 2–3 pain points, the primary audience roles and industry, where they sit on the awareness ladder, the sharpest differentiator from the positioning block, and the strongest trust source.
- Then ask explicitly: "Does this match how you see your brand? Anything off about the audience or positioning?"
- If the user wants edits: there is no CLI ai-edit equivalent yet. Point them to the dashboard at `https://postking.app/dashboard/brands/<brandId>/audience-review` to refine, or capture their corrections so they can be applied later (via `pking brand` PATCH-style commands when available).
- If `pking brand info` reports that audience analysis hasn't run yet, wait a minute and retry once on the user's say-so — do not loop silently.
- Pause here for user confirmation before continuing.

### Stage 4 — Themes review

- `pking brand themes` — full list of the generated themes with descriptions.
- Surface every theme (at minimum the titles, with a one-line gloss for each) to the user and ask whether to keep them, edit specific ones, or regenerate the set. Do not silently proceed.
- To tweak: `pking brand themes edit <themeId> --title "..." --content "..." --intent "..."` or `pking brand themes delete <themeId>`.
- If the set feels off, regenerate with `pking brand generate-themes --count <n>` (n ≤ 10).
- Pause here for user confirmation before continuing.

### Stage 5 — Voice (dashboard)

- There is no CLI command yet for extracting voice from a personal LinkedIn / X / Instagram profile. If the user wants posts to sound like their own writing rather than the default brand tone, point them to the dashboard: `https://postking.app/dashboard/brands/<brandId>/audience-review?onboarding=true` (or the brand voice settings) and have them follow the in-app "Share your best profile" step.
- This is optional — offer it, do not force it. If the user declines, note that and move on.
- Pause here for user confirmation before continuing.

### Stage 6 — Connect socials

- `pking social check` — see what accounts are connected.
- `pking social connect-platform --platform <linkedin|x|instagram|threads|facebook>` — generate an OAuth magic link pre-targeted to a platform. Show the URL to the user and ask them to confirm once they've completed the OAuth flow.
- Ask the user which platform(s) they want connected before running anything — do not pick a default platform silently.
- Pause here for user confirmation before continuing.

### Stage 7 — First post

- Summarise to the user which platform (from the connected socials) and which theme (from the reviewed themes) will be used, and confirm both choices explicitly. Do not pick silently.
- `pking posts generate --platform <platform> --variations 3`
- Show the variations to the user. Ask which variation and what schedule time they want.
- `pking posts approve <postId> --variation <n> --schedule <iso>` — locks it in.
- `pking posts calendar` — confirm the post appears in the upcoming schedule.
- Pause here for user confirmation before continuing.

## Expected Output

- Authenticated session (via env var or stored token).
- Populated brand profile (tone, audience, reviewed/edited themes), confirmed by the user during the Stage 3 reveal.
- Optional voice profile captured via the dashboard "Share your best profile" step.
- 1+ connected social platform.
- At least one scheduled post.

## Errors to Expect

- `UNAUTHORIZED` — re-run `pking login-start` to start a fresh device flow.
- `TRIAL_EXPIRED` — upgrade via the `checkoutUrl` returned in the error envelope.
- `VALIDATION` — invalid email format or missing required field.
- `NOT_FOUND` on the audience reveal — audience analysis hasn't finished yet; wait briefly and ask the user before retrying.
