---
name: landing-page-builder
description: Generate a landing page, iterate with AI edits, optionally connect a custom domain, and publish it live.
version: 0.2.0
compatibility: "Requires Node 18+ and the postking-cli npm package. Auto-installs on first use if missing."
metadata:
  icon: https://postking.app/icons/landing.svg
  free: true
  categories:
    - marketing
    - design
  hermes:
    tags:
      - landing-page
      - design
      - publishing
      - marketing

    category: marketing
---

# Landing Page Builder

From prompt to live landing page, including optional custom-domain publishing.

## Setup

**Setup.** This skill requires the `pking` CLI. If `command -v pking` fails, run `npm i -g postking-cli`. To authorize, run `pking login` and follow the URL it prints. You only need to do this once per machine.

## Prerequisites

- Active brand.
- Optional: a verified custom domain (run `pking domains list` to check).

## How to Use

> "Build a landing page for my product 'Acme Writer'. Add a pricing side-page. Publish on acme.com."

Flow:

1. **`pking lp generate --topic "Acme Writer" [--slug acme-writer] [--voice <voiceProfileId>]`**
   Async; returns a slug + operationId. The CLI polls automatically.
2. **`pking lp view <slug>`** — preview the generated content.
3. **`pking lp edit <slug> --instructions "Tighten the hero, add testimonials, urgent CTA."`** — AI edit pass. Repeat until the page is right.
   For section-level edits on side-pages: `pking lp side section <slug> <sideKey> --id <sectionId> --instructions "..."`
4. (Optional) Generate side-pages:
   `pking lp side generate <slug> --type pricing --wait`
   Supported types include `pricing`, `features`, `legal`, `case-study`.
5. (Optional) Custom domain:
   - `pking domains add <domain>` — adds a TXT record; follow the printed instructions at your registrar.
   - `pking domains verify <domain>` — once the TXT is live.
   - `pking domains connect <domainId> --target lp:<slug>`.
6. **`pking lp publish <slug>`** — free-tier choke point. Prints the live URL; surface the `webUrl` from the response to the user.

## Expected Output

- A published landing page with optional side-pages.
- Custom-domain URL if configured.

## Errors to Expect

- `FREE_CAP_REACHED` on publish — surface the `checkoutUrl` from the error envelope and stop.
- `VALIDATION` on domain format.
- `NOT_FOUND` if the slug does not exist — run `pking lp list` to confirm the slug.
