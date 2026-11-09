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

- `list_landing_pages`, `generate_landing_page`, `view_landing_page`, `get_job` — create and inspect (`get_job` is for the async `generate_landing_page`/`regenerate_landing_page` ops only — vibe-edit sessions are polled with `get_vibe_edit_status`, not `get_job`).
- `import_landing_page_bundle` — import a static-build bundle (entry HTML + its assets) whose files are already hosted at URLs; for a local folder on disk, use the `postking-cli` `import-bundle` command instead — see "Import a static-build bundle".
- `edit_landing_page`, `set_landing_page_section`, `set_lp_section_layout`, `regenerate_landing_page`, `vibe_edit_landing_page`, `get_vibe_edit_status`, `apply_vibe_edit`, `restore_lp_version`, `delete_lp_version` — editing.
- `list_lp_versions`, `view_lp_version`, `view_lp_draft` — landing-page version-history reads (`restore_lp_version`/`delete_lp_version` above are the writes) — see "Version history".
- `list_side_pages`, `generate_side_page`, `import_side_page_html`, `view_side_page`, `edit_side_page`, `set_side_page_section`, `set_side_page_state` — side pages. Author credit: `edit_side_page({ authorId?, hideAuthor? })` per page (`authorId: null` = brand default, `hideAuthor: true` = brand credited instead), `get_side_page_author_defaults` / `set_side_page_author_defaults({ comparison?, feature? })` per kind (author id, `"none"`, or null = blog default). Structured-data business type: `get_business_type` / `set_business_type({ kind, schemaType? } | { reset: true })`.
- `list_block_types`, `add_block`, `edit_block`, `delete_block`, `reorder_blocks` — block-model editing for `sidePageType: "custom"` side pages — see "Custom side pages (block model)".
- `list_side_page_versions`, `view_side_page_version`, `restore_side_page_version`, `delete_side_page_version` — side-page version history — see "Version history".
- `get_landing_page_translations`, `create_landing_page_translation` — landing-page language variants — see "Landing page translations".
- `list_asset_slots`, `assign_asset_to_slot` — discover a page's asset slots and assign/reassign/clear the asset in a slot (works for main LPs and side pages — see "Assign or reassign an asset to a slot").
- `add_domain`, `verify_domain`, `list_domains`, `delete_domain`, `connect_landing_page_domain` — custom domains — see "Custom domain".
- `set_landing_page_distribution` — hosted (PostKing serves it, default) vs. self-hosted (the customer's own infra serves it) — see "Self-hosting".
- `publish_landing_page`, `delete_landing_page` — publish/lifecycle.
- `tool_list`, `tool_get`, `tool_create`, `tool_generate`, `tool_update`, `tool_publish`, `tool_delete`, `tool_preview`, `tool_runs`, `tool_leads` — Tool pages (visitor-facing forms at `{brand domain}/tools/{slug}` that optionally call the client's own backend) — see "Tool page".

## Procedure

### Generate and publish a landing page

1. `generate_landing_page({ brandId, topic, slug?, language? })` — async; creates the LP record and kicks off AI content generation. `language` (`en`/`es`/`pt-BR`/`de`/`fr`/`cs`) overrides the brand's `contentLanguage` for this page only. Returns `{ slug, operationId, pollUrl }`. Poll with `get_job({ pollUrl: operationId, brandId, wait: true })` until `state` is `completed` (or `failed`/`partially_failed`/`cancelled`).
2. `view_landing_page({ slug })` — preview the generated content.
3. `publish_landing_page({ slug })` — free-tier choke point. Surface the returned `webUrl` to the user.
4. Once the page has had visitors, `search_page_journey({ brandId, host, path })` shows what they do on it (where they leave, which buttons get clicked) and returns plain-language `suggestions` for what to change. Call it after a landing-page change to check whether it helped, and before the next edit to decide where to focus. It is part of the `postking-seo` measure/report tools; free, and PostKing-hosted pages are tracked automatically.

### Edit a landing page

- Targeted metadata/instructions patch: `edit_landing_page({ slug, title?, instructions? })`.
- Direct field/section write (no AI): `set_landing_page_section({ slug, section, field?, value, replaceSection? })` — each call creates a new draft version. For many related changes across a page, prefer the vibe-edit flow below. `section` is one of the 12 built-ins (`hero`, `showcase`, `videos`, `howItWorks`, `features`, `categoryExplorer`, `replacesStack`, `comparisonMatrix`, `cta`, `faq`, `pricing`, `roiCalculator` — the same list `set_lp_section_layout` orders) or a top-level root (`slotMap`, `config`, `navigation`, `siteMetadata`). Freeform HTML sections are a separate address: `section: "customHtml"` with `field: "<id>"` and `value: { name?, html }` (or `replaceSection: true` to replace the whole `id -> {name?, html}` map) — the HTML is sanitized server-side and rejected (not stripped) if it breaks the `blk-*`-class/no-script rules.
- Reorder or toggle visibility of sections: `set_lp_section_layout({ slug, sectionOrder?, sectionVisibility? })` — `sectionOrder` takes the 12 built-in keys above plus any `customHtml:<id>` freeform sections in the page.
- Full AI regeneration pass, optionally scoped to specific sections: `regenerate_landing_page({ slug, voiceProfileId?, instructions?, sections? })`.
- Natural-language AI edit ("vibe edit") — this is a propose → review → apply flow, nothing changes until you apply it:
  1. `vibe_edit_landing_page({ slug, instructions, scope?, sectionId?, language? })` — `scope` is `"full" | "section"` (`sectionId` required when `scope: "section"`; example section keys: `hero`, `features`, `pricing`, `cta`, `faq`, `howItWorks`, `showcase`, `categoryExplorer`, `replacesStack`, `comparisonMatrix`, `roiCalculator`). `scope: "section"` is **hard-enforced**: any patch keys outside `sectionId` are silently dropped, not applied elsewhere. Use `scope: "full"` for any edit that spans more than one section. `language` regenerates the edited content in that language for this edit only, overriding the brand's configured content language. Returns `{ operationId }`.
  2. Poll `get_vibe_edit_status({ slug, operationId })` — **not `get_job`**, which cannot find vibe-edit sessions — until `status` is `completed` (or `failed`). The default `detail: "medium"` is the review view: `{ status, stale, changes: [{ index, path, before, after }] }`. A structurally impossible request (e.g. asking for a 6th pricing plan when the schema caps at 5) fails fast with a clear error on the status response instead of looping/repairing — read the error and adjust the instructions rather than retrying the same payload.
  3. Review the `changes` array, then `apply_vibe_edit({ slug, operationId, all? | indices? | paths? })` to apply the accepted changes to the draft as one new version. A stale session (draft moved on since the edit was proposed) returns a `stale_session` error — re-run `vibe_edit_landing_page` or retry with `force: true`.
  4. `publish_landing_page({ slug })` to make it live. `restore_lp_version({ slug, versionId })` rolls the draft back if needed.

### Import a static-build bundle

Use this instead of AI generation when the user already has a static export sitting around (`next build && next export`, Vite, Astro, a Webflow/Figma-to-code export) — either as a folder on disk, or as files already hosted somewhere. Unlike pasting raw HTML, every relative asset path in the bundle (`img/...`, `fonts/...`, `css/...`) resolves and gets rehosted instead of breaking.

- **Local folder on disk** — there is no MCP tool for this; MCP tool arguments can't carry a whole directory's worth of file bytes. Tell the user to run the `postking-cli` command directly instead: `pking lp import-bundle <folder> [--name "..."] [--entry <path>] [--convert-slug <slug>] [--side-pages]`. The CLI walks the folder itself (never following symlinks) and uploads everything in one request.
- **Files already hosted at URLs** (already-uploaded brand Assets, or any public URLs) — `import_landing_page_bundle({ files: [{path, url}, ...], entryPath?, name?, convertExistingSlug?, createMissingSidePages? })`. `files` is a manifest covering every file in the bundle, including the entry HTML itself: `path` is the file's path relative to the bundle root (forward-slash, no leading slash, e.g. `"img/home-v2/a.webp"`), `url` is where the server fetches its bytes from. Pass `entryPath` when the bundle has more than one `.html` file and which one is the entry would be ambiguous — an error result lists the candidates when it can't decide on its own.
- Both paths refuse the same file types outright, naming the offending path: executables, shell/server scripts, archives (a bundle is a folder-drop or a URL manifest, never a zip — a `.zip` inside the folder/manifest is refused like any other bad file, not auto-unzipped), `.wasm`, and server-config files. **A `.env`/`.git/`/`id_rsa`/`.npmrc`/`.netrc`/`.pem`/`.key` anywhere in the bundle aborts the ENTIRE import**, not just that one file — warn the user before pointing the CLI at a raw project folder that still has a `.git/` or `.env` sitting in it. `.map` files, `.DS_Store`, `Thumbs.db`, and other editor/OS cruft are silently skipped instead — no error, no partial failure.
- Caps: 500 files, 150MB total, 20MB per image/font/video file, 2MB per CSS file.
- Result shape matches `import_landing_page_html`'s report, plus a `report.bundle` summary: `{ filesUploaded, totalBytes, entryPath, orphaned (uploaded but never referenced by anything, not rehosted), skipped, sanitizedSvgs (SVGs that had active content like onload=/<script> stripped before hosting), malwareScan, sidePagesCreated }`.
- Pass `createMissingSidePages`/`--side-pages` to turn other `.html` files in a multi-page bundle into side pages under the new landing page, instead of importing only the detected entry page.

### Tool page

A Tool page is a visitor-facing form at `{brand domain}/tools/{slug}` that collects input, optionally forwards it to the client's own backend, renders a templated HTML result, and hands the visitor off to the client's site.

1. `tool_generate({ brandId, description, endpointUrl, endpointMethod?, apiKey?, inputsOutputs })` — async; drafts the input schema and a first-pass `endpointUrl`/`endpointMethod`/`apiKey` wiring from a natural-language description. Returns `{ operationId, status }` — poll `get_job({ pollUrl: operationId, brandId, wait: true })` until `state` is `completed`. The draft has no `resultTemplate`/`handoffUrl` yet — that's step 2.
2. `tool_update({ toolId, resultTemplate, handoffMode, handoffUrl, apiKey?, endpointMethod? })` — set the result template and handoff. `resultTemplate` is Mustache-style HTML using `{{path.to.value}}` to pull dotted paths out of the client endpoint's JSON response, and `{{input.<key>}}` to pull the visitor's own submitted form values (e.g. `{{input.email}}`). `{{#if}}`/`{{#each}}` are **not supported** — no loops or conditionals, they render as literal text. `handoffMode` is `"result"` (show the rendered template plus a Continue button to `handoffUrl`, hidden if `handoffUrl` is null) or `"redirect"` (skip the result screen and redirect immediately once the response/poll resolves). `handoffUrl` uses the same placeholder syntax.
3. `tool_update({ toolId, layout?, layoutConfig?, failureMessage?, usageCountStart? })` — the public page skin, also settable on `tool_create`. `layout` is `"split"` (default, copy-led, any number of `inputSchema` fields), `"video"` (a walkthrough replaces the description; drops chips/deliverables/the sample band; **cannot publish without `layoutConfig.videoUrl`**), or `"product"` (dark hero, screenshot up front; **renders exactly ONE input field** — extra `inputSchema` entries are dropped by the renderer, not rejected). `layoutConfig` is one JSON object holding every layout's content: all layouts get `eyebrow`/`submitLabel`/`formCaption`; `split` adds `chips`/`deliverables`/`deliverablesTitle`/`showDeliverables`/`formHeadline`/`sampleHeadline`/`sampleCopy`/`sampleMetrics` (max 3 `{label,value}`); `video` adds `videoUrl`/`videoAssetId`/`videoPosterUrl`/`videoPosterAssetId`/`videoDurationLabel`/`formHeadline`/`formSubcopy`; `product` adds `steps` (max 3 `{label,title,copy}`) and `logoUrls` (max 4); `sampleImageUrl`/`sampleImageKind`/`sampleImageAssetId` apply to `split` and `product`; `videoControls` (default true) applies to `video` always and to `split`/`product` when `sampleImageKind` is `"video"`. `failureMessage` (string or null) is shown to the visitor when the endpoint call fails. `usageCountStart` seeds the displayed usage counter on top of real executions.
4. `tool_preview({ toolId, sampleResponse, sampleInputs? })` — renders `resultTemplate`/`handoffUrl` against a sample response, no real endpoint call. Returns `{ html, handoffUrl, missingKeys }`. Iterate on `resultTemplate`/`handoffUrl` (step 2) until `missingKeys` is empty, and check the rendered `html` and filled `handoffUrl` look right.
5. `tool_publish({ toolId, isPublished: true })` — only after preview shows an empty `missingKeys`, and after `layoutConfig.videoUrl` is set if `layout` is `"video"`. Fires the same background hooks as landing-page publish (llms.txt, IndexNow, link graph), once.
6. `tool_runs({ toolId })` / `tool_leads({ toolId })` — after the tool is live, check real visitor executions and captured leads.

Kind `"none"` (pure lead capture, no backend call): omit `endpointUrl` (or pass `null` via `tool_update`) — same procedure otherwise, `execute` just echoes the visitor's inputs through to the template. Note the asymmetry: `tool_generate` still requires a plausible `endpointUrl` in the draft description step even for a tool the owner intends to make kind-`none` afterward — generate with a real-looking URL, then null it out via `tool_update`.

### Pricing section (`pricing.plans`)

- 1–5 plans max, no more. A request for a 6th plan is structurally impossible and fails fast (see step 2 above).
- Plan objects are **strict** — unknown keys are rejected. Required: `id`, `name`, `price` (number ≥0), `period`, `description`, `features` (array of `{ text, included }`), `cta`. Optional: `isHighlighted`, `nofollow`, `contactSales`, `isComingSoon`, `outputs[]`, `meta`, `metaItems[]`, `valueProp`, `annualPrice`, `billingNote`, `ctaUrl`, `url`, `annualUrl`, `urls{monthly,annual}`.
- Put a plan's checkout/signup link in `url` or `ctaUrl` (or `urls.monthly`/`urls.annual` for cadence-specific links) — don't invent a new key for it; it'll be rejected.
- **`cta` is the button LABEL (e.g. "Start Free Trial"), never a URL.** The link goes in `url` (or `urls: { monthly, annual }`).
- Enterprise / contact-sales tiers: `hidePrice: true` (default false) hides the price numeral, period and billing note (keep `price: 0`); `priceLabel` (e.g. "Custom") shows instead. A `contactSales` plan with `price: 0` and no `hidePrice` is also treated as hidden.

### Footer link scope

The footer's automatic columns (side pages / comparison pages / blogs) are the same on every page by default. To make each side page list only related links:

- LP default: `set_landing_page_section({ slug, section: "footer", field: "linkScope", value: "all" | "related" })`.
- Per side page: `set_side_page_section({ slug, sideKey, sectionId: "footer", fields: { linkScope?: "all" | "related", mode?: "inherit" | "replace" | "extend", sections? } })` — works on every side-page type (stored as `config.footer`; on landing-type pages only `linkScope`/`mode` go there, `sections` writes keep using `overrides.footer`). `null` removes a key; all-null resets to inherited.
- `"related"` = side pages + articles connected by keyword cluster / internal-link graph (max 6 each); falls back to the full lists when none are connected. The main LP footer is unaffected.

### Assign or reassign an asset to a slot

Images and videos on a page are **not** stored inside section text — they live in a `slotMap` keyed by dotted slot keys (e.g. `hero.backgroundVideo`, `cta.image`, `hero.logoCarousel`, `showcase.video.1`, `pricing.illustration`). Each slot holds one asset ID, or an array of IDs for "array" slots. A main LP's slots are namespaced under a `pageKey` (`main` by default); a side page keeps its **own** separate `slotMap`.

1. Make sure the asset is in the brand library first — `list_assets`, `upload_asset`, or `import_asset_from_url`. `assign_asset_to_slot` only references existing assets; it does not upload.
2. Discover slot keys and their current assignments: `list_asset_slots({ slug, sidePageSlug?, pageKey? })` — pass `sidePageSlug` for a side page. Returns each slot with its media type and current asset.
3. Assign, reassign, or clear the slot: `assign_asset_to_slot({ slug, slotKey, assetId? | assetIds?, sidePageSlug?, pageKey?, clear? })`. Synchronous — applies immediately, no polling.

- **Main landing page:** `assign_asset_to_slot({ slug, slotKey: "hero.backgroundVideo", assetId })`.
- **Side page:** `assign_asset_to_slot({ slug, sidePageSlug, slotKey: "cta.image", assetId })`.
- **Clear a slot:** `assign_asset_to_slot({ slug, slotKey: "hero.logoCarousel", clear: true })`.

Rules:
- Array slots require `assetIds` (plural); single slots require `assetId`. Using the wrong one is rejected.
- The asset must belong to the brand **and** match the slot's media type (IMAGE vs VIDEO), or the call is rejected.
- A main-LP assignment creates a new **draft** version — `publish_landing_page({ slug })` to make it live. Side-page `slotMap` writes apply in place with no version of their own — unlike `set_side_page_section`/`edit_side_page`/the block tools, which do create one (see "Version history").
- A side page carrying its own `slotMap` shadows the parent LP's slots for that page.

### Side pages

1. `list_side_pages({ slug })` — see what already exists under a parent landing page. Each entry includes `previewUrl` (browser-openable draft preview, present whenever the server has a preview domain configured) and `liveUrl` (only when a custom domain is connected) — hand these to the user instead of constructing URLs yourself.
2. `generate_side_page({ slug, key, prompt?, brief?, keywords?, sidePageType?, name? })` — `key` is the required URL-slug fragment for the side page. `sidePageType` is `"landing" | "text" | "comparison" | "custom" | "spotlight"` (defaults to `"landing"`) — there is **no `type` param**, and `"comparison"` requires a persisted `briefId`. `"spotlight"` is the **feature / service spotlight** page: a fixed-schema single-topic page (hero, a 3-6 step feature tour, 3-6 benefits, a stat + quote proof band, CTA footer) with no testimonial carousel, pricing table or showcase — use it whenever the page covers ONE feature or service, instead of `"landing"`, which pulls in the full marketing template. Spotlight is freeform-only: pass `key` + `prompt` (+ optional `name` for the display title, `keywords`); `brief` and `selectedSections` are ignored for it. Async for most cases (returns `operationId`); some comparison briefs run synchronously and return `sidePageId` directly with no `operationId`. Poll `get_job` when an `operationId` is returned; the completed result includes `previewUrl`/`liveUrl` as above.
3. `import_side_page_html({ slug, url? | html?, sidePageSlug?, voiceProfileId?, autoAssignAssets? })` — clone an existing web page as a side page under the parent LP `slug`. Pass `url` (PostKing fetches the page server-side — don't download the HTML yourself; an invalid or SSRF-blocked URL 400s) or `html` directly, not both. `sidePageSlug` is optional with `url` (derived from the URL path), **required** with `html`. Behavior follows the *parent* LP's type: raw-HTML parent → synchronous verbatim import, returns `{ sidePage: { id, slug }, mode: "import", previewUrl?, liveUrl? }`; sectioned parent → the source page's text is extracted and seeds an async side-page generation instead, returns `{ mode: "generate", operationId, key }` — poll with `get_job`, same as `generate_side_page`. 409 = a side page with that slug already exists. Contrast with `import_landing_page_html`, which creates a standalone top-level landing page, not one nested under a parent.
4. `view_side_page({ slug, sideKey })` — inspect content and rendered HTML; also returns `previewUrl`/`liveUrl`.
5. `edit_side_page({ slug, sideKey, name?, newKey?, instructions?, updateReferences? })` — `name` sets the display title, which is what shows up in the parent landing page's auto-generated footer/nav "Solutions" links and in breadcrumbs; changing it updates those automatically. `newKey` renames the side page's slug — 1 or 2 segments, lowercase `[a-z0-9-]`, separated by `/` for a nested page (e.g. `"features/feature-1"`). First-segment values the renderer's own routes shadow (`api`, `assets`, `blog`, `byslug`, `dev`, `tools`) are rejected with 400, as are second-segment values read as a draft/version preview (`draft`, or all-digits like `2024`). `~` is stripped from input — it's the internal separator PostKing substitutes for `/` in nested-slug URLs, so it's reserved. A slug already taken on this landing page returns 409. The old URL 404s afterward (no redirect), and existing internal references (blog backlinks, internal links) are rewritten in the background; poll the returned `slugRewriteOperationId` for that rewrite. `updateReferences` defaults to `true` and can be set `false` to skip that reference-rewrite cascade when renaming via `newKey`. `instructions` still only records a page-level annotation and does not itself trigger an AI edit. Use `set_side_page_section({ slug, sideKey, sectionId, fields?, field?, value?, instructions? })` for a single structured section edit instead.
6. `set_side_page_state({ slug, sideKey, published })` — publish or unpublish it.

### Custom side pages (block model)

A `sidePageType: "custom"` side page (see `generate_side_page` above) stores content as an ordered `blocks[]` array instead of the fixed section list `"landing"`/`"spotlight"` pages use — reach for it when the page needs a shape the fixed sections can't express. Every block tool addresses a specific side page with **both** the parent LP `slug` and the side page's `sideKey` (from `list_side_pages`) — there is no way to add blocks to a top-level landing page directly, only to a `"custom"`-type side page nested under one.

1. `list_block_types({ brandId?, detail? })` — the entire runtime discovery mechanism for what `add_block` accepts: Tier 1 (the 12 built-in Uland sections — `hero`, `showcase`, `videos`, `howItWorks`, `features`, `categoryExplorer`, `replacesStack`, `comparisonMatrix`, `cta`, `faq`, `pricing`, `roiCalculator`) merged with Tier 2 (brand-visible `BlockType` rows seeded in the DB). `detail: "full"` returns each type's complete `jsonSchema` (and `template` for Tier 2) — call it before every `add_block`/`edit_block` to see exactly what shape `props` must have, rather than assuming a fixed catalog; a new Tier 2 type is usable immediately with no server rebuild. Tier 3 (`type: "html"`) is always available and is not listed here.
2. `add_block({ slug, sideKey, type, props, position? })` — `type` is a key from `list_block_types` or the literal `"html"`. `props` must match that type's schema exactly (a mismatch is rejected, not coerced, and the error names the offending field); for `"html"` it's `{ html: "<section class=\"blk-section\">...</section>" }`, sanitized server-side against a `blk-*` class/tag allowlist (no `<script>`/`<style>`/`<img>`/inline standard CSS) and REJECTED with a reason on violation, never silently stripped. `position` is a 0-based insert index into the current array; omit to append. The server generates and returns the new block's id (`blk_xxxxxxxx`).
3. `edit_block({ slug, sideKey, blockId, type?, props? })` — `props` replaces the block's ENTIRE props object, not a shallow merge; fetch current props first via `view_side_page({ slug, sideKey, detail: "full" })` (under `overrides.blocks`) if you're only changing one field, merge locally, then send the full object back. Passing `type` also revalidates `props` against the new type's schema — pass matching `props` in the same call.
4. `delete_block({ slug, sideKey, blockId, confirm: true })` — removes one block; recoverable via `list_side_page_versions`/`restore_side_page_version` even after deletion, since version history is append-only.
5. `reorder_blocks({ slug, sideKey, blockIds })` — `blockIds` must be the full, exact set of the page's current block ids in the new order; it cannot add or remove blocks (use `add_block`/`delete_block` for that), and a set mismatch is rejected.

All four writes create a new draft version (draft-only until `set_side_page_state({ published: true })`) and are fenced by optimistic concurrency: a 409 means another write landed between your last read and this call — call `view_side_page` again to re-fetch current blocks and retry against the fresh state, don't blindly resend the same call.

### Version history

Every content write to a landing page (`set_landing_page_section`, `apply_vibe_edit`, `regenerate_landing_page`, a main-LP `assign_asset_to_slot`) or to a side page (`set_side_page_section`, `edit_side_page`, the block tools above) creates a new **draft** version — publishing is always a separate step (`publish_landing_page` / `set_side_page_state({ published: true })`). The one exception is `assign_asset_to_slot` targeting a side page's `slotMap`, which writes in place with no version of its own.

- Landing page: `list_lp_versions({ slug })` — history, each entry with a `previewUrl`. `view_lp_version({ slug, versionId })` — inspect one version's section content. `view_lp_draft({ slug })` — the current unpublished draft. `restore_lp_version({ slug, versionId })` — roll the draft back to a prior version (does not touch the published version). `delete_lp_version({ slug, versionId, confirm: true })` — permanently remove a historical version; the published version and the last remaining version can't be deleted.
- Side page: `list_side_page_versions({ slug, sideKey })` — history; the response's top-level `currentVersionId` (draft) and `publishedVersionId` (live) tell you draft vs. live without a second call. `view_side_page_version({ slug, sideKey, versionId })` — inspect a snapshot (`{ type, overrides, siteMetadata, slotMap, config }`). `restore_side_page_version({ slug, sideKey, versionId })` — moves the draft forward to a new version copied from the target (forward history is never deleted); the live page is unaffected until you `set_side_page_state({ published: true })` again. `delete_side_page_version({ slug, sideKey, versionId, confirm: true })` — deleting the current draft version is allowed and repoints the draft to the newest remaining version; the published version and the last remaining version can't be deleted.

### Landing page translations

A landing page can have language variants linked into a shared translation group — one page row per language, distinct from the brand's default `contentLanguage`. The target language must already be enabled for the brand (`get_brand_languages`/`add_brand_language`, in the `postking` skill) or `create_landing_page_translation` is rejected with a language-not-enabled error.

1. `get_landing_page_translations({ slug })` — read the translation group `slug` belongs to: `{ slug, languageCode, pathPrefix, translationGroupId, isTranslationSource, variants: [...] }`. `variants` lists the OTHER language variants already in the group (empty if none yet). Call this before `create_landing_page_translation` to check whether a variant in the target language already exists.
2. `create_landing_page_translation({ slug, language })` — `slug` must be the group's actual source page: fails with 400 if `slug` is itself already a translation (chain new variants off the source, never off another variant), and 409 if a variant in that language already exists in the group. Creates a new DRAFT page row (correct slug suffix and `/{languageCode}` path prefix) and kicks off an async job that pulls and translates the source page's current content; the response's `operationId` is pollable with `get_job`, and `pullFromSourceUrl` is a documented manual fallback (an AI-edit pull-from-source call) if you need to re-trigger the pull yourself.

### Custom domain

1. `add_domain({ domain, primaryContentType: "landing_page", brandId? })` — registers the domain and returns a DNS TXT record to add at the registrar. Defaults to `hostingMode: "postking"` (our infra serves it, verification checks an A record + the TXT record). Pass `hostingMode: "self_hosted"` instead when the domain will be served by the customer's own infra — see "Self-hosting".
2. `verify_domain({ domainId })` — checks DNS once the record is added; `domainId` comes from the `add_domain` response or `list_domains`. For a `postking`-hostingMode domain this checks both the A record and the TXT record; for `self_hosted` it's TXT-ownership only (no A record to check).
3. `connect_landing_page_domain({ slug, domainId })` — attaches an already-verified domain to a specific landing page (the page must be published first; requires paid access). **Mode-match rule**: the domain's `hostingMode` must agree with the page's `distribution` (`postking` ↔ `hosted`, `self_hosted` ↔ `self_hosted`) — a mismatch 409s with a message naming which side to change; call `set_landing_page_distribution` or re-add the domain with the matching `hostingMode`, then retry.
4. `delete_domain({ domainId })` — removes a custom domain; connected blogs and landing pages are unlinked but not deleted.

### Self-hosting

`distribution` can also be `"external"` (the customer's own site serves it; PostKing only emits URLs from `siteBaseUrl` + `sidePagePattern`; no domain may be connected — switching with one connected 409s). `set_landing_page_distribution({ slug, distribution?, siteBaseUrl?, sidePagePattern?, trailingSlash? })`: `sidePagePattern` must contain `{slug}` (`{lang}` optional), e.g. `"/p/{slug}/"`, default `"/{slug}"`; `trailingSlash` (default off) rewrites the stored pattern. For external/self-hosted sync, use `list_content_changes` → `export_side_page({ landingPageSlug, sideKey, ifContentHash? })` / `export_landing_page({ slug })` — see the `postking-blog` skill's "External hosting + headless sync" (emit only `schemaGraph`-style JSON-LD from the export, full re-sync after platform URL/schema changes).

By default a landing page's `distribution` is `"hosted"` — PostKing/universal-lp serves the page. Set it to `"self_hosted"` to have the customer's OWN infrastructure serve the page instead, calling PostKing's public API directly:

1. `set_landing_page_distribution({ slug, distribution: "self_hosted" })`. If a domain is already connected whose `hostingMode` disagrees with the new value, this 409s — disconnect (or `delete_domain`) that domain first.
2. **Localhost works immediately, no domain needed** — once the page is published, a local dev server can fetch draft/preview content straight from the public API (see the three endpoints below).
3. For a production host: `add_domain({ domain, hostingMode: "self_hosted" })` → `verify_domain({ domainId })` (TXT ownership record only — there's no A record, since the domain deliberately does NOT point at PostKing) → `connect_landing_page_domain({ slug, domainId })`.
4. `publish_landing_page({ slug })` — for a self-hosted page this makes the content available at the public API; the customer's own renderer, not PostKing/universal-lp, serves the actual public URL.

**Building a custom renderer** — three public read endpoints, no auth required beyond the query params shown:
- `GET /api/landing-pages/{slug}?host=<your host>[&sidePage=]`
- `GET /api/blog-posts?slug={slug}`
- `GET /api/blog-posts/{articleSlug}?slug={slug}`

**Example starter (optional, not required)** — [postking-selfhost](https://github.com/bitsandtea/postking-selfhost) is a minimal, MIT-licensed reference implementation against those same endpoints. Any custom build that calls them directly works identically; the example just saves writing the fetch/render glue from scratch. Setup: `POSTKING_API_URL=https://try.postking.app`, `POSTKING_BRAND_SLUG=<lp slug>`, `SITE_URL=<your local or production URL>`.

## CLI fast path

| Goal | Command |
|---|---|
| List / generate | `pking lp list` / `pking lp generate --topic "..." [--slug <slug>] [--voice <id>]` |
| Import a local static-build folder | `pking lp import-bundle <folder> [--name "..."] [--entry <path>] [--convert-slug <slug>] [--side-pages]` — refuses secrets/VCS (`.env`, `.git/`, `id_rsa`) and executable/script/archive files outright; the primary path for a folder on disk (MCP has no folder-upload tool, only `import_landing_page_bundle` for already-hosted files) |
| View / preview | `pking lp view <slug>` |
| Edit (metadata or AI pass) | `pking lp edit <slug> [--title "..."] [--instructions "..."]` |
| Vibe edit | `pking lp vibe <slug> --instructions "..." [--scope full\|section --section-id <id>] --wait` then `pking lp vibe status <slug> <operationId>` |
| Side pages | `pking lp side list <slug>` / `pking lp side edit <slug> <sideKey> --instructions "..."` — generation is currently MCP-only, see Pitfalls |
| Side page section | `pking lp side section <slug> <sideKey> --id <sectionId> [--content "..." | --file <path>] [--instructions "..."]` |
| Publish / unpublish a side page | `pking lp side state <slug> <sideKey> --publish` (or `--unpublish`) |
| Custom domain: register + verify | `pking domains add <domain> [--hosting-mode self_hosted]` → `pking domains verify <domain>` |
| Custom domain: connect to a landing page | `pking lp connect-domain <slug> --domain-id <id>` |
| Distribution: hosted vs. self-hosted | `pking lp set-distribution <slug> hosted\|self_hosted` |
| Publish the landing page | `pking lp publish <slug>` |
| Tool pages: list / get / create / generate | `pking tools list` / `pking tools get <toolId>` / `pking tools create --file <path>` / `pking tools generate --description "..." --endpoint-url <url> --inputs-outputs "..."` |
| Tool pages: update / preview | `pking tools update <toolId> --file <path>` / `pking tools preview <toolId> --file <path>` (repeat until `missingKeys` is empty) |
| Tool pages: publish / delete | `pking tools publish <toolId>` (add `--unpublish` to take down) / `pking tools delete <toolId>` |
| Tool pages: runs / leads | `pking tools runs <toolId>` / `pking tools leads <toolId>` |

`--brand`/`--json` work on every `pking tools` subcommand, same as the rest of this table.

For the full command catalog, use the `postking` skill's `references/commands.md`, or run `pking lp --help`.

## Pitfalls

- **`import_landing_page_bundle` cannot take a local file path or a `file://` URL as `url`** — it fetches over the network server-side, so every `files[].url` must be publicly reachable http(s). A folder on the user's own machine goes through the `postking-cli` `import-bundle` command instead, never this tool with made-up/local paths.
- **`generate_side_page` has no `type` param** — it's `sidePageType`, and the only valid values are `"landing"`, `"text"`, `"comparison"`, `"custom"`, `"spotlight"`. Don't invent other values (e.g. "pricing", "features", "legal") as `sidePageType` — those aren't recognized side-page types by this tool. A feature page is `"spotlight"`, not a made-up `"features"` value.
- **`key` is required on `generate_side_page`** — it's the side page's URL-slug fragment, not optional metadata.
- **`pking lp side generate <slug> --type <type>` is currently broken** — the inner route ignores the `<slug>` path param and requires `name`/`slug`/`landingPageId` in the request body, so it always 400s. Use the MCP tool `generate_side_page({ slug, key, sidePageType })` instead.
- **`connect_domain_to_publication` is blog-only.** It takes a `publicationId`, not a landing-page slug — don't use it to attach a domain to a landing page; use `connect_landing_page_domain({ slug, domainId })` instead.
- **Pricing `cta` is a label, not a URL** — put the link in `url`/`urls`; use `hidePrice` + `priceLabel` for "Custom" tiers.
- **A domain's `hostingMode` must match the page's `distribution` to connect.** `postking` hostingMode only pairs with `hosted` distribution; `self_hosted` only with `self_hosted`. A mismatch 409s on `connect_landing_page_domain` — see "Self-hosting".
- **Async operations can take 30s–5min.** Poll `get_job` (or the CLI's built-in `--wait`) rather than assuming failure early.
- **Comparison-type side pages may return synchronously** with `sidePageId` and no `operationId` — don't poll `get_job` with an ID you don't have; check the immediate response first.
- **Side-page content writes DO have version history and a draft/publish split**, the same shape as the main LP: `set_side_page_section`, `edit_side_page`, and the block tools (`add_block`/`edit_block`/`delete_block`/`reorder_blocks`) each create a new draft version, and `set_side_page_state({ published: true })` is the side-page equivalent of `publish_landing_page` — see "Version history". The one exception is `assign_asset_to_slot` targeting a side page's `slotMap`, which still writes in place with no version of its own — double-check slot assignments before writing.
- **Renaming a side page's title or key is MCP-only** (`edit_side_page({ name, newKey })`) — there is no `pking` CLI flag for it yet; `pking lp side edit` only supports `--instructions`.
- **A missing `previewUrl` or `liveUrl` is not an error.** No `liveUrl` just means no custom domain is connected yet; no `previewUrl` means the server has no preview domain configured. Don't retry or report failure — just skip handing that link over.
- **`newKey` changes the live URL with no redirect** — the old URL 404s immediately. Warn the user before renaming a published side page, and mention that internal references are fixed up asynchronously (poll `slugRewriteOperationId`), not instantly.
- **`apiKey` on a Tool is write-only** — `tool_get`/`tool_list` never return it, only `apiKeyLast4` (the last 4 chars, or null). Set or rotate it via `tool_update`; there is no way to read it back.
- **Kind `"none"` isn't a separate tool type** — it's the same `Tool` model with `endpointUrl` omitted/null. Don't invent a distinct "lead capture" tool kind; get there by nulling `endpointUrl` via `tool_update`.
- **Placeholder escaping differs by destination.** `{{path}}`/`{{input.key}}` values are HTML-escaped inside `resultTemplate`, but URL-encoded when filled into `handoffUrl` — a value with `&` or spaces looks different in each. A missing value always renders as empty string, never the literal `{{...}}` text. `{{#if}}`/`{{#each}}` are not supported — loops/conditionals in a template render as literal text, not evaluated or rejected.
- **Layout capability limits are enforced by the renderer, not the write API.** Sending more than one `inputSchema` entry to a `"product"`-layout tool isn't rejected — the renderer just drops all but the first at render time. Sending a `"video"`-layout tool without `layoutConfig.videoUrl` isn't rejected either — it blocks `tool_publish` instead. Match `inputSchema`/`layoutConfig` to the chosen `layout` up front rather than relying on either failing loudly.
- **`emailGate` and `isFree` no longer exist on `Tool`.** Both were removed upstream — don't send either field to `tool_create`/`tool_update`, and don't expect them back from `tool_get`/`tool_list`.
- **A Tool needs a verified domain or `websiteUrl` to work for real visitors** — CORS on the public execute endpoint allow-lists only the brand's verified `Domain` rows plus `websiteUrl`. Check `get_brand_info`/domain tools before telling a user their tool is ready if the brand has neither.
- **Always `tool_preview` with a realistic `sampleResponse` before `tool_publish`.** An empty `missingKeys` array on preview is the only signal the template will render correctly for a real visitor; publishing with unchecked `missingKeys` ships broken-looking result pages silently — missing values render blank, not as an error.

## Verification

- `get_credits({ detail: "short" })` — confirms auth and balance.
- `list_landing_pages({})` — returns the brand's landing pages (or an empty list if none created yet, which is expected pre-setup).
