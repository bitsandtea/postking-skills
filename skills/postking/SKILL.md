---
name: postking
description: Generate, schedule, and publish social posts, blogs, SEO articles, and landing pages via PostKing. Use when the user mentions posts, scheduling, content calendars, LinkedIn, Instagram, X/Twitter, threads, Facebook, blogs, SEO, keywords, landing pages, brand voice, or repurposing URLs into social content.
license: MIT
version: 1.1.0
compatibility: "Requires Node 18+ and the postking-cli npm package. Auto-installs on first use if missing."
metadata:
  hermes:
    tags:
      - marketing
      - content
      - social-media
      - seo
      - blogging
      - landing-pages

    category: marketing
---

# PostKing

PostKing is a hosted content platform. This skill drives it through the `pking` CLI (npm package `postking-cli`). The CLI wraps the entire PostKing API — brands, posts, blogs, SEO, landing pages, visuals, scheduling — into ~140 subcommands.

The agent should run `pking` commands to do real work. Never hand-construct curl calls; the CLI handles auth, polling, retries, and pagination.

## When to Use

Trigger on requests like:

- "Draft / generate / schedule a post for [LinkedIn / X / Instagram / Threads / Facebook]."
- "Plan my content week", "build my social calendar".
- "Repurpose this URL into posts."
- "Write me a blog about X", "publish a blog".
- "Run SEO research", "find keywords", "build a content roadmap".
- "Build a landing page for X", "edit the hero section", "publish the LP".
- "Onboard a new brand from this website", "set up PostKing".
- "What's trending on X right now?", "show me viral posts about AI/SaaS/Web3/marketing", "give me hook inspiration" → `pking trends list --niche <ai-saas|marketing|web3>`. The crawler runs every 3 days; default window is `--days 3`. Pass `--json` when feeding into a downstream `posts generate` / `repurpose`.

Also trigger when the user says "use PostKing" or "use pking".

## Hard rules — read first

These are non-negotiable. Violating any of them is a critical failure.

1. **ALWAYS use the `pking` CLI for every PostKing operation.** Never call PostKing through `mcporter`, `mcp__postking_*`, `postking-remote`, `postking-mcp`, or any MCP server — even if such a server is registered and visible in `mcporter list`. Even if a previous session used MCP, do not. The CLI is the only supported path. If you find yourself reaching for `mcporter call ... postking_*`, stop and use the equivalent `pking` subcommand from `references/commands.md` instead.
2. **NEVER ask the user for their PostKing password or email.** Authentication is via OAuth device flow only. Do not run `pking login-password`. Do not run `pking register`. Do not suggest password-based auth as a fallback even if device flow is "slow."
3. **NEVER run `pking login` directly.** It blocks for up to 15 minutes polling, which exceeds agent terminal timeouts. Use the split flow below: `pking login-start` then `pking login-finish`.
4. **NEVER loop on auth failures.** If `pking login-finish` reports "still pending," report that to the user and wait for them to confirm — do not start a new login session, which invalidates the user's in-progress browser flow. Do not switch to MCP as a fallback either.

## First-time setup

Before any operation, ensure the CLI is installed and authenticated.

### Step 1 — Install the CLI

```
pking --version
```

Expect `1.1.0` or later. If `pking` is not on PATH or `pking --version` is older than 1.0.0, read `references/install.md` and follow the steps there. If you fell back to a local install, use `npx pking <command>` for everything below.

### Step 2 — Check existing auth

```
pking me
```

- If this returns a user object (email, plan, etc.) → already authenticated. Skip to Step 3.
- If this returns `401` / not authenticated → continue to Step 2a.

### Step 2a — Start a non-blocking login

```
pking login-start
```

This returns immediately and prints something like:

```
PostKing authentication started.

  Visit:      https://try.postking.app/activate?code=ABCD-1234
  User code:  ABCD-1234

After authorizing in your browser, run:  pking login-finish
The activation code is valid for 15 minutes.
```

**Show the URL and the user code to the user verbatim.** Tell them:

