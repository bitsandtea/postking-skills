---
name: postking
description: Generate, schedule, and publish social posts, blogs, SEO / GEO articles, and landing pages via PostKing. Use when the user mentions posts, scheduling, content calendars, LinkedIn, Instagram, X/Twitter, threads, Facebook, blogs, landing pages, brand voice, visuals, trends, or repurposing URLs into social content.
version: 1.2.0
compatibility: "Works with any MCP-compatible client connected to postking-mcp (local stdio or hosted at https://mcp.postking.app); the pking CLI is an optional fast path if a shell is available."
metadata:
  icon: https://postking.app/icons/postking.svg
  free: true
  categories:
    - marketing
    - content
    - social-media
    - seo
    - blogging
    - landing-pages
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

PostKing is a hosted content platform. This is the recommended entry-point skill for day-to-day content work — social posts, repurposing, content weeks, blogs, landing pages, visuals, and trends. It drives PostKing through MCP tool calls (or the `pking` CLI as an optional fast path) across ~140 commands / 200+ tools.

## Minimal tool subset

A representative core subset of the tools this skill uses most:

- `list_brands`, `set_active_brand`, `get_brand_info` — pick and inspect the active brand.
- `generate_post`, `approve_post`, `get_calendar` — draft, lock in, and verify scheduled posts.
- `repurpose_content` — turn a URL / text / blog / existing post into platform-tailored posts.
- `get_weekly_schedule`, `set_weekly_schedule`, `run_weekly_schedule_day` — recurring content-week cadence.
- `generate_blog_post`, `publish_blog_article` — draft and ship blog articles.
- `generate_landing_page`, `publish_landing_page` — draft and ship landing pages.
- `generate_post_visual_options`, `pick_post_visual` — attach a visual to a post.
- `trends_list` — pull trending hooks/topics for inspiration.

…and the full ~140-command / 200+-tool surface documented in `references/commands.md`.

## Prerequisites

Authenticated with credits and an onboarded brand — see the `getting-started` skill if you haven't done that yet.

## Specialist skills

This skill deliberately stays shallow on a few areas that have their own dedicated skill. Use those instead of expecting deep coverage here:

| For… | Use skill |
|---|---|
| First-time auth, credits/billing, brand onboarding from a URL, connecting socials | `getting-started` |
| SEO / GEO — keyword research through published articles | `seo` |
| Multi-channel marketing campaigns (Storylines: brief → strategy → execute) | `campaign-launch` |
| Competitor discovery, registration & analysis | `competitor-intel` |
| Subreddit pool, content-to-community matching, native Reddit rewrites | [`reddit`](../reddit/) |
| Applying/listing saved voice profiles, de-slop / AI-detection pass | [`brand-voice`](../brand-voice/) |

See also: **BrandMind**, PostKing's agentic brand-knowledge assistant in the dashboard, draws on the brand knowledge base reachable via the `knowledge_list` / `knowledge_create` / `knowledge_get` / `knowledge_update` / `knowledge_delete` tools. **Social Performance**, the post-performance analytics dashboard, has no MCP tool surface — point the user to the PostKing dashboard for it.

## How to Use

### Pick the active brand

1. `list_brands({})` — if there's exactly one brand, it's active; continue.
2. Multiple brands → ask the user which one, then `set_active_brand({ brandId })`.
3. Zero brands → stop and use the `getting-started` skill to onboard one before doing anything else.

### Generate, approve, and schedule a post

1. `generate_post({ brandId, platform, variations })` — draft variations for a platform (linkedin, x, instagram, threads, facebook).
2. Show the variations to the user; ask which one and what time.
3. `approve_post({ postId, variation, schedule })` — locks it in. This is the free-tier choke point.
4. `get_calendar({ days })` — confirm it appears in the upcoming schedule.

### Plan a content week

1. `get_weekly_schedule({ brandId })` — view the current cadence, if any.
2. `set_weekly_schedule({ brandId, monday: "linkedin:1,x:1", ..., timezone: "America/New_York", enable: true })` — define days/timezone/cadence in one call.
3. `run_weekly_schedule_day({ brandId, date })` — generate all posts for a single day (this is the **Smart Week** engine); repeat per day, or let the enabled schedule run automatically.
4. `list_posts({ status: "created" })` → `get_post({ postId })` to review each draft.
5. `approve_post({ postId, variation, schedule })` per draft.
6. `get_calendar({ days: 7 })` — verify the final week.

