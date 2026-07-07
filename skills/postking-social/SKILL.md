---
name: postking-social
description: Generate, approve, and schedule social posts on PostKing across LinkedIn, X, Instagram, Threads, and Facebook — plus content weeks and repurposing.
license: MIT-0
compatibility: "Works with any MCP-compatible client connected to postking-mcp (local stdio or hosted at https://mcp.postking.app/mcp); the pking CLI is an optional fast path when a shell is available."
metadata:
  icon: https://raw.githubusercontent.com/bitsandtea/postking-skills/main/assets/icons/postking-social.svg
  category: marketing
---

# PostKing Social

Handles the social-post lifecycle on PostKing: generating AI drafts, approving/scheduling them, running the recurring Smart Week cadence, repurposing existing content into new posts, and attaching visuals. Requires an active brand — use the `postking` skill's brand-pick flow first if one isn't set.

## When to Use

Use this skill when the user wants to draft, approve, schedule, or reschedule a social post; set up or run a weekly/recurring posting cadence; turn a URL, blog article, or piece of text into platform posts; attach or change a post's image/card/quote/carousel; or browse trending posts for hook inspiration.

## When NOT to Use

- Blog articles (writing, publishing, WordPress/Medium/Substack) → `postking-blog`.
- Landing pages or side pages → `postking-landing-pages`.
- Multi-channel campaign briefs and strategy (Storylines) → `postking-storylines`.
- Subreddit-specific matching or native Reddit rewrites → `postking-reddit`.
- Managing saved voice profiles themselves, or a standalone de-slop/AI-detection pass → `postking-brand-voice`.
- No active brand yet, or first-time social account connection → `postking-getting-started`.

## Minimal tool subset

- `list_brands`, `set_active_brand` — brand context.
- `check_social_accounts` — confirm connected platforms before posting.
- `generate_post`, `approve_post`, `schedule_post`, `reschedule_post`, `cancel_post`, `get_post`, `list_posts`, `get_calendar` — the core post lifecycle.
- `get_weekly_schedule`, `set_weekly_schedule`, `run_weekly_schedule_day` — recurring cadence (Smart Week).
- `repurpose_content` — turn a URL/text/blog/post into new social drafts.
- `generate_post_visual_options`, `pick_post_visual`, `regenerate_post_visual`, `clear_post_visual`, `generate_post_carousel` — visuals.
- `trends_list` — trending-post hooks.

## Procedure

### Generate, approve, and schedule a post

1. `generate_post({ platform, variations?, theme, voice?, brandId? })` — ALWAYS pass `theme` with the specific topic/angle/tone the user wants; omitting it produces a random brand theme. This call polls until generation completes and already saves the result to a single `postId` (variation 1 is the primary, saved content — all variations live under that one `postId`, never call it again to "get the others").
2. Show the variation(s) to the user; ask which one (if more than one) and what time.
3. `approve_post({ postId, scheduledAt, timezone? })` — `scheduledAt` is a future ISO 8601 datetime. This is the free-tier choke point. There is **no `variation` param on this MCP tool** — via MCP you always approve the post's current saved content (variation 1). Picking a different generated variation to schedule is a CLI-only affordance (`pking posts approve <id> --variation N`).
4. `get_calendar({ days })` — confirm it appears in the upcoming schedule.

### Plan a content week (Smart Week)

1. `get_weekly_schedule({ brandId? })` — view the current cadence, if any.
2. `set_weekly_schedule({ enabled?, leadTimeDays?, timezone?, voiceProfileId?, dayConfigs, brandId? })` where `dayConfigs` is an array of `{ dayOfWeek (0=Sun…6=Sat), mediums: [{ medium, postsPerDay }] }`. There are **no `monday`/`tuesday`/… params** and the flag is `enabled`, not `enable` — those are CLI-only sugar.
3. `run_weekly_schedule_day({ date, brandId? })` — `date` is `YYYY-MM-DD`. This is the Smart Week engine; call it per day, or leave the schedule `enabled` to run automatically.
4. `list_posts({ status: "created" })` → `get_post({ postId })` to review each draft.
5. `approve_post({ postId, scheduledAt, timezone? })` per draft.
6. `get_calendar({ days: 7 })` — verify the final week.

### Repurpose a URL, blog, or text into multi-platform posts