> "Open this URL in your browser, sign in to PostKing (or sign up — that takes 2-3 minutes for a new account), confirm the code matches, and tell me when you're done."

Do not proceed until the user confirms they finished. Do not run `login-start` a second time.

### Step 2b — Finish the login

After the user confirms:

```
pking login-finish
```

Three possible outcomes:

- **`SUCCESS: Authenticated`** (exit code 0) → proceed to Step 3.
- **`PENDING: The user has not finished authorizing yet`** (exit code 2) → ask the user to complete the browser flow, wait, then run `pking login-finish` again. **Do not** run `login-start` again — that throws away the in-progress session.
- **`ERROR: ...expired...`** (exit code 1) → the user took longer than 15 minutes. Run `pking login-start` again to issue a fresh code.

### Step 3 — Pick the active brand

```
pking brand list
```

- Exactly one brand → it's active; continue.
- Multiple brands → ask the user which one, then: `pking brand set <brandId>`.
- Zero brands → run the **Brand onboarding** flow below before doing anything else.

## Quick Reference

The most common one-shot calls. Always pass `--help` to a subcommand to see all flags.

| Goal | Command |
|---|---|
| Generate a post | `pking posts generate --platform linkedin --variations 3` |
| Approve & schedule | `pking posts approve <postId> --variation 2 --schedule 2026-05-01T14:00:00Z` |
| Repurpose a URL | `pking repurpose --source-type url --source-url <url> --target-type social --target-platforms linkedin,x` |
| List upcoming posts | `pking posts calendar` |
| Generate a blog | `pking blogs generate --publication <id> --topic "..." --keywords "kw1,kw2"` |
| Publish a blog | `pking blogs publish <articleId>` |
| Onboard a brand from URL | `pking onboard https://example.com --name "Acme"` |
| Run SEO seeds | `pking seo seeds "ai content" "social scheduling"` |
| Generate keyword set | `pking seo generate` |
| Build SEO roadmap | `pking seo roadmap` |
| Generate a landing page | `pking lp generate --topic "..."` |
| Publish a landing page | `pking lp publish <slug>` |

For the full command catalog, read `references/commands.md` in this skill, or run `pking --help` and `pking <group> --help`.

## Common flows

### Brand onboarding (zero → first post)

1. `pking onboard <websiteUrl> --name "<Name>"` — top-level command. Crawls the site, analyzes audience, generates 10 themes. Async; the CLI prints progress and exits when done.
2. `pking social check` — see connected accounts. If empty, `pking social connect` opens a generic OAuth magic link, or `pking social connect-platform --platform <linkedin|x|instagram|threads|facebook>` for a platform-targeted link.
3. `pking posts generate --platform <platform>` — first draft.
4. `pking posts approve <postId> --schedule <iso>` — schedules it.

### Plan a content week

1. `pking weekly-schedule get` — view the current cadence.
2. `pking weekly-schedule set --monday "linkedin:1,x:1" --timezone America/New_York --enable` — define days/timezone/voice in one call.
3. `pking weekly-schedule run-day --date <YYYY-MM-DD>` — generate all posts for a single day; review and approve.

### Repurpose a URL into multi-platform posts

1. `pking repurpose --source-type url --source-url <url> --target-type social --target-platforms linkedin,x,instagram` — generates one post per platform from the URL's content. Returns post IDs.
2. For each post: `pking posts view <id>`, then `pking posts approve <id> --schedule <iso>`.

### SEO end-to-end

1. `pking seo seeds <kw1> <kw2> <kw3>` — register seed keywords.
2. `pking seo generate` — expand into a full keyword set (~100 keywords). Long-running.
3. `pking seo categorize` — tag intent.
4. `pking seo cluster` — group into topic pillars.
5. `pking seo clusters list` — show clusters; ask the user which to target.
6. `pking seo roadmap --cluster <clusterId> --items 20` — produce ~20 prioritized article topics from the cluster.
7. `pking seo write --roadmap-id <roadmapItemId>` — draft an article.
8. `pking seo publish --article-id <articleId>` — publish.

### Landing page

