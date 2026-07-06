---
name: postking-blog
description: Create blog publications and generate, iterate on, and publish blog articles on PostKing — PostKing-hosted or pushed to connected external platforms like WordPress, Medium, or Substack — and check generation/publish status.
license: MIT
compatibility: "Works with any MCP-compatible client connected to postking-mcp (local stdio or hosted at https://mcp.postking.app/mcp); the pking CLI is an optional fast path when a shell is available."
metadata:
  icon: https://raw.githubusercontent.com/bitsandtea/postking-skills/main/assets/icons/postking-blog.svg
  category: marketing
---

# PostKing Blog

Handles the blog lifecycle on PostKing: publications (the containers articles live under), AI article generation, review/editing, and publishing — either to the PostKing-hosted blog or out to connected external platforms. Requires an active brand — use the `postking` skill's brand-pick flow first if one isn't set.

## When to Use

Use this skill when the user wants to create or manage a blog publication; generate, review, edit, or rewrite a blog article; publish an article (hosted or external); check on an in-progress generation; or manage blog authors/categories.

## When NOT to Use

- Social posts, content weeks, or repurposing to social → `postking-social`.
- Landing pages or side pages → `postking-landing-pages`.
- The SEO/GEO keyword-to-roadmap pipeline (seeds → keywords → clusters → briefs) that eventually produces briefs for articles → `postking-seo`. This skill covers direct/manual blog generation and publishing, not that pipeline's brief-to-article automation.
- Saved voice profile management or a standalone de-slop pass → `postking-brand-voice`.
- No active brand yet → `postking-getting-started`.

## Minimal tool subset

- `list_publications`, `create_publication`, `update_publication`, `list_blogs` — publications.
- `generate_blog_post`, `get_blog_status`, `get_job` — async generation + polling.
- `get_blog_article`, `update_blog_article`, `rewrite_text` — review and iterate.
- `publish_blog_article`, `list_publishing_connections`, `schedule_blog_article` — publishing.
- `list_blog_authors`, `create_blog_author`, `list_blog_categories`, `create_blog_category` — metadata.
- `import_blog_articles`, `delete_blog_article` — housekeeping.

## Procedure

### Set up a publication

1. `list_publications({ brandId? })` (or `list_blogs`, which also returns a `statusBreakdown` of existing articles) — find an existing publication.
2. `create_publication({ title, description?, layout?, brandId? })` if none exists — returns a `publicationId`.

### Generate a blog article

1. `generate_blog_post({ publicationId, topic, voiceProfileId?, targetLength?, primaryKeywords?, secondaryKeywords?, brandId? })` — note the param names: **`primaryKeywords`/`secondaryKeywords`, not `keywords`; `targetLength` (`short`/`medium`/`long`), not `length`.** Async; returns `articleId` + `operationId`.
2. Poll `get_blog_status({ articleId })`, or `get_job({ pollUrl: operationId, brandId, wait: true })`, until status is `completed` (or `failed`).
3. `get_blog_article({ articleId })` — review the draft (full content, SEO fields, `previewUrl`, `editUrl`).

### Review and iterate

- `update_blog_article({ articleId, title?, content?, excerpt?, status?, metaTitle?, metaDescription?, authorId?, categoryId? })` for direct edits — only the fields you pass are changed.
- `rewrite_text({ text, voice?, platform: "blog" })` to polish a specific passage, then paste the result back in via `update_blog_article`.
- Re-run `generate_blog_post` with a refined `topic`/`primaryKeywords` for a fresh pass instead of patching piecemeal changes.

### Publish

- **PostKing-hosted:** `update_blog_article({ articleId, status: "published" })` makes it live on the brand's PostKing blog immediately.
- **Scheduled hosted publish:** `schedule_blog_article({ articleId, scheduledAt })` — future ISO 8601 datetime; auto-publishes (and auto-pushes to any `autoPublish`-flagged external connections) when the time arrives.
- **External (WordPress / Medium / Substack, connected via the dashboard):** `list_publishing_connections({ publicationId })` to find connection IDs, then `publish_blog_article({ articleId, connectionIds: [...] })`.
- `get_blog_status({ articleId })` — confirm the final status and get the live URL(s).

## CLI fast path

| Goal | Command |
|---|---|
| List publications / articles | `pking blogs list [--status draft\|published]` / `pking publications list` |
| Create a publication | `pking publications create --title "..." [--description "..."]` |
| Generate a blog | `pking blogs generate --publication <id> --topic "..." [--keywords "kw1,kw2"] [--voice <id>] [--length short\|medium\|long] --wait` |
| Check generation status | `pking blogs status <articleId>` |
| View an article | `pking blogs view <articleId>` |
| Publish (hosted, or external via connections) | `pking blogs publish <articleId> [--connections <id1,id2>]` |
| Delete | `pking blogs delete <articleId>` |

For the full command catalog, use the `postking` skill's `references/commands.md`, or run `pking blogs --help` / `pking publications --help`.

## Pitfalls

- **`generate_blog_post` param names**: `primaryKeywords`/`secondaryKeywords` and `targetLength`, not `keywords`/`length`. Passing the wrong names silently drops them (extra zod fields are ignored, not errored).
- **Hosted publish vs. external publish are different tools.** `publish_blog_article` requires a non-empty `connectionIds` array — it's for pushing to external platforms, not for making the PostKing-hosted copy live. Use `update_blog_article({ status: "published" })` for that.
- **No `pking blogs auto-assign-cta` CLI command exists.** If the user wants a CTA auto-assigned to a piece of content, that capability is the MCP tool `seo_auto_assign_cta` (owned by the `postking-seo` skill's surface) — don't invent a CLI equivalent.
- **Generation is async and can take a couple of minutes.** Poll `get_blog_status` or `get_job` rather than assuming failure early; `get_job` supports `wait: true` to block instead of polling manually.
- **`update_blog_article` is a partial update.** Omitted fields are left unchanged — no need to resend the whole article to change one field.

## Verification

- `get_credits({ detail: "short" })` — confirms auth and balance.
- `list_publications({})` — returns at least one publication once set up (or an empty list if none created yet, which is expected pre-setup).