1. `check_social_accounts({ brandId? })` — confirm which platforms are connected.
2. `repurpose_content({ brandId, sourceType: "url", sourceUrl, targetType: "social", targetPlatforms: ["linkedin", "x"] })` — PostKing crawls the URL internally; do not fetch it yourself.
   - Text input: `sourceType: "text", sourceContent: "<text>"`.
   - From an existing blog: `sourceType: "blog", sourceBlogId: <articleId>`.
   - From an existing post: `sourceType: "social_post", sourcePostId: <postId>`.
   - Optional: `angle`, `variations`, `voiceProfileIds` (single ID applies to all platforms, or `["x:id1","linkedin:id2"]` per platform).
3. `get_post({ postId })` per generated post — inspect before scheduling.
4. `approve_post({ postId, scheduledAt })` — free-tier choke point.
5. `get_calendar({})` — confirm.

### Pick a visual for a post

1. `generate_post_visual_options({ postId, platform })` — always run this first. Relay the numbered options to the user, especially the recommended one — the returned params ARE the spec, no need to render anything yourself.
2. `pick_post_visual({ postId, platform, style?, variant?, assetId?, slot?, kind? })` — pass the chosen option's fields verbatim from step 1's response. There is **no numeric `pick` param on the MCP tool** (that's CLI-only sugar); you must supply one of `style`, `assetId`, or `slot`, or the tool errors. Pass `kind: "quote"` (or `"card"`/`"photo"`/etc.) when the chosen option is a quote/card template style — omitting `kind` defaults to `"card"` and rejects quote styles.
3. "Show me more" → `regenerate_post_visual({ postId, platform, loadExternal: true })` pulls additional stock results, then re-run `generate_post_visual_options`.
4. `clear_post_visual({ postId, platform })` to remove a pick. For carousels: set cards first (`set_post_cards`/`edit_post_card`), then `generate_post_carousel({ postId })`.

### Trends as hooks

`trends_list({ niche?, platform?, days?, limit?, sort? })` — niches: `ai-saas`, `marketing`, `web3` (default `ai-saas`); platform currently only `x`. The crawler runs every 3 days; default window is 3 days. Feed results into `generate_post`'s `theme` or `repurpose_content`'s `angle` for hook inspiration.

## CLI fast path

| Goal | Command |
|---|---|
| Pick active brand | `pking brand list` / `pking brand set <brandId>` |
| Generate a post | `pking posts generate --platform linkedin --variations 3 --theme "..."` |
| Approve & schedule (pick a variation) | `pking posts approve <postId> --schedule 2026-08-01T14:00:00Z --variation 2 --timezone America/New_York` |
| View calendar | `pking posts calendar --days 7` |
| Plan a content week | `pking weekly-schedule set --monday "linkedin:1,x:1" --timezone America/New_York --enable` then `pking weekly-schedule run-day --date <YYYY-MM-DD>` |
| Repurpose a URL | `pking repurpose --source-type url --source-url <url> --target-type social --target-platforms linkedin,x` |
| Visual options / pick | `pking visuals options <postId> --platform <p>` → `pking visuals pick <postId> --platform <p> --pick <N>` |
| Trends | `pking trends list --niche ai-saas --days 3 --json` |

For the full command catalog, use the `postking` skill's `references/commands.md`, or run `pking --help` / `pking <group> --help`.

## Pitfalls

- **`approve_post` has no `variation` param via MCP.** If the user wants a specific generated variation scheduled (not variation 1), that's a CLI-only affordance (`--variation N`); via MCP the only way is to work from the CLI, or use the visual editor link (`editInVisualEditor`, returned by `generate_post`/`get_post`) to edit the saved content directly.
- **`set_weekly_schedule` needs `dayConfigs`, not per-day flags.** Passing `monday`/`tuesday`/etc. or `enable` (instead of `enabled`) will fail validation.
- **`pick_post_visual` needs an explicit `style`/`assetId`/`slot`.** There is no index-based `pick` param — always call `generate_post_visual_options` first and copy its fields.
- **Omitting `theme` on `generate_post` gives a random topic.** Always pass it when the user wants specific content.
- **Visuals are never auto-attached.** `generate_post` prepares visual options but does not attach one — nothing changes until `pick_post_visual` is called.
- **`INSUFFICIENT_CREDITS` / `FREE_CAP_REACHED`** — surface the error envelope's `checkoutUrl` in one line and stop; don't retry or recap.

## Verification

- `get_credits({ detail: "short" })` — confirms auth and balance.
- `check_social_accounts({})` — confirms at least one connected platform.
- `list_posts({ status: "created", limit: 1 })` — confirms the brand/posts API is reachable.
