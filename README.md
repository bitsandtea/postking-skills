# PostKing Skills

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![agentskills.io](https://img.shields.io/badge/agentskills.io-compatible-blue)](https://agentskills.io)
[![npm: postking-cli](https://img.shields.io/npm/v/postking-cli?label=postking-cli)](https://www.npmjs.com/package/postking-cli)

Open-source skills for [PostKing](https://postking.app) — the AI content platform for social posts, blogs, SEO, and landing pages.

Drop these into any agent that speaks the `SKILL.md` standard — **Hermes Agent**, **Claude Code / Claude.ai**, **OpenAI Codex**, **Cursor** — and your agent gains the full PostKing surface: generate posts, plan content weeks, run SEO research, ship blog articles, build landing pages, and more.

---

## Quick start

### Hermes Agent

```bash
hermes skills install bitsandtea/postking-skills/postking
hermes chat
> Draft 5 LinkedIn posts about our new pricing.
```

### Claude Code / Claude.ai

```bash
mkdir -p ~/.claude/skills
cp -r skills/postking ~/.claude/skills/
```

Then in Claude: *"Use the postking skill to draft 3 posts about ..."*

### OpenAI Codex / agentskills.io clients

```bash
cp -r skills/postking ~/.agentskills/
```

All skills here are pure `SKILL.md` + Markdown references — they install with whatever `cp` or skill-installer your client provides.

---

## What's in this repo

The recommended entry point is the **`postking`** skill. The rest are narrower workflow skills you can install individually.

| Skill | What it does | Backend |
|---|---|---|
| **[`postking`](skills/postking/)** | The complete PostKing surface — brands, posts, blogs, SEO, landing pages, scheduling, visuals. Drives ~140 commands. **Recommended.** | [`postking-cli`](https://www.npmjs.com/package/postking-cli) |
| [`onboarding`](skills/onboarding/) | Zero-to-first-post setup for new users — register, onboard a brand, connect socials, ship one post. | MCP |
| [`content-week-planner`](skills/content-week-planner/) | Plan + approve a full week of social posts across all connected platforms. | MCP |
| [`repurpose-url-to-social`](skills/repurpose-url-to-social/) | Turn any URL, blog post, or long text into platform-tailored, voice-matched social posts. | MCP |
| [`blog-publish-pipeline`](skills/blog-publish-pipeline/) | Generate → iterate → publish a blog article (PostKing or external platforms). | MCP |
| [`seo-end-to-end`](skills/seo-end-to-end/) | Full seed-to-publish SEO pipeline: keyword research, clustering, roadmap, drafting, gap analysis, publish. | MCP |
| [`landing-page-builder`](skills/landing-page-builder/) | Generate, iterate, and publish landing pages with custom domains. | MCP |

`postking` is CLI-driven — runs anywhere `node` is available, no API key paste required. The other six are MCP-driven and assume you've connected `postking-mcp` (local or hosted at `mcp.postking.app`).

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

### Option 2 — workflow skills (MCP)

The narrower workflow skills (`onboarding`, `seo-end-to-end`, etc.) talk to PostKing through the [`postking-mcp`](https://www.npmjs.com/package/postking-mcp) server.

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
├── skills/
│   ├── postking/                    # Broad CLI-based skill (recommended)
│   │   ├── SKILL.md
│   │   └── references/commands.md
│   ├── onboarding/                  # Workflow skills (MCP-based)
│   ├── content-week-planner/
│   ├── repurpose-url-to-social/
│   ├── blog-publish-pipeline/
│   ├── seo-end-to-end/
│   └── landing-page-builder/
├── catalog.json                     # Machine-readable index for marketplaces
└── README.md
```

Each `SKILL.md` follows the [agentskills.io specification](https://agentskills.io): YAML frontmatter (`name`, `description`, `version`, `metadata`) plus a Markdown body with `When to Use`, `Procedure`, `Pitfalls`, and `Verification` sections.

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
| [Hermes Agent](https://hermesagent.xyz) | Native | `hermes skills install bitsandtea/postking-skills/postking` |
| [Claude Code](https://claude.com/claude-code) | Native (agentskills.io) | Copy to `~/.claude/skills/` |
| [Claude.ai](https://claude.ai) | Native | Upload via the `/skills` UI |
| [OpenAI Codex](https://github.com/openai/codex) | agentskills.io | Copy into your skills directory |
| [Cursor](https://cursor.sh) | Via MCP | Connect `mcp.postking.app` |
| Any agentskills.io client | Native | Copy `skills/<name>/` |

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
