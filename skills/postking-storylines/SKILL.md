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

This skill only needs these 13 tools out of PostKing's 200+:

- `storyline_create` — start a campaign from a goal prompt.
- `storyline_clarify` — iterative Q&A loop to sharpen positioning/audience/timing/tone.
- `storyline_generate_brief` — draft the campaign brief (async).
- `storyline_edit_brief` — natural-language AI edit of the brief.
- `storyline_confirm_brief` — lock the brief, gating strategy generation.
- `storyline_generate_strategy` — draft the content strategy / line items (async).
- `storyline_get_strategy` — inspect the generated strategy.
- `storyline_add_line_item`, `storyline_update_line_item`, `storyline_delete_line_item` — curate individual content pieces (group of three, used together).
- `storyline_estimate` — dry-run credit cost before executing.
- `storyline_execute` — generate all selected drafts as posts/blogs (async).
- `get_job` — poll any async operation.

## Prerequisites

1. Authenticated with credits — see the `postking-getting-started` skill.
2. A brand already onboarded — see the `postking-getting-started` skill.

## Procedure

> "Launch a campaign for our new pricing page. Goal: drive signups from existing free users."

1. **Create.** `storyline_create({ brandId, prompt: "<campaign goal>", title: "<optional title>" })` — starts the campaign with a plain-language description of the goal.
2. **Clarify (loop).** `storyline_clarify({ storylineId, ... })` — the system asks follow-up questions about positioning, audience, timing, and tone. Call this iteratively, feeding back the user's answers each round, until it signals readiness to proceed. Don't skip this — a thin brief produces a thin strategy.
3. **Generate brief.** `storyline_generate_brief({ storylineId })` → async, poll `get_job` (typically 1–3 min). Produces positioning, key messages, audience, timing, proof points, tone notes, and links.
4. **Review and edit.** Inspect the brief; if it needs adjustment, `storyline_edit_brief({ storylineId, instructions: "<natural language edit>" })` rather than hand-editing fields — this is an AI edit pass, not a structured PATCH.
5. **Confirm.** `storyline_confirm_brief({ storylineId })` — locks the brief. This is a hard gate: strategy generation will not run on an unconfirmed brief.
6. **Generate strategy.** `storyline_generate_strategy({ storylineId })` → async, poll `get_job` (typically 2–5 min). Produces line items — individual content pieces spread across channels (social posts, blog articles, etc.) that together execute the campaign.
7. **Inspect.** `storyline_get_strategy({ storylineId })` — review the full line-item list.
8. **Curate.** Add, adjust, or remove line items as needed: `storyline_add_line_item`, `storyline_update_line_item`, `storyline_delete_line_item` (requires `confirm: true` — this is a destructive action, confirm with the user before calling it). Use `storyline_regenerate_line_item` if a single item needs a fresh take rather than a manual edit.
9. **Estimate.** `storyline_estimate({ storylineId })` — dry-run the credit cost of executing the current line-item set. Show this to the user before executing, especially if the balance is close to the cost.
10. **Execute.** `storyline_execute({ storylineId })` → async, poll `get_job` (typically 3–8 min). Generates all selected drafts as posts and blogs in one batch.
11. **Review output.** `list_posts` and `list_blogs` — pull the generated drafts to show the user what the campaign produced. These are drafts; approve/schedule them through the normal post/blog approval tools as with any other generated content.

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

## Verification

- `storyline_get_strategy({ storylineId })` should return the line-item list after `storyline_generate_strategy` completes.
- `list_posts`/`list_blogs` should show the new drafts after `storyline_execute` finishes.
