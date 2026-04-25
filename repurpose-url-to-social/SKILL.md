---
name: repurpose-url-to-social
description: Turn any URL, blog post, or long text into platform-tailored, voice-matched social posts, then schedule them.
icon: https://postking.app/icons/repurpose.svg
categories: [marketing, writing]
free: true
version: 0.1.0
---

# Repurpose URL to Social

Feed PostKing a link or paragraph, get scheduled social posts on the platforms you care about.

## Prerequisites

- Active brand with social accounts connected.
- URL, plain text, or an existing blog/post ID.

## How to use

> "Repurpose https://example.com/my-post into a LinkedIn and X thread. Schedule LinkedIn for tomorrow 9am UTC."

The agent:

1. **`check_social_accounts`** — confirm platforms.
2. **`repurpose_content`** with `sourceType` = url | text | social_post | blog, `targetPlatforms`, and optional `voiceProfileIds` (per-platform map supported: `["linkedin:voice1", "x:voice2"]`).
3. **`create_post`** with the generated content for each platform.
4. **`approve_post`** with a future `scheduledAt` (ISO 8601 UTC). Free-tier choke point.
5. **`get_calendar`** to confirm.

## Notes

- Do NOT fetch the URL yourself — PostKing crawls it internally for structured extraction.
- Use `includeLink: true` to keep the source URL in the generated post.
- Voice profiles: list with `list_voices`, apply by ID.

## Errors to expect

- `FREE_CAP_REACHED` on approve.
- `NOT_FOUND` if the source URL is unreachable.
