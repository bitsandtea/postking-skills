---
name: blog-publish-pipeline
description: Generate a blog article, iterate with AI edits, and publish it to PostKing or to external platforms (WordPress, Medium, Substack).
icon: https://postking.app/icons/blog.svg
categories: [writing, marketing]
free: true
version: 0.1.0
---

# Blog Publish Pipeline

Prompt-to-published in a handful of tool calls.

## Prerequisites

- Active brand.
- Optional: voice profile (`list_voices`).
- Optional: external publishing connection (WordPress / Medium / Substack).

## How to use

> "Draft a 1500-word blog post about 'AI content calendars' in my brand voice and publish it."

Flow:

1. **`list_blogs`** — find or create a publication (`create_publication`).
2. **`generate_blog_post`** with `publicationId`, `topic`, `voiceProfileId`, `targetLength`, `primaryKeywords`.
3. **`get_blog_article`** — review first 3000 chars.
4. **`update_blog_article`** — iterate on title, content, SEO fields.
5. Publish:
   - **PostKing hosted**: `update_blog_article` with `status: "published"`.
   - **External**: `list_publishing_connections` → `publish_blog_article` with connectionIds.
6. **`get_blog_article`** again to confirm status.

## Expected output

- Published article URL on PostKing or on connected external platforms.
- SEO metadata populated (meta title, description, slug).

## Errors to expect

- `FREE_CAP_REACHED` on publish.
- `VALIDATION` when required fields (title, content) are missing.
