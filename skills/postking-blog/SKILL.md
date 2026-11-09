---
name: postking-blog
description: Create blog publications and generate, iterate on, and publish articles on PostKing — hosted, or pushed to WordPress, Medium, or Substack.
license: MIT-0
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

- `list_publications`, `create_publication`, `update_publication`, `set_publication_hosting_mode`, `delete_publication`, `list_blogs` — publications.
- `generate_blog_post`, `get_blog_status`, `get_job` — async generation + polling.
- `get_blog_article`, `update_blog_article`, `rewrite_text` — review and iterate.
- `list_assets`, `import_asset_from_url` — pick or import an image to set as an article's featured/header image.
- `publish_blog_article`, `list_publishing_connections`, `schedule_blog_article` — publishing.
- `list_blog_authors`, `create_blog_author`, `get_side_page_author_defaults`/`set_side_page_author_defaults` (author credit on comparison/feature side pages), `list_blog_categories`, `create_blog_category` — metadata.
- `import_blog_articles`, `delete_blog_article` — housekeeping.
- `list_content_changes`, `export_blog_article`, `export_side_page`, `export_site_index` — headless/external sync (see "External hosting + headless sync").
- `seo_check_cannibalization`, `refresh_blog_article`, `seo_cannibalization_report` — cannibalization guard (owned by the `postking-seo` skill's surface, used here when generation collides with an existing article; see below).

## Procedure

### Set up a publication

1. `list_publications({ brandId? })` (or `list_blogs`, which also returns a `statusBreakdown` of existing articles) — find an existing publication.
2. `create_publication({ title, description?, layout?, languageCode?, brandId? })` if none exists — returns a `publicationId`. `languageCode` is optional (defaults to the brand's content language) and must already be enabled on the brand's language roster — see the `postking` skill's language section; an un-enabled code is rejected with a 403.
3. `delete_publication({ publicationId, brandId? })` to remove one — only allowed when it has zero articles and no connected domain/external sync/publishing connections; the API refuses with an explanation of what's still attached otherwise (delete every article first with `delete_blog_article`, disconnect domain/connections).

### Self-hosting a blog

By default a publication's `hostingMode` is `"postking"` — PostKing serves the blog. Set it to `"self_hosted"` to have the customer's OWN app render it from PostKing's public API (`GET /api/blog-posts?blogId=<publicationId>`).

1. `set_publication_hosting_mode({ publicationId, hostingMode: "self_hosted" })` (or `update_publication` with `hostingMode`). `list_publications` shows the current `hostingMode`.
2. For a production host: `add_domain({ domain, hostingMode: "self_hosted" })` → `verify_domain({ domainId })` (TXT ownership only — no A record, the domain does NOT point at PostKing) → `connect_domain_to_publication({ publicationId, domainId })`.
3. **Mode-match rule**: the connected domain's `hostingMode` must equal the publication's. A mismatch 409s with `hosting_mode_mismatch` — call `set_publication_hosting_mode` or re-add the domain with the matching `hostingMode`, then retry.
4. Run a renderer with `POSTKING_BLOG_ID=<publicationId>`. The optional example repo https://github.com/bitsandtea/postking-selfhost reads `POSTKING_API_URL`, `POSTKING_BLOG_ID`, `SITE_URL`; any custom app against the endpoint works equally well.

### Public URL settings

Set on `create_publication` / `update_publication` (partial update; `list_publications({ detail: "medium" })` shows them plus `publicUrl` and `exampleArticleUrl`):

- `siteBaseUrl` — https origin the URLs are built on (e.g. `https://example.com`); `null` clears. Honoured with no connected domain when `hostingMode: "external"`.
- `articlePattern` — tokens `{slug}` (required), `{category}`, `{lang}`, e.g. `"/blog/{category}/{slug}/"`. Default `{pathPrefix}/{slug}`.
- `categoryPattern` — category hub path with `{category}`, e.g. `"/blog/{category}/"`. Must not be able to match the same URL as `articlePattern`: `"/blog/{category}/"` + `"/blog/{category}/{slug}/"` is fine; a 3-segment `"/blog/category/{category}/"` collides with a 3-segment article pattern and 400s.
- `uncategorizedSegment` — substituted for `{category}` on articles with no category (default `"uncategorized"`).
- `trailingSlash: boolean` (default off, slashless) — rewrites the stored `articlePattern` (and `categoryPattern`) to end/not end with `/`. Can be combined with `articlePattern`.
- `{category}` is the article's Category **slug**: `create_blog_category`, then `update_blog_article({ articleId, categoryId })`.
- Changing URLs keeps the old ones as `previousUrls` so 301s can be served.

### External hosting + headless sync

`hostingMode: "external"` = the customer's own site serves the content; PostKing only emits URLs (`siteBaseUrl` + patterns) and no domain may be connected. Set via `create_publication`/`update_publication`/`set_publication_hosting_mode`. The customer's build then syncs content:

1. First sync: `export_site_index({})` (or `list_content_changes` with no `since`).
2. Loop: `list_content_changes({ since: <stored serverTime>, types?, limit?, cursor? })` — oldest first; follow `nextCursor` while `hasMore`; store the response's `serverTime` for next time. `since` is inclusive, so dedupe by `id` + `updatedAt`/`contentHash`.
3. For each row: `export_blog_article({ id, ifContentHash? })` (or `export_side_page({ landingPageSlug: row.parentSlug, sideKey: row.slug, ifContentHash? })`). Pass the last `meta.contentHash` as `ifContentHash` to get `{ unchanged: true }` instead of the body. Payloads are full, never truncated.
4. Rows with `status: "unpublished"` or `"deleted"` tombstones (`previousUrls` included; categories too) → remove from the site / 301 from `previousUrls`.
5. Occasionally `list_content_changes({ includeIds: true })` and delete anything whose id is missing from `publishedIds`.
6. `type: "category"` rows are category hubs (carry `url`/`path`).

JSON-LD: emit ONLY `schemaGraph` from the export, as one `<script type="application/ld+json">`. Ignore the standalone `breadcrumbSchema`. The graph already holds Organization, WebSite, Blog, WebPage, BreadcrumbList, BlogPosting and FAQPage (built from the article's FAQ section). After any platform change to JSON-LD or URLs, do one full re-sync (omit `since`) — the schema can change without `updatedAt` changing.

`get_blog_article` also returns `canonicalUrl` + `schemaGraph` (and `contentResolved` when links resolve differently), but build pages from `export_*`, not from it.

### Generate a blog article

1. `generate_blog_post({ publicationId, topic, voiceProfileId?, targetLength?, primaryKeywords?, secondaryKeywords?, brandId?, language? })` — note the param names: **`primaryKeywords`/`secondaryKeywords`, not `keywords`; `targetLength` (`short`/`medium`/`long`), not `length`.** `language` (`en`/`es`/`pt-BR`/`de`/`fr`/`cs`) overrides the brand's `contentLanguage` for this article only. Async; returns `articleId` + `operationId`.
2. Poll `get_blog_status({ articleId })`, or `get_job({ pollUrl: operationId, brandId, wait: true })`, until status is `completed` (or `failed`).
3. `get_blog_article({ articleId })` — review the draft (full content, SEO fields, `previewUrl`, `editUrl`, and the current `featuredImageUrl`/`featuredImageAlt`/`featuredImageDescription`).

**Cannibalization guard (automatic).** `generate_blog_post` runs a cannibalization check server-side before writing — usually fine to just call it directly. If the job fails with a `cannibalizationGuard` payload (`{ decision, reason, matches, targetArticleId?, conflictingBriefId? }`):
- `decision: "refresh"` — an existing article already covers this topic/keyword closely enough. Call `refresh_blog_article({ blogId: targetArticleId, topic?, instructions? })` instead of re-running `generate_blog_post`. Don't silently retry the write.
- `decision: "blocked"` — an in-flight brief (`conflictingBriefId`) already targets this keyword; wait for it to resolve rather than generating a second one.
- Explicit preflight (optional): `seo_check_cannibalization({ title, primaryKeyword, language?, briefId?, exactOnly? })` returns the same decision shape before you even call `generate_blog_post`.
- Only pass `generate_blog_post({ ..., forceCannibalization: true })` when the user has explicitly asked for a duplicate/second article on the topic anyway — never decide this on your own initiative.
- `seo_cannibalization_report({ brandId })` — run periodically, or when asked "do we have duplicate/competing articles?", to audit the existing corpus for near-duplicates the guard didn't catch (it only guards new writes). Read-only, zero writes; proposes a canonical winner per duplicate group.

### Review and iterate

- `update_blog_article({ articleId, title?, content?, excerpt?, status?, metaTitle?, metaDescription?, authorId?, categoryId?, featuredImageUrl?, featuredImageAlt?, featuredImageDescription?, cta?, sidePageInfo? })` for direct edits — only the fields you pass are changed.
- `rewrite_text({ text, voice?, platform: "blog" })` to polish a specific passage, then paste the result back in via `update_blog_article`.
- Re-run `generate_blog_post` with a refined `topic`/`primaryKeywords` for a fresh pass instead of patching piecemeal changes.

### Set or replace the featured (header) image

The header image can be set at generation time (`generate_blog_post`'s `generateAiImage`/`attachVisualAsset`/`selectedAssetId`), but to add or swap it on an **existing** article:

1. Get an image URL — `list_assets({ brandId? })` and copy an asset's `url`/`fileUrl` (returned as an absolute CDN URL, e.g. `https://cdn.postking.app/assets/<brandId>/images/x.png`); or `import_asset_from_url({ url })` to pull an external image into the brand library first; or use any public image URL / `data:` URI directly.
2. `update_blog_article({ articleId, featuredImageUrl, featuredImageAlt?, featuredImageDescription? })` — sets the header image. Pass `featuredImageUrl: ""` to remove it. An external URL is auto-downloaded to brand storage when the article is published.
3. `get_blog_article({ articleId })` — the response echoes `featuredImageUrl`/`featuredImageAlt`/`featuredImageDescription`, so you can confirm the change stuck.

### Edit the CTA (call-to-action) block

The CTA lives in structured data (`sidePageInfo` on the article), **not** in the article body — never write CTA markup into `content` to change it; that gets stripped/ignored.

- Simple case — `update_blog_article({ articleId, cta: { url, label?, headline?, body? } })`:
  - `url` → CTA link target (required when enabling a CTA)
  - `label` → CTA button text
  - `headline` → CTA block headline
  - `body` → CTA block body copy
- Disable/remove the CTA — `update_blog_article({ articleId, cta: { enabled: false } })` (equivalent to `sidePageInfo: null`).
- Advanced case — link the CTA to an existing side page instead of a raw URL: `update_blog_article({ articleId, sidePageInfo: { id } })` (id from `list_side_pages`). `cta` and `sidePageInfo` are mutually exclusive — pass only one per call.
- Example:
  ```
  update_blog_article({
    articleId: "art_123",
    cta: {
      url: "https://example.com/pricing",
      label: "See pricing",
      headline: "Ready to get started?",
      body: "Compare plans and find the right fit."
    }
  })
  ```
- **Malformed CTA payloads are now rejected server-side** (strict validation) instead of being silently persisted — pass exactly the documented fields (misspelled keys, e.g. `ctaUrl` instead of `url`, will error rather than silently doing nothing).

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
| Self-host a publication | `pking publications set-hosting <publicationId> postking\|self_hosted` |
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
- **The CTA is structured data, not article HTML.** Use `cta`/`sidePageInfo` on `update_blog_article`, never embed CTA markup in `content` — it will be stripped on write, not rendered. Malformed `cta`/`sidePageInfo` shapes (unknown keys, empty `{}`, enabling without a `url`) are rejected with a 400, not silently dropped.
- **`forceCannibalization: true` bypasses the guard entirely** — only pass it when the user has explicitly asked for a duplicate article anyway, never to work around a `refresh`/`blocked` decision on your own initiative.
- **The featured image is a URL/path, not an asset ID.** `featuredImageUrl` takes an image URL, a brand-asset URL (the `fileUrl` from `list_assets` — now an absolute CDN URL, e.g. `https://cdn.postking.app/assets/<brandId>/images/x.png`), or a `data:` URI — **not** an asset ID. Bare legacy paths (`/assets/<brandId>/...`) are still accepted as input for backward compatibility. (Contrast generation, where `generate_blog_post` uses `selectedAssetId`, an asset ID.) Pass an empty string to clear it.

- **A domain's `hostingMode` must match the publication's `hostingMode` to connect.** `postking` pairs with `postking`, `self_hosted` with `self_hosted`; a mismatch 409s (`hosting_mode_mismatch`) on `connect_domain_to_publication` / `update_publication` — see "Self-hosting a blog".

- **`articlePattern` needs `{slug}`; `categoryPattern` must not overlap it.** A same-depth collision (e.g. `/blog/category/{category}/` vs `/blog/{category}/{slug}/`) 400s.
- **Use `trailingSlash`, don't hand-edit the slash** into one pattern and forget the other — the toggle rewrites both.
- **Headless sites: one `schemaGraph` only.** Don't also emit `breadcrumbSchema` (duplicate BreadcrumbList), and re-sync fully (no `since`) after platform JSON-LD/URL changes.
- **Never build headless pages from `get_blog_article`** — use `export_blog_article` (absolute URLs, resolved links, `meta.contentHash`).

## Verification

- `get_credits({ detail: "short" })` — confirms auth and balance.
- `list_publications({})` — returns at least one publication once set up (or an empty list if none created yet, which is expected pre-setup).
