---
name: postking-reddit
description: Build a fit-scored subreddit pool for a brand, match content to the best communities with posting angles, and natively rewrite a post for a specific subreddit before a human posts it.
license: MIT
compatibility: "Works with any MCP-compatible client connected to postking-mcp (local stdio or hosted at https://mcp.postking.app/mcp); the pking CLI is an optional fast path when a shell is available."
metadata:
  icon: https://raw.githubusercontent.com/bitsandtea/postking-skills/main/assets/icons/postking-reddit.svg
  category: marketing
---

# Reddit Growth

Grow a brand's presence on Reddit the way Reddit actually rewards: find the communities that fit, match a specific piece of content to the best ones with a tailored angle, and rewrite it in native Reddit voice — then hand it to the human to review and post.

## When to Use

Use this skill when a user wants to find subreddits that fit their brand, match an existing piece of content (blog post, post draft) to the best-fit communities with a posting angle, or get a Reddit-native rewrite of content before posting it themselves.

## When NOT to Use

Not for automatically publishing to Reddit — PostKing never posts on the brand's behalf; this skill only prepares review-ready drafts. Not for other-platform content generation or scheduling — that's `postking-social`/`postking`. Not for applying a saved voice profile in general — that's `postking-brand-voice` (this skill only consumes a `voiceId` from it).

## Minimal tool subset

This skill needs only these tools out of PostKing's 200+:

- `reddit_generate_pool` — crawl Reddit and build the brand's fit-scored subreddit pool from brand signals (async).
- `reddit_get_pool` — read the saved pool in the order the API returns it (subs that allow promotion first, then by brand fit); each row has `brand_fit` (0-10), `activity`, `tiny`, `promo_policy`/`promo_label`, `posting_style` and `no_promotion_reason`.
- `reddit_discover_subreddits` — search Reddit with explicit queries for candidates beyond the brand-signal pool (sync, 15 credits flat).
- `reddit_add_to_pool` — add chosen subreddits to the pool by name (sync, 0 credits).
- `reddit_remove_from_pool` — prune subreddits from the pool by name (sync, 0 credits).
- `reddit_suggest` — match a piece of content to up to 8 best-fit subreddits with posting angles (sync, 0 credits). Optional `founderContext`.
- `reddit_rewrite` — natively rewrite a post for a specific subreddit, optionally in a saved voice; every draft is checked and comes back passed, needs_review or blocked.
- `reddit_list_posts` — list previously rewritten Reddit posts.
- `list_voices` — get a `voiceId` to pass into `reddit_rewrite`.
- `get_job` — poll `reddit_generate_pool`.

Optional: `reddit_global_pool` — informational stats on the global known-subreddit dataset (not brand-specific). `get_post` — fallback poll if `reddit_rewrite` times out and returns a `postId` instead of a finished draft.

## Prerequisites

1. Authenticated with credits — see the `postking-getting-started` skill.
2. A brand already onboarded — see the `postking-getting-started` skill.

## Procedure

> "Find subreddits for us and turn this blog post into a Reddit post."

Async tools return `{ operationId, status }` — poll `get_job({ operationId })` until `state` is `completed`, `failed`, `partially_failed`, or `cancelled`.

1. **Build the pool.** `reddit_generate_pool({ brandId })` → async, poll `get_job`. Crawls Reddit and builds the brand's fit-scored subreddit pool from brand signals alone. Only needs to run once per brand (re-run occasionally to refresh). See "Manual subreddit curation" below for the alternative/companion path when the user wants to search in a specific direction instead of (or in addition to) this brand-signal crawl.
2. **Review the pool.** `reddit_get_pool({ brandId, top })` — the brand-level match: no content needed yet. Rows keep the API's order and are not re-sorted. Read each row like this:
   - `brand_fit` (0-10) — how well the community's readers match the brand. This is the relevance number.
   - `activity` — status plus "last post N days ago". `tiny: true` means under 500 members.
   - `promo_policy` / `promo_label` — null label means promotion is allowed; "No promotion" or "Megathread only" means a standalone post must not promote the brand.
   - `posting_style` and `no_promotion_reason` — the community's posting norms. There is no separate "get subreddit rules" tool; it's surfaced here.
   Two rules apply everywhere: a community under **500 members** is not worth a post, and one with **no post in the last 30 days** is inactive. Both are flagged in `warnings`, and `reddit_suggest` leaves them out. Tell the user plainly when a sub they care about is one of these.
