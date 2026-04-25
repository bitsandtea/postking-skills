---
name: landing-page-builder
description: Generate a landing page, iterate with AI edits, optionally connect a custom domain, and publish it live.
icon: https://postking.app/icons/landing.svg
categories: [marketing, design]
free: true
version: 0.1.0
---

# Landing Page Builder

From prompt to live landing page, including optional custom-domain publishing.

## Prerequisites

- Active brand.
- Optional: an already-verified custom domain (see Domains).

## How to use

> "Build a landing page for my product 'Acme Writer'. Add a pricing side-page. Publish on acme.com."

Flow:

1. **`create_landing_page`** with `topic`, optional `slug`, `voiceProfileId`. Returns slug.
2. **`update_landing_page`** with `instructions` — e.g. "tighten hero, add testimonials, urgent CTA". Repeat until the page is right.
3. (Optional) **`generate_side_page`** — feature, pricing, legal, case-study pages.
4. (Optional) Custom domain:
   - `add_domain` → adds a TXT record to create at your registrar.
   - `verify_domain` once the TXT is live.
   - `connect_domain` with `target: lp:<slug>`.
5. **`publish_landing_page`** — free-tier choke point. Returns the live URL.

## Expected output

- A published landing page with optional side-pages.
- Custom-domain URL if configured.

## Errors to expect

- `FREE_CAP_REACHED` on publish.
- `VALIDATION` on domain format.
- `NOT_FOUND` if the slug does not exist.
