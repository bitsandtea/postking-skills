---
name: content-week-planner
description: Plan, generate, and approve a full week of social posts across all connected platforms in one go.
version: 0.2.0
compatibility: "Requires Node 18+ and the postking-cli npm package. Auto-installs on first use if missing."
metadata:
  icon: https://postking.app/icons/calendar.svg
  free: true
  categories:
    - marketing
    - productivity
  hermes:
    tags:
      - content
      - scheduling
      - calendar
      - social
      - marketing

    category: marketing
---

# Content Week Planner

Generate a themed, spaced-out week of social posts with one prompt.

## Setup

**Setup.** This skill requires the `pking` CLI. If `command -v pking` fails or `pking --version` is older than 1.0.0, follow `references/install.md`. To authorize, run `pking login` and follow the URL it prints. You only need to do this once per machine.

## Prerequisites

- Active brand with at least one connected social account (`pking social check`).
- Some credits for generation.

## How to Use

> "Plan my next 7 days of content. 2 posts/day on LinkedIn and X."

The agent:

1. **`pking social check`** — confirms which platforms are connected.
2. **`pking weekly-schedule get`** — view the current cadence (if any).
3. **`pking weekly-schedule set --monday "linkedin:1,x:1" --tuesday "linkedin:1,x:1" ... --timezone America/New_York --enable`** — define the weekly cadence; repeat `--day` flags for each active day.
4. **`pking weekly-schedule run-day --date <YYYY-MM-DD>`** — generate all posts for a specific day; repeat per day or let the schedule run automatically.
5. **`pking posts list --status created`** — surface the generated drafts.
6. **`pking posts view <postId>`** — inspect individual drafts.
7. **`pking posts approve <postId> --variation 1 --schedule <iso>`** — lock each draft to a time slot. This is the free-tier choke point.
8. **`pking posts calendar --days 7`** — verify the final schedule.

## Expected Output

- A structured 7-day calendar with staggered, platform-tailored posts.
- Variations per platform (e.g. LinkedIn long-form vs. X short).
- Confirmation that approved posts appear in the schedule.

## Errors to Expect

- `FREE_CAP_REACHED` on `pking posts approve` — surface the `checkoutUrl` from the error envelope and stop.
- `INSUFFICIENT_CREDITS` during generation — same treatment.