3. **Match content.** `reddit_suggest({ postId })` (an existing BlogArticle id) or `reddit_suggest({ title, content, detail: "medium" })` (pasted content) — returns up to 8 best-fit subreddits from the pool, each with 2–3 posting angles (angle type + tailored title + framing hook), `match_score`, `promotion_mode`, `buyer_intent`, `reason`, `rule_to_watch`, `activity` and `past_removals`. Use `detail: "medium"` to get all angles plus `reason`, `rule_to_watch` and `restrictions`. Requires the pool from step 1 to exist.
   - **Order:** subs that allow promotion come first; restricted subs (no promotion, megathread only) follow, demoted but still listed, each with a `promo_label` and the quoted rule in `restrictions`. Each group is ordered by score. Say which ones are restricted when presenting the list.
   - **Left out:** subs under 500 members or inactive. The response's `excluded` counts say how many.
   - **Removals:** if `removal_history` is `never_synced`, removals are not tracked until the browser extension has synced the user's Reddit account, so `past_removals` shows 0 everywhere. Tell the user that once.
   - **Framing:** pass `founderContext` only when the user has given real first-person facts (a founder story, real numbers). Without it, angles use customer or observation framing, never a made-up "I" story. Never invent founder context.
4. **(Optional) Pick a voice.** `list_voices({})` → get a `voiceId`, or use `"none"` for no voice. See the `postking-brand-voice` skill for details on managing voice profiles.
5. **Rewrite natively.** `reddit_rewrite({ subreddit, voiceId, sourcePostId, angle, length, variations })` (or `sourceContent` + optional `sourceTitle` instead of `sourcePostId`) — `subreddit` and `angle` should come from `reddit_suggest`'s results; `voiceId` is required. Pass `founderContext` only with real facts the user supplied; without it the draft is an observation, how-to or question, never a first-person story. Usually returns the finished post inline; if it returns a `postId` instead, poll `get_post({ postId })` until `operationStatus` is `COMPLETED`.
   Every draft is checked against the subreddit's stated rules and promotion policy, and ends in one of three states:
   - **passed** — saved approved. Ready for the user to review and post.
   - **needs review** — quality issues survived automatic repair (the `issues` list says which). NOT approved. Do not post it as it is; edit it or rewrite again.
   - **blocked** — the text breaks a rule the subreddit states, or its promotion policy. `rule_breaks` gives the rule and the quoted words. NOT approved. Do not post it as it is; change the text or pick another subreddit.
   The result shows `counts` (passed / needsReview / blocked) and each draft's `check`. Tell the user which drafts are not ready and why.
   `approve_post` on a needs-review or blocked draft returns 409 `CHECK_OVERRIDE_REQUIRED` with the reasons. Show them to the user and stop. Retry with `override: true` only if the user explicitly says to approve that draft anyway; never override on your own.
6. **Review saved posts.** `reddit_list_posts({ brandId })` — cursor-paginated list of past rewrites, each with `status`, a `check` (passed / needs_review / blocked, with `rule_breaks` or `issues`), `ready_to_post`, and `outputData` (redditTitle, body, notes, wordCount, angle, length, subreddit). Only `passed` drafts are approved.

**Important — Reddit is repurpose-to-review, not scheduled publishing.** PostKing does not post to Reddit on the brand's behalf. `reddit_rewrite` produces a review-ready native draft; the human copies it into Reddit and posts it themselves. Always surface the suggested `rule_to_watch`, `promotion_mode` and any `restrictions` to the user before they post, so they don't get a post removed for breaking a community's rules.

