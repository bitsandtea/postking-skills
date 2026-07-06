---
name: competitor-intel
description: Discover, register, and analyze a brand's competitors in PostKing — probe for rival domains, classify candidates, add them as tracked competitors, and pull comparison/overview intelligence.
version: 0.3.0
compatibility: "Works with any MCP-compatible client connected to postking-mcp (local stdio or hosted at https://mcp.postking.app); the pking CLI is an optional fast path if a shell is available."
metadata:
  icon: https://postking.app/icons/competitor-intel.svg
  free: true
  categories:
    - marketing
    - data
    - seo
  hermes:
    tags:
      - seo
      - competitors
      - marketing
      - research
      - intel

    category: marketing
---

# Competitor Intelligence

Build a competitive-intelligence picture for a brand in PostKing — discover rivals, classify them, register and analyze them, and review the resulting comparison/overview data.

Once competitors are registered on the brand, PostKing's SEO clustering automatically becomes competitor-aware — see the [`seo`](../seo/) skill for how registered competitors then shape keyword clustering (product-fit classification + competitor "conquest" brief plans).

## Minimal tool subset

This skill needs 7 core tools:

- `competitor_probe` — discover candidate rival domains for the brand.
- `competitor_probe_status` — poll the probe.
- `competitor_probe_classify` — classify each candidate as direct / similar / not_relevant.
- `competitor_add` — register up to 20 domains as tracked competitors (async).
- `competitor_analyze` — re-run analysis on pending/failed competitor rows (async).
- `competitor_list` — review registered competitors and their status.
- `get_job` — poll any async operation.

**Going deeper (optional)** — read/refresh intelligence on already-registered competitors:

- `competitor_get_comparison` — head-to-head keyword comparison against a competitor.
- `competitor_recompute_comparison` — refresh that comparison.
- `competitor_get_overview` — AI-synthesized landscape summary.
- `competitor_generate_overview` — (re)generate that overview (async).
- `competitor_comparison_sources` — the SEO sources feeding a comparison.
- `competitor_update` — edit a competitor row (e.g. toggle exclusion flags).
- `competitor_delete` — remove a competitor.

## Prerequisites

1. Authenticated with credits — see [`getting-started`](../getting-started/).
2. A brand already onboarded — see [`getting-started`](../getting-started/).

## How to Use

> "Find our competitors and register them."

Async tools return `{ operationId, status }` — poll `get_job({ operationId })` until `state` is `completed`, `failed`, `partially_failed`, or `cancelled`. `competitor_probe` returns `{ started: true }` — poll `competitor_probe_status` instead.

1. **Probe.** `competitor_probe({ brandId })` — kicks off discovery of candidate rival domains. Returns `{ started: true }`.
2. **Poll.** `competitor_probe_status({ brandId })` — poll until the probe completes.
3. **Classify.** `competitor_probe_classify({ brandId })` — classifies each discovered candidate as `direct`, `similar`, or `not_relevant`. Review the results with the user before adding — don't blindly register every candidate.
4. **Add.** `competitor_add({ brandId, domains: [...] })` — registers up to 20 domains as tracked competitors. Async, poll `get_job`.
5. **Analyze.** `competitor_analyze({ brandId })` — runs deeper analysis on any pending/failed competitor rows. Async, poll `get_job`.
6. **Review.** `competitor_list({ brandId, detail: "medium" })` — status + analysis state per competitor.
7. **(Optional) Go deeper.** `competitor_get_comparison` / `competitor_recompute_comparison` for head-to-head keyword comparison against a specific competitor; `competitor_get_overview` / `competitor_generate_overview` (async, poll `get_job`) for an AI-synthesized landscape summary; `competitor_comparison_sources` for the SEO sources feeding the comparison; `competitor_update` (e.g. toggle exclusion flags) or `competitor_delete` to manage the list.

Next: with competitors registered, run the SEO pipeline in the [`seo`](../seo/) skill — clustering will now automatically flag competitor-dominated clusters and draft conquest briefs.

## CLI fast path

Competitor tools (`competitor_*`) are MCP-only today — there is no `pking` CLI equivalent.

## Expected Output

A set of registered, classified, analyzed competitors on the brand, plus (optionally) a saved head-to-head comparison and landscape overview.

## Troubleshooting / Errors to Expect

- `INSUFFICIENT_CREDITS` — point the user to [`getting-started`](../getting-started/)'s billing section (`billing_list_packs` → `billing_topup`).
- `RATE_LIMITED` — back off using the `retryAfter` value from the error envelope before retrying.
- `UNAUTHORIZED` — re-run the login flow described in [`getting-started`](../getting-started/).
