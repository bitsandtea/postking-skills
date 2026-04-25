---
name: postking
description: Generate, schedule, and publish social posts, blogs, SEO articles, and landing pages via PostKing. Use when the user mentions posts, scheduling, content calendars, LinkedIn, Instagram, X/Twitter, threads, Facebook, blogs, SEO, keywords, landing pages, brand voice, or repurposing URLs into social content.
version: 1.0.0
license: MIT
metadata:
  hermes:
    tags: [Marketing, Content, Social Media, SEO, Blogging, Landing Pages]
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

Also trigger when the user says "use PostKing" or "use pking".

## First-time setup

Before any operation, ensure the CLI is installed and authenticated.

1. Check installation:
   ```
   pking --version
   ```
   Expect `1.0.3` or later. If `command not found`, install it:
   ```
   npm install -g postking-cli
   ```

2. Authenticate (only if not already logged in):
   ```
   pking user me
   ```
   - On `401` / not authenticated → run `pking login`. The CLI prints a short code + URL. Show **both** to the user and tell them to visit the URL in their browser, sign in, and paste the code. The CLI polls until they confirm and then exits.
   - If `pking user me` returns a user object, skip login.

3. Pick the active brand:
   ```
   pking brand list
   ```
   - If exactly one brand, it's already active.
   - If multiple, ask the user which one to use, then: `pking brand set <brandId>`.
   - If zero brands, run the **Brand onboarding** flow below before doing anything else.

## Quick Reference

The most common one-shot calls. Always pass `--help` to a subcommand to see all flags.

| Goal | Command |
|---|---|
| Generate a post | `pking posts generate --platform linkedin --variations 3` |
| Approve & schedule | `pking posts approve <postId> --variation 2 --schedule 2026-05-01T14:00:00Z` |
| Repurpose a URL | `pking repurpose <url> --platforms linkedin,x` |
| List upcoming posts | `pking posts calendar` |
| Generate a blog | `pking blogs generate --topic "..." --keyword "..."` |
| Publish a blog | `pking blogs publish <articleId>` |
| Onboard a brand from URL | `pking brand onboard https://example.com --name "Acme"` |
| Run SEO seeds | `pking seo seeds "ai content" "social scheduling"` |
| Generate keyword set | `pking seo generate` |
| Build SEO roadmap | `pking seo roadmap` |
| Generate a landing page | `pking lp generate --topic "..."` |
| Publish a landing page | `pking lp publish <slug>` |

For the full command catalog, read `references/commands.md` in this skill, or run `pking --help` and `pking <group> --help`.

## Common flows

### Brand onboarding (zero → first post)

1. `pking brand onboard <websiteUrl> --name "<Name>"` — crawls the site, analyzes audience, generates 10 themes. Async; the CLI prints progress and exits when done.
2. `pking social check` — see connected accounts. If empty, `pking social connect` opens the OAuth link for a chosen platform.
3. `pking posts generate --platform <platform>` — first draft.
4. `pking posts approve <postId> --schedule <iso>` — schedules it.

### Plan a content week

1. `pking weekly-schedule get` — view the current cadence.
2. `pking weekly-schedule set` — define days/times/platforms (interactive flags).
3. `pking weekly-schedule run-day <YYYY-MM-DD>` — generate all posts for a single day; review and approve.

### Repurpose a URL into multi-platform posts

1. `pking repurpose <url> --platforms linkedin,x,instagram` — generates one post per platform from the URL's content. Returns post IDs.
2. For each post: `pking posts view <id>`, then `pking posts approve <id> --schedule <iso>`.

### SEO end-to-end

1. `pking seo seeds <kw1> <kw2> <kw3>` — register seed keywords.
2. `pking seo generate` — expand into a full keyword set (~100 keywords). Long-running.
3. `pking seo categorize` — tag intent.
4. `pking seo cluster` — group into topic pillars.
5. `pking seo clusters list` — show clusters; ask the user which to target.
6. `pking seo roadmap` — produce ~20 prioritized article topics from the cluster.
7. `pking seo write <roadmapItemId>` — draft an article.
8. `pking seo publish <articleId>` — publish.

### Landing page

1. `pking lp generate --topic "..." --keyword "..."` — async; returns a slug + operation id.
2. `pking lp view <slug>` — preview.
3. `pking lp edit <slug> --instructions "..."` (vibe edit, async). For specific sections, `pking lp side section <slug> <sideKey>` etc.
4. `pking lp publish <slug>`.

### Long-running operations

`pking blogs generate`, `pking lp generate`, `pking lp edit --vibe`, and `pking seo generate` return operation/job ids and poll automatically — but if the agent needs to check status manually:

- Generic jobs: `pking jobs list`
- Blog status: `pking blogs status <articleId>`
- LP vibe-edit status: `pking lp vibe status <slug> <operationId>`

## Pitfalls

- **`401 Unauthorized`** → `pking login` again; the device-code session expired. Don't loop without telling the user.
- **`429 / RATE_LIMITED`** → back off **at least 30 seconds** before retrying. The error envelope's `retryAfter` is authoritative when present.
- **`INSUFFICIENT_CREDITS`** → surface the `checkoutUrl` from the error and stop. Do not retry; the user must top up.
- **`FREE_CAP_REACHED`** (publish-time on free plan) → surface the `checkoutUrl` and stop.
- **Missing `brandId`** → run `pking brand list` first. Never pick a brand silently — ask the user.
- **Async operations** → blogs, LP generate, LP vibe-edit, and SEO keyword generation can take 30s–3min. The CLI polls; do not assume failure before the CLI exits.
- **`pking --version` predates 1.0.0** → tell the user to upgrade with `npm install -g postking-cli@latest` before continuing.

## Verification

After setup, this should always work:

```
pking user me
```

Returns the authenticated user's email, plan tier, and remaining free-tier quota.

```
pking brand list
```

Returns at least one brand (active brand is starred).

If both succeed, the skill is fully operational.

## Full command reference

For the complete `pking` surface (every group, every flag), read `references/commands.md` in this skill. The agent should consult it before running an unfamiliar command.