## Manual subreddit curation

`reddit_generate_pool` only finds subreddits from brand signals. Use this path instead (or alongside it) when the user wants to search in a specific direction — a competitor's niche, a product category, a named community they've heard of.

1. **Discover.** `reddit_discover_subreddits({ queries, context, minSubs })` — 1–8 explicit search queries, sync, 15 credits flat per call regardless of cache state. Pass `context` to disambiguate ambiguous terms (e.g. a brand name that also means something else). `minSubs` defaults to 500. Returns candidates sorted by subscriber count, each flagged `already_in_pool` and `already_cached`, with `activity` and `tiny`; say plainly when a candidate is under 500 members or inactive (no post in 30 days). **This only returns candidates — it does not add anything to the pool.**
2. **Add.** `reddit_add_to_pool({ names })` — add the user's picks by name, 1–25 per call, free. Each name is resolved live against Reddit and merged into the pool. Partial success is normal, not an error: each added row shows `brand_fit`, `activity` and `tiny` — a sub under 500 members or inactive is still added, so warn the user. Check the returned `failed` array — `retryable: true` means a transient lookup failure (retry the call), `retryable: false` means the subreddit itself can't be added (nonexistent/banned/private).
3. **Remove.** `reddit_remove_from_pool({ names })` — prune by name, 1–50 per call, free, case-insensitive. Brand-scoped unsubscribe only — it never touches the platform's global subreddit cache, so re-adding later is cheap. Names not currently in the pool come back in `notFound`, which is still success.

Note: `reddit_rewrite` auto-adds whatever subreddit(s) it targets to the pool at rewrite time, before the draft is even reviewed — so `reddit_add_to_pool` isn't a prerequisite for rewriting. Use it when curating the pool ahead of time, independent of any single rewrite, and use `reddit_remove_from_pool` afterward to prune anything a rewrite auto-added that the user doesn't want kept.

## CLI fast path

The `pking` CLI has `pking reddit pool` (brand fit, members, activity, promotion rules) and `pking reddit rewrite --subreddit <name> --seed <text>` (prints each draft's check). Discovery, suggest and pool edits are MCP-only.

## Expected Output

A subreddit pool scored for brand fit and activity, a shortlist of best-fit communities with tailored angles for a given piece of content, and a Reddit-native rewritten draft (with rule/promotion-mode guidance) ready for human review and posting.

## Pitfalls

- Empty pool / "no subreddit pool yet" on `reddit_suggest` or `reddit_rewrite` — run `reddit_generate_pool` and poll `get_job` to completion first.
- Expecting `reddit_discover_subreddits` to save results — it only returns candidates; call `reddit_add_to_pool` with the chosen names to actually add them.
- Retrying `approve_post` with `override: true` without the user's explicit go-ahead.
- Posting a `needs_review` or `blocked` draft as it is — only `passed` drafts are approved.
- Inventing founder context to make a draft sound personal — pass `founderContext` only with facts the user gave.
- Treating `past_removals: 0` as clean when `removal_history` is `never_synced` — removals are not tracked yet.
- Subreddit not in pool on `reddit_rewrite` — pick a `subreddit` value from `reddit_get_pool` or a `reddit_suggest` result rather than typing one in.
- `INSUFFICIENT_CREDITS` — surface the `checkoutUrl` from the error envelope and stop, or hand off to the `postking-getting-started` skill's billing section (`billing_list_packs` → `billing_topup`).
- `RATE_LIMITED` — back off using the `retryAfter` value from the error envelope before retrying.
- `UNAUTHORIZED` — re-run the login flow described in the `postking-getting-started` skill.

## Verification

- `reddit_get_pool({ brandId })` should return a non-empty pool after `reddit_generate_pool` completes.
- `reddit_list_posts({ brandId })` should include the new draft after `reddit_rewrite` finishes.
