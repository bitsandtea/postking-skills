---
name: postking-seo
description: Run SEO / GEO from seed keywords to published, AI-citation-friendly articles using PostKing — seed, expand, cluster, roadmap, brief, write, and publish.
license: MIT
compatibility: "Works with any MCP-compatible client connected to postking-mcp (local stdio or hosted at https://mcp.postking.app/mcp); the pking CLI is an optional fast path when a shell is available."
metadata:
  icon: https://raw.githubusercontent.com/bitsandtea/postking-skills/main/assets/icons/postking-seo.svg
  category: marketing
---

# SEO / GEO

An agentic pipeline that takes a brand from 3–10 seed keywords all the way to published, SEO / GEO-targeted blog articles using PostKing.

## What is GEO?

GEO stands for **Generative Engine Optimization** — writing and structuring content so that AI assistants (Claude, ChatGPT, Perplexity, Gemini, Google AI Overviews) quote and cite it when answering user questions.

Classic SEO optimizes for click-through from a search-results page; success is measured as rank position and CTR. GEO optimizes for being included as a cited source *inside* an AI-generated answer; success is measured as citation rate in AI responses. The end user may never click — being the quoted authority is the win. Both still matter: SEO drives discovery surfaces that index your content, and GEO determines whether that content survives extraction into an AI answer.

GEO-friendly content follows formatting rules that LLM extractors prefer:

- **Definition-first paragraphs** — the first 1–2 sentences under each H2 answer the heading as a standalone, quotable claim.
- **Headings as questions or definitive statements**, not clever wordplay, so hierarchy maps cleanly to extractable Q&A.
- **FAQ blocks** with explicit Q→A structure (`<dt>/<dd>` or `<h3>` + paragraph). Highest-yield GEO format.
- **Source-attributable claims** — concrete numbers, named studies, dated stats, primary sources cited inline.
- **Schema markup** (Article, FAQPage, HowTo) as JSON-LD so crawlers parse intent unambiguously.
- **Self-contained sections** — each H2 makes sense without scrolling up; LLMs extract sections out of context.
- **Comparison tables and lists** — easy for LLMs to lift verbatim.
- **Authority signals in-prose** — founding dates, sample sizes, first-person expert framing.

PostKing's SEO / GEO pipeline produces content optimized for both — keyword-target SEO with GEO-friendly structure (definition-first paragraphs, FAQ blocks, source-attributable claims) baked in by default.

## When to Use

Use this skill when a user wants to grow organic/AI-citation traffic from keywords: seeding a topic list, clustering keywords into topical hubs, generating a content roadmap, drafting briefs, writing SEO/GEO-structured articles, or publishing/scheduling them. Also use it for post-publish audit work (gap analysis, competitor content diff) and for generating cluster-linked side pages or standalone comparison pages.

## When NOT to Use

Not for one-off social posts, blogs without a keyword/cluster angle, or general brand content — route those to the `postking-social`/`postking` skill instead. Not for discovering or registering competitor domains — that's the `postking-competitor-intel` skill (though its output feeds this one). Not for full multi-channel campaign planning — that's `postking-storylines`.

## Minimal tool subset

This skill needs 15 tools out of PostKing's 200+:

- `seo_add_seeds` — register seed keywords.
- `seo_generate_keywords` — expand seeds into a full keyword set (async).
- `seo_categorize` — tag keywords by search intent (per-keyword `updates` array, built from `seo_list_keywords` IDs).
- `seo_generate_clusters` — group keywords into topic-pillar clusters (async).
- `seo_list_clusters` — list clusters.
- `seo_bulk_approve_clusters` — approve clusters to unlock brief generation.
- `seo_generate_roadmap` — build prioritized article topics from an approved cluster.
- `seo_list_briefs` — list generated briefs.
- `seo_get_brief` — inspect one brief.
- `seo_edit_brief` — structured edit of a brief (full `briefData` replacement and/or status flip).
- `seo_approve_briefs` — approve briefs, unlocking article writing.
- `seo_write_article` — draft an article from an approved brief.
- `seo_list_results` — list generated articles.
- `seo_publish_article` — publish or schedule.
- `get_job` — poll any async operation.

