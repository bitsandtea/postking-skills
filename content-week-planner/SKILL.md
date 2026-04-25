---
name: content-week-planner
description: Plan, generate, and approve a full week of social posts across all connected platforms in one go.
icon: https://postking.app/icons/calendar.svg
categories: [marketing, productivity]
free: true
version: 0.1.0
---

# Content Week Planner

Generate a themed, spaced-out week of social posts with one prompt.

## Prerequisites

- Active brand with at least one connected social account (`check_social_accounts`).
- Some credits for generation.

## How to use

> "Plan my next 7 days of content. 2 posts/day on LinkedIn and X."

The agent:

1. **`check_social_accounts`** — confirms which platforms are connected.
2. **`smart_week`** (or `generate_batch`) — creates the weekly draft plan.
3. **`list_posts`** with `status: created` — surface the generated drafts.
4. **`get_post`** on any ID — inspect individual drafts.
5. **`approve_post`** with `scheduledAt` — lock each draft to a time. This is the free-tier choke point.
6. **`get_calendar`** — verify the final schedule.

## Expected output

- A structured 7-day calendar with staggered, platform-tailored posts.
- Variations per platform (e.g. LinkedIn long-form vs. X short).
- Confirmation that approved posts are in the schedule.

## Errors to expect

- `FREE_CAP_REACHED` on `approve_post` / `post_now` — upgrade link in the envelope.
- `INSUFFICIENT_CREDITS` during generation.
