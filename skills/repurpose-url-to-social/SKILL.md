---
name: repurpose-url-to-social
description: Turn any URL, blog post, or long text into platform-tailored, voice-matched social posts, then schedule them.
version: 0.2.0
compatibility: "Requires Node 18+ and the postking-cli npm package. Auto-installs on first use if missing."
metadata:
  icon: https://postking.app/icons/repurpose.svg
  free: true
  categories:
    - marketing
    - writing
  hermes:
    tags:
      - repurpose
      - social
      - content
      - marketing
      - writing

    category: marketing
---

# Repurpose URL to Social

Feed PostKing a link or paragraph, get scheduled social posts on the platforms you care about.

## Setup

**Setup.** This skill requires the `pking` CLI. If `command -v pking` fails, run `npm i -g postking-cli`. To authorize, run `pking login` and follow the URL it prints. You only need to do this once per machine.

## Prerequisites

- Active brand with social accounts connected.
- URL, plain text, or an existing blog/post ID.

## How to Use

> "Repurpose https://example.com/my-post into a LinkedIn and X thread. Schedule LinkedIn for tomorrow 9am UTC."

The agent:

1. **`pking social check`** — confirm which platforms are connected.
2. **`pking repurpose --source-type url --source-url <url> --target-type social --target-platforms linkedin,x [--voice <id_or_per_platform_map>] [--variations 2] [--include-link]`**
   PostKing crawls the URL internally — do NOT fetch it yourself. Returns one post ID per platform.
   - For text input: `--source-type text --source-content "<text>"`
   - For a blog: `--source-type blog --source-blog <articleId>`
   - For an existing post: `--source-type social_post --source-post <postId>`
   - Per-platform voices: `--voice "linkedin:<voiceId>,x:<voiceId>"`
3. **`pking posts view <postId>`** — inspect each generated draft.
4. **`pking posts approve <postId> --schedule <iso>`** — schedule each post. Free-tier choke point.
5. **`pking posts calendar`** — confirm the posts appear in the upcoming schedule.

## Notes

- Do NOT fetch the URL yourself — PostKing crawls it internally for structured extraction.
- Use `--include-link` to keep the source URL in the generated post body.
- Voice profiles: `pking voice list --platform linkedin` to find IDs.

## Errors to Expect

- `FREE_CAP_REACHED` on `pking posts approve` — surface the `checkoutUrl` and stop.
- `NOT_FOUND` if the source URL is unreachable by PostKing's crawler.