### Repurpose a URL, blog, or text into multi-platform posts

1. `check_social_accounts({ brandId })` — confirm which platforms are connected.
2. `repurpose_content({ brandId, sourceType: "url", sourceUrl, targetType: "social", targetPlatforms: ["linkedin", "x"] })` — PostKing crawls the URL internally; do not fetch it yourself. Returns one post per platform.
   - Text input: `sourceType: "text", sourceContent: "<text>"`.
   - From an existing blog: `sourceType: "blog", sourceBlogId: <articleId>`.
   - From an existing post: `sourceType: "social_post", sourcePostId: <postId>`.
   - Per-platform voice can be passed as a platform→voiceId map.
3. `get_post({ postId })` per generated post — inspect before scheduling.
4. `approve_post({ postId, schedule })` — free-tier choke point.
5. `get_calendar({})` — confirm.

### Generate and publish a blog

1. `list_publications({ brandId })` — find an existing publication, or `create_publication({ brandId, title, description })` to make one.
2. `generate_blog_post({ publicationId, topic, keywords, length, voiceProfileId })` — async; returns an operation. Poll `get_job` until `state` is `completed`/`failed`.
3. `get_blog_article({ articleId })` — review the draft.
4. Iterate as needed: re-run `generate_blog_post` with refined topic/keywords, `update_blog_article({ articleId, ... })` for direct edits, or `rewrite_text({ text, platform: "blog" })` to polish a section.
5. Publish:
   - PostKing-hosted: `publish_blog_article({ articleId })`.
   - External connections (WordPress / Medium / Substack, connected via the dashboard): `list_publishing_connections({ brandId })` to find connection IDs, then `publish_blog_article({ articleId, connectionIds: [...] })`.
6. `get_blog_status({ articleId })` — confirm `status: published` and get the live URL(s), including any external platform URLs.

### Build and publish a landing page

1. `generate_landing_page({ brandId, topic, slug })` — async; returns a slug + operationId. Poll `get_job`.
2. `view_landing_page({ slug })` — preview.
3. `edit_landing_page({ slug, instructions })` for a targeted AI edit pass, or `vibe_edit_landing_page({ slug, instructions, scope })` + poll `get_vibe_edit_status({ slug, operationId })` for a fuller vibe edit. For a specific side-page section: `set_side_page_section({ slug, sideKey, sectionId, instructions })`.
4. Optional side pages: `generate_side_page({ slug, type })` — types include `pricing`, `features`, `legal`, `case-study`.
5. Optional custom domain: `add_domain({ domain })` → follow the printed DNS/TXT instructions at the registrar → `verify_domain({ domain })` once live → `connect_domain_to_publication({ domainId, target: "lp:<slug>" })`.
6. `publish_landing_page({ slug })` — free-tier choke point. Surface the returned `webUrl` to the user.

### Pick a visual for a post

1. `generate_post_visual_options({ postId, platform })` — always run this first. Relay the numbered options to the user, especially the recommended one and any inline card/quote text and colors — the params returned ARE the spec, no need to render anything yourself.
2. `pick_post_visual({ postId, platform, pick: N })` — submit the user's choice by index. Don't hand-construct style/variant/asset parameters; `pick` resolves the right params from the cache `generate_post_visual_options` just wrote.
3. "Show me more" → `regenerate_post_visual({ postId, platform, loadExternal: true })` pulls additional stock results, then re-run `generate_post_visual_options`.
4. `clear_post_visual({ postId, platform })` to remove a pick. For carousels: `generate_post_carousel({ postId })` renders directly from the post's cards — no uploaded asset required.

### Trends inspiration

`trends_list({ niche, days })` — niches include `ai-saas`, `marketing`, `web3`. The crawler runs every 3 days; default window is 3 days. Feed results into `generate_post` or `repurpose_content` for hook inspiration.

