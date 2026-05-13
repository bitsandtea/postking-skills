---
name: blog-publish-pipeline
description: Generate a blog article, iterate with AI edits, and publish it to PostKing or to external platforms (WordPress, Medium, Substack).
version: 0.2.0
compatibility: "Requires Node 18+ and the postking-cli npm package. Auto-installs on first use if missing."
metadata:
  icon: https://postking.app/icons/blog.svg
  free: true
  categories:
    - writing
    - marketing
  hermes:
    tags:
      - blog
      - writing
      - publishing
      - content
      - marketing

    category: marketing
---

# Blog Publish Pipeline

Prompt-to-published in a handful of CLI calls.

## Setup

**Setup.** This skill requires the `pking` CLI. If `command -v pking` fails, run `npm i -g postking-cli`. To authorize, run `pking login` and follow the URL it prints. You only need to do this once per machine.

## Prerequisites

- Active brand.
- Optional: voice profile ID (`pking voice list --platform blog`).
- Optional: an external publishing connection (WordPress / Medium / Substack — connect via the PostKing dashboard).

## How to Use

> "Draft a 1500-word blog post about 'AI content calendars' in my brand voice and publish it."

Flow:

1. **`pking publications list`** — find an existing publication, or create one:
   `pking publications create --title "My Blog" --description "..."`
2. **`pking blogs generate --publication <id> --topic "AI content calendars" --length medium --keywords "ai,content calendar,scheduling" [--voice <voiceProfileId>] --wait`**
   Async; `--wait` blocks until the draft is ready. Without `--wait`, poll with `pking blogs status <articleId>`.
3. **`pking blogs view <articleId>`** — review the draft.
4. Iterate as needed — re-run `pking blogs generate` with refined topic/keywords, or use `pking editor rewrite --text "..." --platform blog` to polish sections manually.
5. Publish:
   - **PostKing hosted**: `pking blogs publish <articleId>`
   - **External connections**: `pking blogs publish <articleId> --connections <connectionId1,connectionId2>`
6. **`pking blogs view <articleId>`** again to confirm `status: published` and capture the live URL.

## Expected Output

- Published article URL on PostKing or on connected external platforms.
- SEO metadata populated (meta title, description, slug).

## Errors to Expect

- `FREE_CAP_REACHED` on publish — surface the `checkoutUrl` from the error envelope and stop.
- `VALIDATION` when required fields (title, content) are missing.
