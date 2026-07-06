---
name: reddit
description: Build a fit-scored subreddit pool for a brand, match content to the best communities with posting angles, and natively rewrite a post for a specific subreddit before a human posts it.
version: 0.1.0
compatibility: "Works with any MCP-compatible client connected to postking-mcp (local stdio or hosted at https://mcp.postking.app); the pking CLI is an optional fast path if a shell is available."
metadata:
  icon: https://postking.app/icons/reddit.svg
  free: true
  categories:
    - marketing
    - social-media
    - writing
  hermes:
    tags:
      - reddit
      - social
      - repurpose
      - community
      - marketing

    category: marketing
---

# Reddit Growth

Grow a brand's presence on Reddit the way Reddit actually rewards: find the communities that fit, match a specific piece of content to the best ones with a tailored angle, and rewrite it in native Reddit voice — then hand it to the human to review and post.

## Minimal tool subset

This skill needs only these tools out of PostKing's 200+:

- `reddit_generate_pool` — crawl Reddit and build the brand's fit-scored subreddit pool (async).
- `reddit_get_pool` — read the saved pool, sorted most-relevant first; each row includes `posting_style` and `no_promotion_reason`.
- `reddit_suggest` — match a piece of content to up to 8 best-fit subreddits with posting angles (sync, 0 credits).
- `reddit_rewrite` — natively rewrite a post for a specific subreddit, optionally in a saved voice.
- `reddit_list_posts` — list previously rewritten Reddit posts.
- `list_voices` — get a `voiceId` to pass into `reddit_rewrite`.
- `get_job` — poll `reddit_generate_pool`.

Optional: `reddit_global_pool` — informational stats on the global known-subreddit dataset (not brand-specific). `get_post` — fallback poll if `reddit_rewrite` times out and returns a `postId` instead of a finished draft.

## Prerequisites

1. Authenticated with credits — see [`getting-started`](../getting-started/).
2. A brand already onboarded — see [`getting-started`](../getting-started/).

## How to Use

> "Find subreddits for us and turn this blog post into a Reddit post."

Async tools return `{ operationId, status }` — poll `get_job({ operationId })` until `state` is `completed`, `failed`, `partially_failed`, or `cancelled`.

1. **Build the pool.** `reddit_generate_pool({ brandId })` → async, poll `get_job`. Crawls Reddit and builds the brand's fit-scored subreddit pool. Only needs to run once per brand (re-run occasionally to refresh).
2. **Review the pool.** `reddit_get_pool({ brandId, top })` — the fit-scored subreddit pool, most relevant first. This is the brand-level match: no content needed yet. Each row's `posting_style` and `no_promotion_reason` fields are where you read a community's posting norms and promo tolerance — there is no separate "get subreddit rules" tool, it's surfaced here.
3. **Match content.** `reddit_suggest({ postId })` (an existing BlogArticle id) or `reddit_suggest({ title, content, detail: "medium" })` (pasted content) — returns up to 8 best-fit subreddits from the pool, each with 2–3 posting angles (angle type + tailored title + framing hook), `match_score`, `promotion_mode`, `buyer_intent`, `reason`, and `rule_to_watch`. Use `detail: "medium"` to get all angles plus `reason`/`rule_to_watch`. Requires the pool from step 1 to exist.
4. **(Optional) Pick a voice.** `list_voices({})` → get a `voiceId`, or use `"none"` for no voice.
5. **Rewrite natively.** `reddit_rewrite({ subreddit, voiceId, sourcePostId, angle, length, variations })` (or `sourceContent` + optional `sourceTitle` instead of `sourcePostId`) — `subreddit` and `angle` should come from `reddit_suggest`'s results; `voiceId` is required. Usually returns the finished post inline; if it returns a `postId` instead, poll `get_post({ postId })` until `operationStatus` is `COMPLETED`.
6. **Review saved posts.** `reddit_list_posts({ brandId })` — cursor-paginated list of past rewrites, each with `outputData` (redditTitle, body, notes, wordCount, angle, length, subreddit).

**Important — Reddit is repurpose-to-review, not scheduled publishing.** PostKing does not post to Reddit on the brand's behalf. `reddit_rewrite` produces a review-ready native draft; the human copies it into Reddit and posts it themselves. Always surface the suggested `rule_to_watch` and `promotion_mode` to the user before they post, so they don't get a post removed for breaking a community's rules.

## CLI fast path

The `reddit_*` tools are MCP-only today — there is no `pking` CLI equivalent.

## Expected Output

A fit-scored subreddit pool for the brand, a shortlist of best-fit communities with tailored angles for a given piece of content, and a Reddit-native rewritten draft (with rule/promotion-mode guidance) ready for human review and posting.

## Troubleshooting / Errors to Expect

- Empty pool / "no subreddit pool yet" on `reddit_suggest` or `reddit_rewrite` — run `reddit_generate_pool` and poll `get_job` to completion first.
- Subreddit not in pool on `reddit_rewrite` — pick a `subreddit` value from `reddit_get_pool` or a `reddit_suggest` result rather than typing one in.
- `INSUFFICIENT_CREDITS` — surface the `checkoutUrl` from the error envelope and stop, or hand off to [`getting-started`](../getting-started/)'s billing section (`billing_list_packs` → `billing_topup`).
- `RATE_LIMITED` — back off using the `retryAfter` value from the error envelope before retrying.
- `UNAUTHORIZED` — re-run the login flow described in [`getting-started`](../getting-started/).
