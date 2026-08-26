---
name: postking-storylines
description: Launch a full marketing campaign end-to-end with PostKing — brief, strategy, and multi-channel content generation — using the Storylines tools.
license: MIT
compatibility: "Works with any MCP-compatible client connected to postking-mcp (local stdio or hosted at https://mcp.postking.app/mcp); the pking CLI is an optional fast path when a shell is available."
metadata:
  icon: https://raw.githubusercontent.com/bitsandtea/postking-skills/main/assets/icons/postking-storylines.svg
  category: marketing
---

# PostKing · Storylines

Launch a full marketing campaign from a single goal statement to a batch of scheduled, multi-channel content. PostKing calls this feature **Storylines** internally — an agent-facing "campaign" is a storyline. The tool prefix is `storyline_*` throughout; don't look for a separate "campaign" tool namespace.

## When to Use

Use this skill when a user wants a coordinated, multi-channel push around a single goal or launch — e.g. "launch a campaign for our new pricing page" — that needs a brief, a strategy with multiple content line items across channels, and batch execution into drafts.

## When NOT to Use

Not for a single post, blog, or landing page with no broader campaign structure — route those to the `postking-social`/`postking` skill. Not for SEO-keyword-driven content planning — that's `postking-seo`. Once `storyline_execute` produces drafts, hand off approval/scheduling of individual posts and blogs to `postking`.

## Minimal tool subset

This skill needs these 21 tools out of PostKing's 200+:

- `storyline_list` — list the brand's campaigns (id, title, status) — the entry point for resuming.
- `storyline_get` — fetch one campaign's full state (brief, strategy, line items, status) — use this to figure out where a resumed campaign left off.
- `storyline_create` — start a campaign from a goal prompt.
- `storyline_clarify` — iterative Q&A loop to sharpen positioning/audience/timing/tone.
- `storyline_generate_brief` — draft the campaign brief (async).
- `storyline_set_brief` — manually replace the entire brief object (structured PUT, optimistic concurrency).
- `storyline_edit_brief` — natural-language AI edit of the brief.
- `storyline_confirm_brief` — lock the brief, gating strategy generation.
- `storyline_generate_strategy` — draft the content strategy / line items (async).
- `storyline_get_strategy` — inspect the generated strategy.
- `storyline_edit_strategy` — natural-language AI edit of the strategy/line items as a set.
- `storyline_add_line_item`, `storyline_update_line_item`, `storyline_delete_line_item` — curate individual content pieces (group of three, used together).
- `storyline_regenerate_line_item` — re-generate a single line item's draft (async) instead of manually editing it.
- `storyline_estimate` — dry-run credit cost before executing.
- `storyline_execute` — generate all selected drafts as posts/blogs (async).
- `storyline_update` — edit campaign metadata (title, isLive, start/end/launch dates).
- `storyline_delete` — archive (soft-delete) a campaign.
- `storyline_restore` — bring an archived campaign back to active.
- `get_job` — poll any async operation.

## Prerequisites

1. Authenticated with credits — see the `postking-getting-started` skill.
2. A brand already onboarded — see the `postking-getting-started` skill.

## Procedure

> "Launch a campaign for our new pricing page. Goal: drive signups from existing free users."

1. **Create.** `storyline_create({ brandId, prompt: "<campaign goal>", title: "<optional title>" })` — starts the campaign with a plain-language description of the goal.
2. **Clarify (loop).** `storyline_clarify({ storylineId, ... })` — the system asks follow-up questions about positioning, audience, timing, and tone. Call this iteratively, feeding back the user's answers each round, until it signals readiness to proceed. Don't skip this — a thin brief produces a thin strategy.
3. **Generate brief.** `storyline_generate_brief({ storylineId })` → async, poll `get_job` (typically 1–3 min). Produces positioning, key messages, audience, timing, proof points, tone notes, and links.
4. **Review and edit.** Inspect the brief; if it needs adjustment, `storyline_edit_brief({ storylineId, instruction: "<natural language edit>" })` — this is an AI edit pass, not a structured PATCH. Prefer this over `storyline_set_brief` for normal adjustments. Reach for `storyline_set_brief({ storylineId, brief, expectedVersion })` only when you have a *complete* brief object to write wholesale — e.g. a human-authored brief supplied up front (skipping `storyline_generate_brief` entirely), or writing back a fully-formed object you already fetched and modified structurally. It replaces the whole brief and needs every field (`positioning`, `audience`, `keyMessages`, `timing`, `proofPoints`, `dos`, `donts`, `toneNotes`, `links`) plus the current `expectedVersion` (`0` when setting from scratch).
5. **Confirm.** `storyline_confirm_brief({ storylineId })` — locks the brief. This is a hard gate: strategy generation will not run on an unconfirmed brief.
6. **Generate strategy.** `storyline_generate_strategy({ storylineId })` → async, poll `get_job` (typically 2–5 min). Produces line items — individual content pieces spread across channels (social posts, blog articles, etc.) that together execute the campaign.
7. **Inspect.** `storyline_get_strategy({ storylineId })` — review the full line-item list.
8. **Curate.** Add, adjust, or remove line items as needed: `storyline_add_line_item`, `storyline_update_line_item`, `storyline_delete_line_item` (requires `confirm: true` — this is a destructive action, confirm with the user before calling it). Use `storyline_regenerate_line_item` if a single item needs a fresh take rather than a manual edit. For a broad change across the whole line-item set (e.g. "add a LinkedIn video for launch week", "drop the influencer outreach items"), use `storyline_edit_strategy({ storylineId, instruction: "<natural language edit>" })` instead of doing it item-by-item — it's the strategy-level analog of `storyline_edit_brief`.
9. **Estimate.** `storyline_estimate({ storylineId })` — dry-run the credit cost of executing the current line-item set. Show this to the user before executing, especially if the balance is close to the cost.
10. **Execute.** `storyline_execute({ storylineId })` → async, poll `get_job` (typically 3–8 min). Generates all selected drafts as posts and blogs in one batch.
11. **Review output.** `list_posts` and `list_blogs` — pull the generated drafts to show the user what the campaign produced. These are drafts; approve/schedule them through the normal post/blog approval tools as with any other generated content.