## CLI fast path

If a shell is available, the same flows via `pking` (npm package `postking-cli`; see `references/install.md` if not installed):

| Goal | Command |
|---|---|
| Pick active brand | `pking brand list` / `pking brand set <brandId>` |
| Generate a post | `pking posts generate --platform linkedin --variations 3` |
| Approve & schedule | `pking posts approve <postId> --variation 2 --schedule 2026-05-01T14:00:00Z` |
| View calendar | `pking posts calendar --days 7` |
| Plan a content week | `pking weekly-schedule set --monday "linkedin:1,x:1" --timezone America/New_York --enable` then `pking weekly-schedule run-day --date <YYYY-MM-DD>` |
| Repurpose a URL | `pking repurpose --source-type url --source-url <url> --target-type social --target-platforms linkedin,x` |
| Generate a blog | `pking blogs generate --publication <id> --topic "..." --keywords "kw1,kw2" --wait` |
| Publish a blog | `pking blogs publish <articleId>` (or `--connections <id1,id2>` for external platforms) |
| Generate a landing page | `pking lp generate --topic "..."` |
| Edit / vibe-edit a landing page | `pking lp edit <slug> --instructions "..."` / `pking lp vibe <slug> --instructions "..." --wait` |
| Custom domain | `pking domains add <domain>` → `pking domains verify <domain>` → `pking domains connect <id> --target lp:<slug>` |
| Publish a landing page | `pking lp publish <slug>` |
| Visual options / pick | `pking visuals options <postId> --platform <p>` → `pking visuals pick <postId> --platform <p> --pick <N>` |
| Trends | `pking trends list --niche ai-saas --days 3 --json` |

For the full command catalog, read `references/commands.md`, or run `pking --help` and `pking <group> --help`.

## Expected Output

- Scheduled/published posts, blog articles, or landing pages, each with a confirmed live or dashboard URL.
- For repurposing and content-week flows: a batch of platform-tailored drafts, reviewed and approved by the user.
- For visuals: a post with a picked image/card/quote/carousel attached.

## Troubleshooting / Errors to Expect

- **`UNAUTHORIZED`** — the session expired. Re-authenticate via the `getting-started` skill (device-flow login or reconnect the connector). Tell the user once; never silently loop.
- **`RATE_LIMITED`** — back off at least 30 seconds before retrying; the error envelope's `retryAfter` is authoritative when present.
- **`INSUFFICIENT_CREDITS`** — reply in one short line with the `checkoutUrl` from the error envelope and stop, or hand off to the `getting-started` skill's billing section (`billing_list_packs` → `billing_topup`). Do not retry, do not recap prior steps.
- **`FREE_CAP_REACHED`** (publish-time on free plan) — same one-line `checkoutUrl` treatment as `INSUFFICIENT_CREDITS`. No recap, no next-step promises.
- **Missing `brandId`** — run `list_brands` first. Never pick a brand silently — ask the user.
- **Async operations** — blog generation, landing-page generation/vibe-edit, and visual regeneration can take 30s–5min. Poll `get_job` (or the CLI's built-in polling) rather than assuming failure early.
- **Visuals "not found" / empty options** — almost always a parameter bug, not an empty library. Always call `generate_post_visual_options` before `pick_post_visual`; don't hand-construct style/variant/asset parameters.
- **Changing brand colors or default avatar** — there is no per-pick override; `generate_post_visual_options` surfaces a settings URL when the user wants different branding. They edit it in the dashboard, then re-run the tool to see updated params.
- **`webUrl` on success** — after post create/approve/reschedule, visual pick, blog generate/publish, or landing-page generate/update/publish, surface the returned `webUrl`/"View in browser" URL to the user; they're already logged into the dashboard.

## Verification

After setup, these should always succeed:

- `get_credits({ detail: "short" })` — returns the authenticated user's balance and free-tier status.
- `list_brands({})` — returns at least one brand (active brand marked).

If both succeed, the skill is fully operational.

## Full command reference

For the complete `pking` CLI surface (every group, every flag), read `references/commands.md` in this skill. The agent should consult it before running an unfamiliar CLI command.
