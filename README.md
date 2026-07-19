# PostKing Skills

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![agentskills.io](https://img.shields.io/badge/agentskills.io-compatible-blue)](https://agentskills.io)
[![npm: postking-cli](https://img.shields.io/npm/v/postking-cli?label=postking-cli)](https://www.npmjs.com/package/postking-cli)

Open-source skills for [PostKing](https://postking.app) — the AI content platform for social posts, blogs, SEO / GEO, and landing pages.

Drop these into any agent that speaks the `SKILL.md` standard — **Hermes Agent**, **Claude Code / Claude.ai**, **OpenAI Codex**, **Cursor** — and your agent gains the full PostKing surface: generate posts, plan content weeks, run SEO / GEO research, ship blog articles, build landing pages, and more.

---

## Quick start

### Hermes Agent

```bash
hermes skills install bitsandtea/postking-skills/postking
hermes chat
> Draft 5 LinkedIn posts about our new pricing.
```

Install any other skill in this repo the same way, swapping the slug: `hermes skills install bitsandtea/postking-skills/<slug>` (e.g. `postking-seo`, `postking-reddit`).

### Claude Code plugin marketplace

```bash
claude plugin marketplace add bitsandtea/postking-skills
claude plugin install postking
```

The repo ships a `.claude-plugin/marketplace.json` so Claude Code can discover and install any of the 10 skills by slug directly from the marketplace.

### Claude Code / Claude.ai (manual copy)

```bash
mkdir -p ~/.claude/skills
cp -r skills/postking ~/.claude/skills/
```

Then in Claude: *"Use the postking skill to draft 3 posts about ..."*

### OpenAI Codex / agentskills.io clients

```bash
cp -r skills/postking ~/.agentskills/
```

All skills here are pure `SKILL.md` + Markdown references — they install with whatever `cp` or skill-installer your client provides. Swap `postking` for any other slug from the table below.

### Well-known discovery endpoint

Agents that support skill discovery via the [agentskills.io](https://agentskills.io) well-known convention can list and fetch every skill in this repo from `https://try.postking.app/.well-known/skills/index.json` (backed by `catalog.json`), without cloning the repo.

**Recommended first install:** `postking` — it's the thin router that picks/confirms the active brand and hands off to whichever specialist skill below owns the task, so most agents only need to install it plus whichever specialist skills the user's workflows touch.

---

## What's in this repo

Ten skills, MCP-first. The recommended entry point is **`postking`**; the other nine are specialist skills it routes to for first-run setup, social, blog, landing pages, SEO / GEO, campaigns, competitor intelligence, Reddit growth, and brand voice / quality.

| Skill | What it does | Backend |
|---|---|---|
| **[`postking`](skills/postking/)** | Thin router — picks/confirms the active brand, then hands off to the right specialist skill below. **Install this first.** | CLI + MCP |
| [`postking-getting-started`](skills/postking-getting-started/) | First-run flow — connect, authenticate, check credits/top-up/subscribe, onboard a first brand, connect socials, ship a first post. | MCP (+ optional CLI) |
| [`postking-social`](skills/postking-social/) | Generate, approve, and schedule social posts (LinkedIn, X/Twitter, Instagram, Threads, Facebook); Smart Week cadence; repurposing; visuals; trends. | MCP (+ optional CLI) |
| [`postking-blog`](skills/postking-blog/) | Blog publications, AI article generation/iteration, publish to PostKing-hosted or external platforms (WordPress, Medium, Substack). | MCP (+ optional CLI) |
| [`postking-landing-pages`](skills/postking-landing-pages/) | Landing-page generation, editing/vibe-editing, side pages (comparison/text/landing sub-pages), custom domains, publishing. | MCP (+ optional CLI) |
| [`postking-seo`](skills/postking-seo/) | Full seed-to-publish SEO / GEO pipeline: keyword research, clustering, roadmap, drafting, gap analysis, publish. | MCP (+ optional CLI) |
| [`postking-storylines`](skills/postking-storylines/) | Launch a full marketing campaign (Storylines) end-to-end. | MCP (+ optional CLI) |
| [`postking-competitor-intel`](skills/postking-competitor-intel/) | Discover, register, and analyze a brand's competitors (probe → classify → add → analyze → compare). | MCP |
| [`postking-reddit`](skills/postking-reddit/) | Build a fit-scored subreddit pool, match content to communities with posting angles, and natively rewrite posts for Reddit. | MCP |
| [`postking-brand-voice`](skills/postking-brand-voice/) | List and apply saved voice profiles, and run the de-slop / AI-detection pass on content. | MCP (+ optional CLI) |

`postking` drives both the `pking` CLI and MCP tool calls directly. The other nine are MCP-driven, with an optional `pking` CLI fast path where one exists — connect `postking-mcp` (local stdio or hosted at `mcp.postking.app`) to use them.

---

## Setup

### Option 1 — `postking` skill (recommended)

The `postking` skill uses the [`postking-cli`](https://www.npmjs.com/package/postking-cli) under the hood. First-time setup runs once per machine; the agent handles it for you when you install the skill.

```bash
npm install -g postking-cli
pking login        # device-code OAuth — paste the code from your browser
pking brand list   # confirm everything is wired up
```

Then in your agent:

> *"Use the `postking` skill to repurpose this URL into LinkedIn and X posts."*

### Option 2 — specialist skills (MCP)

The nine specialist skills (`postking-getting-started`, `postking-social`, `postking-blog`, `postking-landing-pages`, `postking-seo`, `postking-storylines`, `postking-competitor-intel`, `postking-reddit`, `postking-brand-voice`) talk to PostKing through the [`postking-mcp`](https://www.npmjs.com/package/postking-mcp) server.

**Local stdio:**

```bash
npm install -g postking-mcp
# Configure your client to launch postking-mcp as an MCP server.
```

**Hosted remote MCP:**

Point your client at `https://mcp.postking.app`. See [postking-mcp on npm](https://www.npmjs.com/package/postking-mcp) for client-specific configs (Claude Desktop, Cursor, etc.).

---

## Repository layout

```
postking-skills/
├── .claude-plugin/
│   └── marketplace.json             # Claude Code plugin marketplace manifest
├── skills/
│   ├── postking/                    # Router skill (recommended entry point)
│   │   ├── SKILL.md
│   │   └── references/commands.md, install.md
│   ├── postking-getting-started/    # First-run: auth, billing, brand onboarding, socials, first post
│   ├── postking-social/             # Social posts, Smart Week, repurposing, visuals, trends
│   ├── postking-blog/               # Blog publications, article generation, publishing
│   ├── postking-landing-pages/      # Landing pages, side pages, custom domains
│   ├── postking-seo/                # SEO / GEO: seeds → clusters → briefs → published articles
│   ├── postking-storylines/         # Storylines-driven campaign launch
│   ├── postking-competitor-intel/   # Competitor discovery, registration & analysis
│   ├── postking-reddit/             # Subreddit pool, content matching, native rewrites
│   └── postking-brand-voice/        # Voice profile listing/application, de-slop / AI-detection
├── assets/icons/                    # Per-skill catalog icons (SVG)
├── catalog.json                     # Machine-readable index for marketplaces
└── README.md
```

Each `SKILL.md` follows the [agentskills.io specification](https://agentskills.io): YAML frontmatter with `name`, `description`, `license`, `compatibility`, and `metadata` (icon + category) — plus a Markdown body with `When to Use`, `Procedure`, `Pitfalls`, and `Verification` sections. Note: `version` is NOT a frontmatter field — per-skill versions live in `catalog.json`.

---

## How skills work

A skill is **plain text instructions for an LLM** — no binaries, no servers. When your agent loads a skill, it reads the `SKILL.md`, follows the procedure, and calls the underlying tool (the `pking` CLI or the PostKing MCP server) on your behalf.

Progressive disclosure keeps token usage low:

1. The agent sees only `name` + `description` at session start (~50 tokens).
2. When triggered, it reads the `SKILL.md` body (~150 lines).
3. For deep details, it loads `references/commands.md` on demand.

Read more about the format at [agentskills.io](https://agentskills.io) and [Hermes Skills docs](https://hermesagent.xyz/docs/user-guide/features/skills).

---

## Compatibility

| Client | Format support | How to install |
|---|---|---|
| [Hermes Agent](https://hermesagent.xyz) | Native | `hermes skills install bitsandtea/postking-skills/<slug>` |
| [Claude Code](https://claude.com/claude-code) | Native (plugin marketplace + agentskills.io) | `claude plugin marketplace add bitsandtea/postking-skills`, or copy to `~/.claude/skills/` |
| [Claude.ai](https://claude.ai) | Native | Upload via the `/skills` UI |
| [OpenAI Codex](https://github.com/openai/codex) | agentskills.io | `cp -r skills/<slug> ~/.claude/skills/` (or your client's skills dir) |
| [Cursor](https://cursor.sh) | Via MCP | Connect `mcp.postking.app` |
| Any agentskills.io client | Native | `cp -r skills/<slug> ~/.claude/skills/`, or fetch via the well-known endpoint |

---

## Contributing

Pull requests welcome. To add a new skill:

1. Create `skills/<your-skill-slug>/SKILL.md` following the [agentskills.io spec](https://agentskills.io).
2. Add an entry to `catalog.json`.
3. Update the table in this README.
4. Test it locally with at least one supported client before opening the PR.

Bug reports and feature requests: [open an issue](https://github.com/bitsandtea/postking-skills/issues).

---

## Related

- **[postking-cli](https://www.npmjs.com/package/postking-cli)** — the `pking` CLI that backs the `postking` skill.
- **[postking-mcp](https://www.npmjs.com/package/postking-mcp)** — the MCP server that backs the workflow skills.
- **[mcp.postking.app](https://mcp.postking.app)** — hosted remote MCP endpoint.
- **[try.postking.app](https://try.postking.app)** — sign up for PostKing.
- **[PostKing API docs](https://try.postking.app/docs/api)** — REST surface for direct integrations.

---

## License

[MIT](LICENSE) © PostKing.