## Resume an existing campaign

> "What's the status of our pricing-page campaign?" / "Continue the campaign we started last week."

Don't assume a fresh `storyline_create` — most requests to touch a campaign are about one that already exists.

1. **List.** `storyline_list({ brandId })` — short detail gives `{ id, title, status }` for every campaign on the brand. Find the one the user means (or ask, if ambiguous).
2. **Get full state.** `storyline_get({ storylineId, detail: "full" })` — returns the campaign's brief, strategy, line items, and `status` in one call. This replaces guessing at where things left off.
3. **Resume from the right step**, based on `status` (the same state machine steps 1–10 walk through, in order — never skip ahead of it):
   - `intake` — no confirmed brief yet. Continue the clarify loop (step 2) if it's still gathering context, or go straight to `storyline_generate_brief` (step 3) if enough context already exists.
   - `brief_ready` — a brief exists but isn't executing yet. Review it; if it's not yet confirmed, edit with `storyline_edit_brief`/`storyline_set_brief` (step 4) and confirm with `storyline_confirm_brief` (step 5); if already confirmed, move on to `storyline_generate_strategy` (step 6).
   - `strategy_ready` — a strategy exists. `storyline_get_strategy` (step 7) to see the line items, curate (step 8), then `storyline_estimate` → `storyline_execute` (steps 9–10).
   - `executing` — a batch is running. Find the in-flight operation and poll `get_job` rather than re-triggering `storyline_execute`.
   - `completed` — nothing left to run. Jump to step 11 (`list_posts`/`list_blogs`) to show what it produced, or use `storyline_update` for metadata touch-ups.
   - `archived` — soft-deleted. `storyline_restore({ storylineId })` first, then re-check `status` and resume normally.
   - `failed` — something errored mid-flow. Use the full `storyline_get` payload to see which stage failed and retry that specific async step (`storyline_generate_brief`/`storyline_generate_strategy`/`storyline_execute`) rather than starting over.
4. **Housekeeping tools**, usable at any point in the flow: `storyline_update` to rename or reschedule a campaign, `storyline_delete` (requires `confirm: true`) to archive one the user no longer wants, `storyline_restore` to bring one back.

## CLI fast path

Storylines/campaigns are MCP-only today — there is no `pking` CLI equivalent for the `storyline_*` tools. Drive this skill via MCP tool calls; use the `postking` skill's CLI fast path for reviewing/approving the resulting post and blog drafts once `storyline_execute` completes.

## Expected Output

- A confirmed campaign brief (positioning, audience, timing, tone, proof points).
- A strategy with multiple content line items spanning channels.
- A batch of generated post/blog drafts, ready for review and scheduling.

## Pitfalls

- `INSUFFICIENT_CREDITS` — check `storyline_estimate` before executing; if you hit this mid-flow, point the user to the `postking-getting-started` skill's billing section (`billing_list_packs` → `billing_topup`).
- `RATE_LIMITED` — back off using the `retryAfter` value from the error envelope before retrying.
- `UNAUTHORIZED` — re-run the login flow described in the `postking-getting-started` skill.
- Strategy generation attempted before brief confirmation — `storyline_confirm_brief` must succeed first; the strategy step is gated on a confirmed brief.
- Calling any campaign-mutating tool on an `archived` campaign — `storyline_restore` it first.
- `storyline_set_brief` version conflict — it takes `expectedVersion` for optimistic concurrency; re-fetch with `storyline_get` and retry with the current version if it's rejected.

## Verification

- `storyline_get_strategy({ storylineId })` should return the line-item list after `storyline_generate_strategy` completes.
- `list_posts`/`list_blogs` should show the new drafts after `storyline_execute` finishes.
