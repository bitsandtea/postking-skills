---
name: seo-end-to-end
description: Run SEO from seed keywords to published articles using PostKing. Covers the full pipeline — seeds, expansion, clustering, roadmap, drafting, gap analysis, publish.
version: 0.2.0
compatibility: "Requires Node 18+ and the postking-cli npm package. Auto-installs on first use if missing."
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
      - content
      - marketing
      - keywords
      - blogging

    category: marketing
---

# SEO End-to-End

An agentic pipeline that takes a brand from 3–10 seed keywords all the way to published, SEO-targeted blog articles using PostKing.

## Setup

**Setup.** This skill requires the `pking` CLI. If `command -v pking` fails, run `npm i -g postking-cli`. To authorize, run `pking login` and follow the URL it prints. You only need to do this once per machine.

## Prerequisites

1. PostKing account with credits (free tier works for drafting; publishing is capped on free).
2. A brand already onboarded (use the `onboarding` skill first if not).

## How to Use

> "Run the PostKing SEO flow for my brand. Seeds: 'AI content generation', 'social scheduling', 'brand voice'."

The agent runs these commands in order:

1. **`pking seo seeds "AI content generation" "social scheduling" "brand voice"`** — register seed keywords.
2. **`pking seo generate`** — expand seeds into ~100 keywords. Long-running; the CLI polls automatically.
3. **`pking seo categorize`** — tag each keyword by search intent (informational / commercial / navigational / transactional).
4. **`pking seo cluster`** — group keywords into topic-pillar clusters.
5. **`pking seo clusters list`** — surface clusters; ask the user which to target (or auto-pick by size/volume).
6. **`pking seo roadmap --cluster <clusterId> --items 20`** — produce ~20 prioritized article topics from the chosen cluster.
7. **`pking seo write --roadmap-id <roadmapItemId>`** — draft an article from a roadmap item. Repeat per item.
8. **`pking seo gap`** + **`pking seo competitor --domain <competitorDomain>`** — audit what's missing before publishing.
9. **`pking seo publish --article-id <articleId> [--publication <id>] [--schedule <iso>]`** — publish or schedule to a publication. Free-tier choke point.
10. **`pking seo stats`** — report roadmap progress.

## Expected Output

- A populated content roadmap tied to keyword clusters.
- 5+ draft blog articles scored against primary keywords.
- A gap-analysis report listing uncovered high-value topics.
- At least one published or scheduled article (if credits + free tier allow).

## Errors to Expect

- `INSUFFICIENT_CREDITS` on keyword generation or article drafting — surface the `checkoutUrl` from the error envelope and stop.
- `FREE_CAP_REACHED` on `pking seo publish` — same treatment.
- `RATE_LIMITED` — back off at least 30 seconds (use the `retryAfter` value from the error envelope).