1. `pking lp generate --topic "..."` — async; returns a slug + operation id.
2. `pking lp view <slug>` — preview.
3. `pking lp edit <slug> --instructions "..."` (AI edit pass) or `pking lp vibe <slug> --instructions "..." --wait` for full vibe edits. For specific sections of side-pages, `pking lp side section <slug> <sideKey> --id <sectionId> ...`.
4. `pking lp publish <slug>`.

### Long-running operations

`pking blogs generate`, `pking lp generate`, `pking lp vibe`, and `pking seo generate` return operation/job ids and poll automatically — but if the agent needs to check status manually:

- Generic jobs: `pking jobs list`
- Blog status: `pking blogs status <articleId>`
- LP vibe-edit status: `pking lp vibe status <slug> <operationId>`

## Pitfalls

- **`401 Unauthorized`** → the session expired. Re-run the **First-time setup** auth flow (`login-start` → user authorizes → `login-finish`). Tell the user once; never silently loop.
- **`429 / RATE_LIMITED`** → back off **at least 30 seconds** before retrying. The error envelope's `retryAfter` is authoritative when present.
- **`INSUFFICIENT_CREDITS`** → reply in **one short line** with the `checkoutUrl` from the error envelope and stop. Do not retry, do not recap prior steps, do not promise what will happen after top-up. Example: `Out of credits — top up at <checkoutUrl>.`
- **`FREE_CAP_REACHED`** (publish-time on free plan) → same one-line treatment with the `checkoutUrl`. No recap, no next-step promises.
- **Missing `brandId`** → run `pking brand list` first. Never pick a brand silently — ask the user.
- **Async operations** → blogs, LP generate, LP vibe-edit, and SEO keyword generation can take 30s–3min. The CLI polls; do not assume failure before the CLI exits.
- **`pking --version` predates 1.0.0** → tell the user to read `references/install.md` and follow the upgrade steps there before continuing.
- **Visuals 404 / "command not found"** → almost always a syntax bug, NOT an empty library. The CLI uses `pking visuals <verb>` (no `visuals-post` namespace). `upload` needs `--file <path>`; `import-url <url>` and `search-stock <query>` are positional. A carousel does NOT require an uploaded asset — `pking visuals carousel <postId>` renders directly from the post's cards. If you hit 404s, re-read `references/commands.md` before suggesting the user upload anything.
- **Picking a visual for a post** → always run `pking visuals options <postId> --platform <p>` first, relay the numbered list to the user (especially the `► Recommended` line and the card/quote text shown inline), and submit their choice with `pking visuals pick <postId> --platform <p> --pick <N>`. Do NOT hand-construct `--style/--variant/--asset/--slot` — `--pick <N>` reads the cache that `options` just wrote and resolves the right pickArgs.
- **Describing a visual before picking** → for card/quote templates, `options` prints the resolved render params under each row (colors, avatar, fonts, quote text). Use those when the user asks "what would this look like" — the params ARE the spec; you do not need to render anything.
- **Changing colors or default avatar** → there is NO per-pick override flag. The `options` footer prints a `To change brand colors or default avatar: <url>` line — surface that URL verbatim if the user wants different branding. They edit settings in the dashboard, then re-run `options` to see the new resolved params.
- **"Show me more visuals"** → there is no `--more` flag. The canonical "load more" path is `pking visuals regenerate <postId> --load-external`, which pulls additional Unsplash + Pexels results. Re-run `options` afterwards.
- **`webUrl` on success** → after `posts create / approve / reschedule`, `visuals pick`, `blogs generate / publish`, `lp generate / update / publish`, surface the `View in browser: <url>` line to the user. The user is already logged into the dashboard, so the link works directly.

## Verification

After setup, this should always work:

```
pking me
```

Returns the authenticated user's email, plan tier, and remaining free-tier quota.

```
pking brand list
```

Returns at least one brand (active brand is starred).

If both succeed, the skill is fully operational.

## Full command reference

For the complete `pking` surface (every group, every flag), read `references/commands.md` in this skill. The agent should consult it before running an unfamiliar command.
