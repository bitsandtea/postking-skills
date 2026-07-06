---
name: seo
description: Run SEO / GEO from seed keywords to published, AI-citation-friendly articles using PostKing — seed, expand, cluster, roadmap, brief, write, and publish.
version: 0.5.0
compatibility: "Works with any MCP-compatible client connected to postking-mcp (local stdio or hosted at https://mcp.postking.app); the pking CLI is an optional fast path if a shell is available."
metadata:
  icon: https://postking.app/icons/seo.svg
  free: true
  categories:
    - marketing
    - writing
    - data
  hermes:
    tags:
      - seo
      - geo
      - content
      - keywords
      - blogging

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

## Minimal tool subset

This skill needs 15 tools out of PostKing's 200+:

- `seo_add_seeds` — register seed keywords.
- `seo_generate_keywords` — expand seeds into a full keyword set (async).
- `seo_categorize` — tag keywords by search intent.
- `seo_generate_clusters` — group keywords into topic-pillar clusters (async).
- `seo_list_clusters` — list clusters.
- `seo_bulk_approve_clusters` — approve clusters to unlock brief generation.
- `seo_generate_roadmap` — build prioritized article topics from an approved cluster.
- `seo_list_briefs` — list generated briefs.
- `seo_get_brief` — inspect one brief.
- `seo_edit_brief` — natural-language or structured edit of a brief.
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

1. Authenticated with credits — see [`getting-started`](../getting-started/).
2. A brand already onboarded — see [`getting-started`](../getting-started/).
3. Optional: competitors registered for competitor-aware clustering — see [`competitor-intel`](../competitor-intel/).

## How to Use

> "Run the SEO / GEO flow for my brand. Seeds: 'AI content generation', 'social scheduling', 'brand voice'."

Async tools return `{ operationId, status }` immediately — poll `get_job({ operationId })` until `state` is `completed`, `failed`, `partially_failed`, or `cancelled`.

