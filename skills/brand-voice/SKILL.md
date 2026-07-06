---
name: brand-voice
description: List and apply saved PostKing voice profiles when generating or rewriting content, and run the de-slop / humanize / AI-detection pass on any text.
version: 0.1.0
compatibility: "Works with any MCP-compatible client connected to postking-mcp (local stdio or hosted at https://mcp.postking.app); the pking CLI is an optional fast path if a shell is available."
metadata:
  icon: https://postking.app/icons/brand-voice.svg
  free: true
  categories:
    - marketing
    - writing
  hermes:
    tags:
      - voice
      - writing
      - humanize
      - quality
      - marketing

    category: marketing
---

# Brand Voice & Quality

Keep generated and rewritten content on-brand and human-sounding — list saved voice profiles, apply one to a generation or rewrite, and run a de-slop / AI-detection pass before it ships.

**Voice-profile creation is web-only.** Building a new voice profile (from a writing sample, imported social posts, or a deep voice for a key author) happens in the PostKing dashboard's voice-profiles page — there is no MCP tool to create, extract, or import a voice. This skill covers what comes after: listing an existing profile and applying it.

## Minimal tool subset

This skill needs only these 5 tools out of PostKing's 200+:

- `list_voices` — list saved voice profiles with their IDs.
- `rewrite_with_voice` — rewrite arbitrary text in a specific saved voice.
- `rewrite_text` — general rewrite, optionally applying a voice profile.
- `humanize_text` — de-slop pass: rewrites and swaps AI-tell words/phrases to reduce AI-detection signals.
- `check_ai_content` — score how AI-detectable a piece of text reads.

## Prerequisites

1. Authenticated with credits — see [`getting-started`](../getting-started/).
2. A brand already onboarded — see [`getting-started`](../getting-started/).

Voices get applied downstream in [`postking`](../postking/) (posts, blogs, landing pages, repurposing), [`seo`](../seo/) (article writing), and [`campaign-launch`](../campaign-launch/) (campaign drafts).

## How to Use

> "Rewrite this in our brand voice and make sure it doesn't read as AI-written."

1. **Build a voice (dashboard only, one-time).** If no voice profile exists yet, tell the user to build one in the PostKing dashboard's voice-profiles page (from a writing sample, imported social posts, or a deep author voice) — there is no MCP tool for this step.
2. **List voices.** `list_voices({})` — get the `profileId` for the voice to use.
3. **Apply at generation time (preferred).** Pass that voice id as the `voice`/`voiceProfileId` param directly on `generate_post`, `generate_blog_post`, `generate_landing_page`, `repurpose_content`, or `reddit_rewrite` — no separate rewrite pass needed. These tools live in the [`postking`](../postking/), [`seo`](../seo/), [`campaign-launch`](../campaign-launch/), and [`reddit`](../reddit/) skills.
4. **Apply to existing text.** `rewrite_with_voice({ profileId, text, platform })` — or `rewrite_text({ text, voice: profileId, platform })` for a lighter-touch pass.
5. **De-slop / humanize.** `humanize_text({ text, platform })` — strips AI-tell words and clichés and reduces AI-detection signals. Run this after voice rewriting, right before the text ships.
6. **Score it.** `check_ai_content({ text })` — returns an AI-detection score + analysis. If the score is poor, re-run `humanize_text` and check again.

## CLI fast path

`pking voice list` lists saved voices (equivalent to `list_voices`). Beyond that, the MCP path above is primary — do not assume additional `pking` flags exist for rewrite/humanize without checking `pking --help` first.

## Expected Output

Content rewritten in a chosen brand voice, then de-slopped for lower AI-detection signal, with a `check_ai_content` score confirming the result reads as human.

## Troubleshooting / Errors to Expect

- `list_voices` returns empty — no voice profile exists yet; build one in the PostKing dashboard (creation is web-only, not available via MCP).
- `INSUFFICIENT_CREDITS` — surface the `checkoutUrl` from the error envelope and stop, or hand off to [`getting-started`](../getting-started/)'s billing section (`billing_list_packs` → `billing_topup`).
- `RATE_LIMITED` — back off using the `retryAfter` value from the error envelope before retrying.
- `UNAUTHORIZED` — re-run the login flow described in [`getting-started`](../getting-started/).
