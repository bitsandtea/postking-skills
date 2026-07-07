---
name: postking-landing-pages
description: Generate, edit, vibe-edit, and publish landing pages on PostKing, including comparison/side pages and custom domains, for launches and campaigns.
license: MIT-0
compatibility: "Works with any MCP-compatible client connected to postking-mcp (local stdio or hosted at https://mcp.postking.app/mcp); the pking CLI is an optional fast path when a shell is available."
metadata:
  icon: https://raw.githubusercontent.com/bitsandtea/postking-skills/main/assets/icons/postking-landing-pages.svg
  category: marketing
---

# PostKing Landing Pages

Handles the landing-page lifecycle on PostKing: AI generation, targeted or full AI ("vibe") edits, side pages under a parent page, custom domains, and publishing. Requires an active brand — use the `postking` skill's brand-pick flow first if one isn't set.

## When to Use

Use this skill when the user wants to generate a new landing page, edit or AI-rewrite one, add or edit a side page (a comparison, text, or landing-style sub-page under a parent LP), attach a custom domain, or publish/preview a page.

## When NOT to Use

- Social posts or content weeks → `postking-social`.
- Blog articles → `postking-blog`.
- The SEO/GEO roadmap pipeline that decides *what* pages to brief (seeds → keywords → clusters → roadmap items) → `postking-seo`. This skill covers generating and editing the pages themselves, including side pages fed by a roadmap brief.
- Saved voice profile management or a standalone de-slop pass → `postking-brand-voice`.
- No active brand yet → `postking-getting-started`.

## Minimal tool subset

- `list_landing_pages`, `generate_landing_page`, `view_landing_page`, `get_job` — create and inspect.
- `edit_landing_page`, `set_landing_page`, `regenerate_landing_page`, `vibe_edit_landing_page`, `get_vibe_edit_status` — editing.
- `list_side_pages`, `generate_side_page`, `view_side_page`, `edit_side_page`, `set_side_page_section`, `set_side_page_state` — side pages.
- `add_domain`, `verify_domain`, `list_domains` — custom domains (partial support — see Pitfalls).
- `publish_landing_page`, `delete_landing_page` — publish/lifecycle.

## Procedure

### Generate and publish a landing page

1. `generate_landing_page({ brandId, topic, slug? })` — async; creates the LP record and kicks off AI content generation. Returns `{ slug, operationId, pollUrl }`. Poll with `get_job({ pollUrl: operationId, brandId, wait: true })` until `state` is `completed` (or `failed`/`partially_failed`/`cancelled`).
2. `view_landing_page({ slug })` — preview the generated content.
3. `publish_landing_page({ slug })` — free-tier choke point. Surface the returned `webUrl` to the user.

### Edit a landing page

- Targeted metadata/instructions patch: `edit_landing_page({ slug, title?, instructions? })`.
- Manual overwrite (no AI): `set_landing_page({ slug, title?, content?, metadata? })` — versioned; returns a `versionId`.
- Full AI regeneration pass, optionally scoped to specific sections: `regenerate_landing_page({ slug, voiceProfileId?, instructions?, sections? })`.
- Natural-language AI edit ("vibe edit"): `vibe_edit_landing_page({ slug, instructions, scope?, sectionId? })` — `scope` is `"headline" | "cta" | "full"`. Poll `get_vibe_edit_status({ slug, operationId })` (or `get_job`) until done.

### Side pages

1. `list_side_pages({ slug })` — see what already exists under a parent landing page.
2. `generate_side_page({ slug, key, prompt?, brief?, keywords?, sidePageType? })` — `key` is the required URL-slug fragment for the side page. `sidePageType` is `"landing" | "text" | "comparison"` (defaults to `"landing"`) — there is **no `type` param**, and `"comparison"` requires a persisted `briefId`. Async for most cases (returns `operationId`); some comparison briefs run synchronously and return `sidePageId` directly with no `operationId`. Poll `get_job` when an `operationId` is returned.
3. `view_side_page({ slug, sideKey })` — inspect content and rendered HTML.
4. `edit_side_page({ slug, sideKey, instructions? })` for page-level instructions, or `set_side_page_section({ slug, sideKey, sectionId, content?, instructions? })` for a single section.
5. `set_side_page_state({ slug, sideKey, published })` — publish or unpublish it.

### Custom domain (partial support)

1. `add_domain({ domain, primaryContentType: "landing_page", brandId? })` — registers the domain and returns a DNS TXT record to add at the registrar.
2. `verify_domain({ domainId })` — checks DNS once the record is added; `domainId` comes from the `add_domain` response or `list_domains`.
3. **There is currently no MCP or CLI tool to connect a verified domain to a specific landing page.** `connect_domain_to_publication` only attaches a domain to a *blog* publication (it takes a `publicationId`, not a landing-page slug) — using it for a landing page is a no-op/wrong target, not a substitute. Tell the user to finish the connection from the landing page's own settings in the PostKing dashboard.

## CLI fast path

| Goal | Command |
|---|---|
| List / generate | `pking lp list` / `pking lp generate --topic "..." [--slug <slug>] [--voice <id>]` |
| View / preview | `pking lp view <slug>` |
| Edit (metadata or AI pass) | `pking lp edit <slug> [--title "..."] [--instructions "..."]` |
| Vibe edit | `pking lp vibe <slug> --instructions "..." [--scope full\|section --section-id <id>] --wait` then `pking lp vibe status <slug> <operationId>` |
| Side pages | `pking lp side list <slug>` / `pking lp side edit <slug> <sideKey> --instructions "..."` — generation is currently MCP-only, see Pitfalls |
| Side page section | `pking lp side section <slug> <sideKey> --id <sectionId> [--content "..." | --file <path>] [--instructions "..."]` |
| Publish / unpublish a side page | `pking lp side state <slug> <sideKey> --publish` (or `--unpublish`) |
| Custom domain (register + verify only) | `pking domains add <domain>` → `pking domains verify <domain>` |
| Publish the landing page | `pking lp publish <slug>` |

For the full command catalog, use the `postking` skill's `references/commands.md`, or run `pking lp --help`.

## Pitfalls

- **`generate_side_page` has no `type` param** — it's `sidePageType`, and the only valid values are `"landing"`, `"text"`, `"comparison"`. Don't invent other values (e.g. "pricing", "features", "legal") as `sidePageType` — those aren't recognized side-page types by this tool.
- **`key` is required on `generate_side_page`** — it's the side page's URL-slug fragment, not optional metadata.
- **`pking lp side generate <slug> --type <type>` is currently broken** — the inner route ignores the `<slug>` path param and requires `name`/`slug`/`landingPageId` in the request body, so it always 400s. Use the MCP tool `generate_side_page({ slug, key, sidePageType })` instead.
- **Custom domains for landing pages stop short of dashboard-only.** Registering (`add_domain`) and verifying (`verify_domain`) work via MCP/CLI, but attaching the verified domain to a specific landing page does not — don't tell the user `connect_domain_to_publication` will do it; that tool is blog-only.
- **`pking domains connect <id> --target lp:<slug>` is not a working command** — the CLI accepts the flag but the endpoint it calls does not exist on the server; don't suggest it.
- **Async operations can take 30s–5min.** Poll `get_job` (or the CLI's built-in `--wait`) rather than assuming failure early.
- **Comparison-type side pages may return synchronously** with `sidePageId` and no `operationId` — don't poll `get_job` with an ID you don't have; check the immediate response first.

## Verification

- `get_credits({ detail: "short" })` — confirms auth and balance.
- `list_landing_pages({})` — returns the brand's landing pages (or an empty list if none created yet, which is expected pre-setup).