1. **Seed.** `seo_add_seeds({ brandId, seeds: ["AI content generation", "social scheduling", "brand voice"] })`.
2. **Expand.** `seo_generate_keywords({ brandId })` → async, poll `get_job`. Expands seeds into ~100 scored keywords.
3. **Categorize.** `seo_categorize({ brandId })` — tags each keyword by intent (informational / commercial / navigational / transactional). Informational-intent keywords are prime GEO targets — they're the questions users ask AI assistants directly.
4. **Cluster.** `seo_generate_clusters({ brandId })` → async, poll `get_job`. Groups keywords into topic-pillar clusters. Cluster pillars become topic-authority hubs that AI engines cite back to. If competitors are registered on the brand, this step automatically classifies each cluster's `productFit` and flags competitor-dominated clusters.

   **Competitor-aware clustering.** If the brand has competitors registered first (via the [`competitor-intel`](../competitor-intel/) skill), `seo_generate_clusters` automatically injects a "Registered competitors" block into the cluster-labeling step — no separate tool call, it's automatic once competitors exist. For each cluster the labeler then returns:
   - `clusterMeta.productFit`: `"core"`, `"adjacent"`, or `"out_of_scope"`.
   - `clusterMeta.archetype: "competitor"` (only when the cluster's keywords are dominated by a known rival brand), plus `detectedCompetitor` and a pre-built "conquest" brief plan (review article + head-to-head comparison + alternatives listicle, optionally a pricing blog) instead of the generic hub-and-spoke plan.
   - Inspect these via `seo_list_clusters({ brandId, detail: "medium" })` or `"full"` — `clusterMeta.productFit`/`archetype` only surface at `medium`/`full` detail, not `short`.
   - When you `seo_bulk_approve_clusters` a `archetype: "competitor"` cluster, it produces the conquest-plan briefs rather than generic ones.
   - Register competitors first with the [`competitor-intel`](../competitor-intel/) skill.
5. **Review clusters.** `seo_list_clusters({ brandId, detail: "medium" })` — surface clusters with pillar keyword, keyword count, and top keywords (use `"full"` for raw data; `clusterMeta.productFit`/`archetype` only surface at `medium`/`full`). Ask the user which to target, or pick by size/volume.
6. **Approve.** `seo_bulk_approve_clusters({ clusterIds: [...] })` (or `seo_approve_cluster` for a single one; reverse with `seo_reject_cluster`/`seo_bulk_reject_clusters`). Approval is the gate — only approved clusters get briefs drafted, and this fires an async brief-generation job per cluster.
7. **Roadmap.** `seo_generate_roadmap({ clusterId })` — produces prioritized article topics from the approved cluster. Each roadmap item gets a brief auto-drafted in the background, specifying FAQ blocks and definition-first headings to drive the GEO format.
8. **Review & edit briefs.** `seo_list_briefs({ brandId, status: "pending_review" })` → `seo_get_brief({ briefId })` to inspect. Refine with `seo_edit_brief({ briefId, instructions })` (or a structured `briefData` replacement), or rebuild from scratch with `seo_regenerate_brief({ briefId })` (async). Brief approval is where you lock in the FAQ blocks and definition-first H2 structure — the article writer treats the approved `briefData` as a contract.
9. **Approve briefs.** `seo_approve_briefs({ briefIds: [...] })` — unlocks article writing.
10. **Write.** `seo_write_article({ roadmapItemId })` — drafts an article from a roadmap item whose brief is approved. Repeat per item. Articles ship with FAQ schema and definition-first H2 paragraphs by default.
11. **Review results.** `seo_list_results({ brandId })` — list generated articles and their status.
12. **Publish.** `seo_publish_article({ articleId, publicationId, schedule })` — publish, or schedule for later. This is the free-tier choke point.
13. **Audit (optional).** `seo_gap({ brandId })` — uncovered high-value topics. `seo_competitor({ brandId, domain })` — competitor content diff.
14. **Auto-assign CTAs (optional, post-publish).** `seo_auto_assign_cta({ blogIds: "all" })` (or a 1–50 array) — batch-assigns a published side-page CTA to each blog. Skips blogs that already have a CTA and Webflow-synced blogs by default; pass overwrite/include-webflow flags to change that. Hard cap of 50 blogs per call.
15. **Cluster-linked side pages / comparison pages (optional).** `seo_generate_side_page({ parentSlug, key, clusterId })` — generates a side page linked to an SEO cluster, strengthening the cluster's topical authority: it expands covered subtopics, feeding the citation patterns AI engines use when answering questions in that domain. `create_comparison_page({ ... })` generates a standalone comparison/alternatives page. Auto-assigned CTAs increase conversion yield without changing the article's GEO-friendly structure.

## CLI fast path

If a shell is available, the same flow via `pking`:

```
pking seo seeds "AI content generation" "social scheduling" "brand voice"
pking seo generate                                  # expand seeds (long-running)
pking seo categorize
pking seo cluster
pking seo clusters list
pking seo clusters approve <clusterId> [<clusterId>...]
pking seo roadmap --cluster <clusterId>
pking seo briefs list [--status pending_review]
pking seo briefs view <briefId>
pking seo briefs edit <briefId> --brief-data <json>
pking seo briefs regenerate <briefId>
pking seo briefs approve <briefId1> <briefId2>
pking seo write --roadmap-id <roadmapItemId>
pking seo gap
pking seo competitor --domain <competitorDomain>
pking seo publish --article-id <articleId> [--publication <id>] [--schedule <iso>]
pking seo stats
pking blogs auto-assign-cta --all
pking lp side generate <parentSlug> --key <key> --cluster <clusterId>
```

**CLI-only granular operations** (no direct MCP tool equivalent — useful when composing your own pipeline):

- **List scored keywords:** `pking seo keywords list --json` — the full scored keyword list (keyword, volume, KD, intent). Pipe a filtered subset into CSV clustering below.
- **Cluster from a CSV:** `pking seo clusters generate --csv keywords.csv --brand <brandId>` (or `--brand <brandId>` alone to use the brand's existing keywords). Async — the CLI polls automatically and prints `{clustersCreated, clustersMerged, keywordsAssigned}`; pass `--no-wait` for an `operationId` and poll manually with `pking jobs list`. CSV needs a header row with a required `keyword` column; optional `volume`, `kd`, `intent` columns improve cluster quality:
  ```
  keyword,volume,kd,intent
  ai content generation,8100,42,informational
  seo content tool,2400,38,commercial
  ```
  Other flags: `--target-size <n>`, `--no-cannibal-check` (skip dedup check), `--json` (full cluster objects).
- **Generate a single brief from agent-supplied data:** `pking seo briefs generate-one --pillar "ai content generation" --supporting "ai blog writer,seo content tool" --type blog --research advanced --brand <brandId> --json` — produces one content brief without a roadmap, from data you supply directly. Async, polls and prints the full `briefData` JSON. Required: `--pillar`, `--supporting` (comma-list, no spaces), `--type blog|comparison|tool|landing`, `--brand`. Optional: `--intent`, `--research advanced|fast` (default `advanced`; `fast` skips external research), `--tier hub|spoke|side_page`, `--json`, `--no-wait`. The returned `briefData` is the same structure accepted by `pking seo briefs edit <briefId> --brief-data <json>`.

## Expected Output

- A populated content roadmap tied to keyword clusters.
- Draft articles scored against primary keywords, each with FAQ/definition-first GEO structure.
- A gap-analysis report listing uncovered high-value topics (if run).
- At least one published or scheduled article (if credits + free tier allow).

## Troubleshooting / Errors to Expect

- `INSUFFICIENT_CREDITS` — surface the `checkoutUrl` from the error envelope and stop, or hand off to [`getting-started`](../getting-started/)'s billing section (`billing_list_packs` → `billing_topup`).
- `FREE_CAP_REACHED` on `seo_publish_article` — same treatment as `INSUFFICIENT_CREDITS`.
- `RATE_LIMITED` — back off using the `retryAfter` value from the error envelope before retrying.
- `UNAUTHORIZED` — re-run the login flow described in [`getting-started`](../getting-started/).
- `clusterMeta` missing/empty on a cluster — either no competitors are registered on the brand yet (see [`competitor-intel`](../competitor-intel/)), or the cluster simply doesn't overlap a known rival's keyword footprint (expected for purely core/adjacent clusters).
