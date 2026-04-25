---
name: seo-end-to-end
description: Run SEO from seed keywords to published articles using PostKing. Covers the full pipeline — seeds, expansion, clustering, roadmap, drafting, gap analysis, publish.
icon: https://postking.app/icons/seo.svg
categories: [marketing, writing, data]
free: true
version: 0.1.0
---

# SEO End-to-End

An agentic pipeline that takes a brand from 3–10 seed keywords all the way to published, SEO-targeted blog articles using PostKing.

## Prerequisites

1. PostKing account with credits (free tier works for drafting; publishing is capped on free).
2. A brand already onboarded (use the `onboarding` skill first if not).
3. `mcp.postking.app` connected, or `postking-mcp` installed locally.

## How to use

Ask the agent:

> "Run the PostKing SEO flow for my brand. Seeds: 'AI content generation', 'social scheduling', 'brand voice'."

The agent will call these MCP tools in order:

1. **`seo_add_seeds`** — register your seed keywords.
2. **`seo_generate_keywords`** — expand the seeds (default 100 keywords, burns credits).
3. **`seo_categorize`** — tag by search intent (informational / commercial / navigational / transactional).
4. **`seo_cluster`** — group keywords into topic-pillar clusters.
5. **`seo_list_clusters`** — surface clusters for you to pick (or auto-pick by size/volume).
6. **`seo_generate_roadmap`** — turn the chosen cluster into ~20 prioritized article topics.
7. **`seo_write_article`** — draft articles from roadmap items.
8. **`seo_gap_analysis`** + **`seo_competitor_diff`** — audit what's missing before publishing.
9. **`seo_publish_article`** — publish/schedule to a publication (free-tier choke point).
10. **`seo_roadmap_stats`** — report progress.

## Expected output

- A populated content roadmap tied to keyword clusters.
- 5+ draft blog articles scored against primary keywords.
- A gap-analysis report listing un-covered high-value topics.
- At least one published or scheduled article (if credits + free tier allow).

## Errors to expect

- `INSUFFICIENT_CREDITS` on keyword generation or article drafting — top up at the `checkoutUrl` returned in the error envelope.
- `FREE_CAP_REACHED` on `seo_publish_article` — upgrade via returned `checkoutUrl`.
- `RATE_LIMITED` — retry after `Retry-After` seconds.