**Granular & optional operations** — use these to drive a single step in isolation (e.g. you already have a keyword list and want to skip to clustering) or for post-publish/competitive work, rather than running the full pipeline:

- `seo_list_keywords` — list the brand's scored keyword set (keyword, volume, KD, intent).
- `seo_gap` — gap analysis: high-value topics not yet covered.
- `seo_competitor` — competitor content diff against a domain.
- `seo_auto_assign_cta` — batch-assign a published side-page CTA to blogs.
- `seo_generate_side_page` — generate a side page linked to an SEO cluster for topical authority.
- `create_comparison_page` — generate a standalone comparison/alternatives page.

## Prerequisites

1. Authenticated with credits — see the `postking-getting-started` skill.
2. A brand already onboarded — see the `postking-getting-started` skill.
3. Optional: competitors registered for competitor-aware clustering — see the `postking-competitor-intel` skill.

## Procedure

> "Run the SEO / GEO flow for my brand. Seeds: 'AI content generation', 'social scheduling', 'brand voice'."

Async tools return `{ operationId, status }` immediately — poll `get_job({ operationId })` until `state` is `completed`, `failed`, `partially_failed`, or `cancelled`.

1. **Seed.** `seo_add_seeds({ brandId, seeds: ["AI content generation", "social scheduling", "brand voice"] })`.
2. **Expand.** `seo_generate_keywords({ brandId })` → async, poll `get_job`. Expands seeds into ~100 scored keywords.
3. **Categorize.** There's no brand-wide categorize call — build a per-keyword `updates` array first. Call `seo_list_keywords({ brandId })` to get keyword IDs, then `seo_categorize({ brandId, updates: [{ keywordId, intent, userTags? }, ...] })` — `updates` is required (min 1, max 500 entries), each entry is `{keywordId, intent?, userTags?}` with `intent` ∈ informational | commercial | navigational | transactional. Informational-intent keywords are prime GEO targets — they're the questions users ask AI assistants directly.
4. **Cluster.** `seo_generate_clusters({ brandId })` → async, poll `get_job`. Groups keywords into topic-pillar clusters. Cluster pillars become topic-authority hubs that AI engines cite back to. If competitors are registered on the brand, this step automatically classifies each cluster's `productFit` and flags competitor-dominated clusters.

   **Competitor-aware clustering.** If the brand has competitors registered first (via the `postking-competitor-intel` skill), `seo_generate_clusters` automatically injects a "Registered competitors" block into the cluster-labeling step — no separate tool call, it's automatic once competitors exist. For each cluster the labeler then returns:
   - `clusterMeta.productFit`: `"core"`, `"adjacent"`, or `"out_of_scope"`.
   - `clusterMeta.archetype: "competitor"` (only when the cluster's keywords are dominated by a known rival brand), plus `detectedCompetitor` and a pre-built "conquest" brief plan (review article + head-to-head comparison + alternatives listicle, optionally a pricing blog) instead of the generic hub-and-spoke plan.
   - Inspect these via `seo_list_clusters({ brandId, detail: "medium" })` or `"full"` — `clusterMeta.productFit`/`archetype` only surface at `medium`/`full` detail, not `short`.
   - When you `seo_bulk_approve_clusters` a `archetype: "competitor"` cluster, it produces the conquest-plan briefs rather than generic ones.
   - Register competitors first with the `postking-competitor-intel` skill.
5. **Review clusters.** `seo_list_clusters({ brandId, detail: "medium" })` — surface clusters with pillar keyword, keyword count, and top keywords (use `"full"` for raw data; `clusterMeta.productFit`/`archetype` only surface at `medium`/`full`). Ask the user which to target, or pick by size/volume.
6. **Approve.** `seo_bulk_approve_clusters({ clusterIds: [...] })` (or `seo_approve_cluster` for a single one; reverse with `seo_reject_cluster`/`seo_bulk_reject_clusters`). Approval is the gate — only approved clusters get briefs drafted, and this fires an async brief-generation job per cluster.
7. **Roadmap.** `seo_generate_roadmap({ clusterId })` — produces prioritized article topics from the approved cluster. Each roadmap item gets a brief auto-drafted in the background, specifying FAQ blocks and definition-first headings to drive the GEO format.
8. **Review & edit briefs.** `seo_list_briefs({ brandId, status: "pending_review" })` → `seo_get_brief({ briefId })` to inspect. Refine with `seo_edit_brief({ briefId, briefData })` — pull the current `briefData` from `seo_get_brief`, mutate the outline (H2s, FAQs, keyword targets, etc.) locally, and send the full replacement back; there is no free-text `instructions` param — or rebuild from scratch with `seo_regenerate_brief({ briefId })` (async). Brief approval is where you lock in the FAQ blocks and definition-first H2 structure — the article writer treats the approved `briefData` as a contract.
9. **Approve briefs.** `seo_approve_briefs({ briefIds: [...] })` — unlocks article writing.
10. **Write.** `seo_write_article({ roadmapItemId })` — drafts an article from a roadmap item whose brief is approved. Repeat per item. Articles ship with FAQ schema and definition-first H2 paragraphs by default.
11. **Review results.** `seo_list_results({ brandId })` — list generated articles and their status.
12. **Publish.** `seo_publish_article({ articleId, publicationId, schedule })` — publish, or schedule for later. This is the free-tier choke point.
13. **Audit (optional).** `seo_gap({ brandId })` — uncovered high-value topics. `seo_competitor({ brandId, domain })` — competitor content diff.
14. **Auto-assign CTAs (optional, post-publish).** `seo_auto_assign_cta({ blogIds: "all" })` (or a 1–50 array) — batch-assigns a published side-page CTA to each blog. Skips blogs that already have a CTA and Webflow-synced blogs by default; pass overwrite/include-webflow flags to change that. Hard cap of 50 blogs per call.
15. **Cluster-linked side pages / comparison pages (optional).** `seo_generate_side_page({ slug, key, clusterId })` — `slug` is the parent landing page's slug; generates a side page under it, linked to an SEO cluster, strengthening the cluster's topical authority: it expands covered subtopics, feeding the citation patterns AI engines use when answering questions in that domain. `create_comparison_page({ ... })` generates a standalone comparison/alternatives page. Auto-assigned CTAs increase conversion yield without changing the article's GEO-friendly structure.

## CLI fast path

If a shell is available, the same flow via `pking`:

```
pking seo seeds "AI content generation" "social scheduling" "brand voice"
pking seo generate                                  # expand seeds (long-running)
pking seo categorize
pking seo cluster
pking seo clusters list
pking seo clusters approve <clusterId>
pking seo clusters bulk-approve <clusterId1> <clusterId2>
pking seo briefs generate --cluster <clusterId> [--cluster <clusterId2> ...]   # generates the roadmap + briefs (L3) for approved cluster(s)
pking seo roadmap [--cluster <clusterId>]                                      # READ-ONLY: lists roadmap items, optionally filtered by cluster
pking seo briefs list [--status pending_review]
pking seo briefs view <briefId>
pking seo briefs regenerate <briefId>
pking seo briefs approve <briefId>
pking seo briefs bulk-approve <briefId1> <briefId2>
pking seo write <roadmapItemId> [--hero-image] [--voice <id>] [--no-wait] [--json]
pking seo gap
pking seo competitor --domain <competitorDomain>
pking seo publish --article-id <articleId> [--publication <id>] [--schedule <iso>]
pking seo stats
```

CTA auto-assignment is MCP/dashboard-only — there is no `pking blogs auto-assign-cta` command. Use the `seo_auto_assign_cta({ blogIds })` MCP tool instead (`blogIds` is an array of up to 50 blog IDs, or the literal `"all"`).

Brief edits (`seo_edit_brief`'s structured `briefData` replacement) have no CLI command — do them via the MCP tool or the PostKing dashboard's brief editor. `pking seo roadmap edit <id>` supports only `--action <ignore|restore|start|complete>` and `--status <suggested|ignored|in_progress|completed>` (no `--title`/`--priority`). `pking seo clusters approve <id>` and `pking seo briefs approve <briefId>` each take a single id — use `bulk-approve`/`bulk-reject` (clusters) for multiple ids at once.

**CLI-only granular operations** (no direct MCP tool equivalent — useful when composing your own pipeline):

- **List scored keywords:** `pking seo keywords list --json` — the full scored keyword list (keyword, volume, KD, intent). Pipe a filtered subset into CSV clustering below.
- **Cluster from a CSV:** `pking seo clusters generate --csv keywords.csv --brand <brandId>` (or `--brand <brandId>` alone to use the brand's existing keywords). Async — the CLI polls automatically and prints `{clustersCreated, clustersMerged, keywordsAssigned}`; pass `--no-wait` for an `operationId` and poll manually with `pking jobs list`. CSV needs a header row with a required `keyword` column; optional `volume`, `kd`, `intent` columns improve cluster quality:
  ```
  keyword,volume,kd,intent
  ai content generation,8100,42,informational
  seo content tool,2400,38,commercial
  ```
  Other flags: `--target-size <n>`, `--no-cannibal-check` (skip dedup check), `--json` (full cluster objects).
- **Generate a single brief from agent-supplied data:** `pking seo briefs generate-one --pillar "ai content generation" --supporting "ai blog writer,seo content tool" --type blog --research advanced --brand <brandId> --json` — produces one content brief without a roadmap, from data you supply directly. Async, polls and prints the full `briefData` JSON. Required: `--pillar`, `--supporting` (comma-list, no spaces), `--type blog|comparison|tool|landing`, `--brand`. Optional: `--intent`, `--research advanced|fast` (default `advanced`; `fast` skips external research), `--tier hub|spoke|side_page`, `--json`, `--no-wait`. The returned `briefData` is the same structure accepted by `seo_edit_brief`'s `briefData` param.

## Expected Output

- A populated content roadmap tied to keyword clusters.
- Draft articles scored against primary keywords, each with FAQ/definition-first GEO structure.
- A gap-analysis report listing uncovered high-value topics (if run).
- At least one published or scheduled article (if credits + free tier allow).

## Pitfalls

- `INSUFFICIENT_CREDITS` — surface the `checkoutUrl` from the error envelope and stop, or hand off to the `postking-getting-started` skill's billing section (`billing_list_packs` → `billing_topup`).
- `FREE_CAP_REACHED` on `seo_publish_article` — same treatment as `INSUFFICIENT_CREDITS`.
- `RATE_LIMITED` — back off using the `retryAfter` value from the error envelope before retrying.
- `UNAUTHORIZED` — re-run the login flow described in the `postking-getting-started` skill.
- `clusterMeta` missing/empty on a cluster — either no competitors are registered on the brand yet (see the `postking-competitor-intel` skill), or the cluster simply doesn't overlap a known rival's keyword footprint (expected for purely core/adjacent clusters).
- Calling `seo_edit_brief` with a free-text `instructions` param — it's silently ignored; the inner route only accepts `briefData` (structured) and/or `status`.
- `seo_generate_side_page` rejects a `parentSlug` param name — the tool expects `slug` for the parent landing page.
- `pking lp side generate <slug> --type <type>` is currently broken — the inner route ignores the `<slug>` path param and requires `name`/`slug`/`landingPageId` in the request body, so it always 400s. Use the MCP tool `generate_side_page({ slug, key, sidePageType })` instead (`sidePageType` ∈ `landing` | `text` | `comparison`).

## Verification

- `seo_list_clusters({ brandId, detail: "short" })` should return without error and reflect any clusters just created.
- `seo_list_briefs({ brandId })` should list briefs whose count matches the roadmap items just generated.
- `seo_list_results({ brandId })` should show the article after `seo_write_article` completes.
