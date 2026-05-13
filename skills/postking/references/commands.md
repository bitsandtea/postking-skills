# `pking` command reference

Full command tree for `postking-cli` v1.0.3+. Run `pking <group> --help` for canonical flag documentation.

Conventions:
- Items in `<angle>` are required positional arguments.
- Items in `[brackets]` are optional.
- Flags ending in `...` accept multiple values.

### `webUrl` on responses

Most artifact-producing commands print a `View in browser: <url>` line on
success. The underlying API returns a `webUrl` field — the dashboard page
where the user can review the artifact (post detail page, blog edit
page, landing-page review page). Surface this URL to the user verbatim
whenever you confirm an action like "post created", "post scheduled",
"post published", "blog generated", "landing page updated", or "visual
picked". The user is assumed to be already logged in to the dashboard in
their browser.

Commands that include `webUrl`: `pking posts create / approve /
reschedule`, `pking visuals pick`, `pking blogs generate / publish`,
`pking lp generate / update / publish`.

---

## Auth

```
pking login                                Device-code OAuth flow (blocks; polls server).
pking login-start                          Print URL+code, save device_code, exit.
pking login-finish                         Finish a login-start session (exits 2 if pending).
pking logout                               Clear local credentials.
pking register --email <e> --password <p> [--client-name <n>]
                                           Create a new PostKing account.
pking login-password --email <e> --password <p> [--client-name <n>]
                                           Password login (legacy).
pking me                                   Show authenticated user, scope, credits, brands.
pking user credits                         Show plan + remaining credits.
```

## Brand

```
pking brand list                           List brands; active is starred.
pking brand info                           Show active brand details.
pking brand create <name> [--description <d>] [--website <url>] [--tone <t>] [--audience <a>]
                                           Create a brand manually (no crawl).
pking brand delete <brandId> --destructive [--confirm <brandName>] [--json]
                                           Delete a brand workspace (owner only; soft-delete).
                                           --destructive is required. After it, you must
                                           type the exact brand name when prompted to proceed.
                                           In non-interactive mode (--json), pass
                                           --confirm <brandName> to skip the prompt; mismatched
                                           or missing --confirm aborts with exit 1.
pking brand set <brandId>                  Switch active brand.
pking onboard <websiteUrl> [--name <n>]    Top-level. Crawl site, analyze audience,
                                           generate 10 themes. Async.
pking brand generate-themes [--count <n>] [--instructions <t>] [--input <path_or_text>]
                                           Regenerate themes for active brand.

pking brand themes list                    List content themes.
pking brand themes edit <themeId> [--title <t>] [--content <t>]
pking brand themes delete <themeId>        Delete a theme.
```

## Posts

```
pking posts list [--status <s>] [--platform <p>] [--limit <n>]
pking posts view <postId>
pking posts generate --platform <p> [--theme <t>] [--variations <n>] [--voice <id>]
pking posts generate-bulk --platform <p> [--voice <id>] [--frequency <f>] [--posts-per-day <n>] [--times <csv>] [--days <n>]
                                           Generate + schedule across a date range. frequency:
                                           daily|every_other|every_third|weekdays.
pking posts create --platform <p> --content "<text>" [--schedule <iso>]
pking posts approve <postId> [--variation <n>] [--schedule <iso>] [--timezone <tz>]
pking posts schedule <postId> --date <iso> [--variation <n>] [--timezone <tz>]
pking posts calendar [--days <n>]          Upcoming scheduled posts.
pking posts cancel <postId>
pking posts reschedule <postId> --date <iso>
pking posts delete <postId>
```

## Repurpose

```
pking repurpose --source-type <url|text|blog|social_post> --target-type <social|blog|text>
                [--source-url <url> | --source-content <text> | --source-blog <id> | --source-post <id>]
                [--target-platforms <csv>] [--variations <n>] [--angle <text>]
                [--specific-goal <text>] [--text-length <short|medium|long|custom:<words>>]
                [--voice <id_or_x:id,linkedin:id>] [--theme-id <id>] [--include-link]
                                           Turn a URL, text, blog, or post into
                                           platform-tailored content. No positional arg.
```

## Voice

```
pking voice list [--platform <p>] [--filter <shallow|deep>]
                                           List public voice profiles. Returns
                                           id, display name, supported
                                           platforms, and a description
                                           (social proof + tone). Use
                                           --platform (x, linkedin, threads,
                                           blog, lp) to narrow to voices
                                           trained on that medium.
pking voice rewrite --profile_id <id> --text <t> [--platform <p>]
                                           Rewrite text in a specific voice profile.
```

## Trends

PostKing crawls top-performing posts every 3 days for three niches —
`ai-saas`, `marketing`, `web3` — and deconstructs each one (hook, template,
pattern, virality reason). Use this as inspiration / writing prompts.
Currently only X is supported; more platforms coming.

```
pking trends list [--niche <ai-saas|marketing|web3>] [--platform <x>]
                  [--days <n>] [--limit <n>] [--sort <engagement|recent>]
                  [--json]
                                           Defaults: niche=ai-saas,
                                           platform=x, days=3 (the freshest
                                           crawl batch), limit=20,
                                           sort=engagement.
```

`--json` returns the full payload including the deconstruction object
(`hook`, `template`, `pattern`, `viralityReason`, `topic`) — feed that
into `pking posts generate` or `pking repurpose` as inspiration.

## Social

```
pking social check                         List connected social accounts.
pking social connect                       Generate generic OAuth magic link.
pking social connect-platform --platform <linkedin|x|instagram|threads|facebook>
                                           Magic link pre-targeted to a platform.
pking social disconnect <platform_or_id>
```

## Weekly schedule

```
pking weekly-schedule get
pking weekly-schedule set [--monday <spec>] [--tuesday <spec>] ... [--sunday <spec>]
                          [--timezone <tz>] [--lead-time <days>] [--voice <id>]
                          [--enable] [--disable]
                                           spec: "linkedin:1,x:1". Empty string clears a day.
pking weekly-schedule enable
pking weekly-schedule disable
pking weekly-schedule delete
pking weekly-schedule run-day --date <YYYY-MM-DD>
                                           Generate posts for a single date.
```

## Editor utilities

```
pking editor rewrite --text <t> [--voice <id>] [--platform <p>]
pking editor humanize --text <t> [--platform <p>]
pking editor detect --text <t>             AI-detection score.
```

## Blogs

```
pking blogs list [--status <s>]
pking blogs view <articleId>
pking blogs create --title "..." [--description "..."]
                                           Create a publication (NOT an article).
pking blogs generate --publication <id> --topic "..." [--voice <id>]
                                           [--length <short|medium|long>] [--keywords <csv>]
                                           [--wait] [--timeout <sec>]
                                           Async draft generation.
pking blogs status <articleId>             Poll a generation job.
pking blogs publish <articleId> [--connections <csv>]
pking blogs delete <articleId>

pking publications list
pking publications create --title "..." [--description "..."]

pking authors list
pking authors create --first-name <n> --last-name <n> [--email <e>]

pking categories list --publication <id>
pking categories create --publication <id> --name "..." --slug <s> [--description "..."]
```

## Jobs

```
pking jobs list [--status <s>] [--limit <n>]
                                           All async background jobs.
```

## Domains

```
pking domains list
pking domains add <domain>
pking domains verify <domain>
pking domains delete <id>
pking domains connect <id> --target <lp:<slug>|publication:<id>>
                                           Attach a verified domain.
```

## SEO pipeline

```
pking seo seeds <kw1> <kw2> ... [--brand <id>]
                                           Register seed keywords.
pking seo generate [--brand <id>] [--count <n>]
                                           Expand seeds → ~100 keywords. Async.
pking seo keywords [--brand <id>] [--limit <n>]
                                           List generated keywords.
pking seo categorize [--brand <id>]        Tag intent (informational, commercial,
                                           navigational, transactional).
pking seo cluster [--brand <id>]           Group keywords into topic pillars.
pking seo clusters list [--brand <id>]     Show clusters.
pking seo roadmap [--cluster <id>] [--items <n>] [--limit <n>] [--brand <id>]
                                           List items, or generate ~N topics
                                           from a cluster.
pking seo roadmap view <id> [--brand <id>]
pking seo roadmap edit <id> [--title <t>] [--status <s>] [--priority <n>]
pking seo roadmap delete <id> --destructive
pking seo write --roadmap-id <id> [--count <n>] [--brand <id>]
                                           Draft article(s) from a roadmap item.
pking seo gap [--brand <id>]               Gap analysis.
pking seo competitor --domain <domain> [--brand <id>]
                                           Competitor diff.
pking seo publish --article-id <id> [--publication <id>] [--schedule <iso>] [--brand <id>]
pking seo stats [--brand <id>]             Roadmap progress.
```

## Landing pages

```
pking lp list
pking lp view <slug>
pking lp generate --topic "..." [--slug <s>] [--voice <id>]
                                           Async; returns slug + operationId.
pking lp edit <slug> [--title <t>] [--instructions <text>]
                                           AI edit pass or manual title.
pking lp publish <slug>
pking lp delete <slug> --destructive       --destructive is required to confirm.
pking lp regenerate <slug> [--voice <id>] [--instructions <t>] [--section <id>]
pking lp draft view <slug>                 Preview pending changes.
pking lp versions list <slug>
pking lp versions view <slug> <versionId>

pking lp vibe <slug> --instructions "..." [--scope <full|section>] [--section-id <id>] [--wait]
                                           Vibe-edit. Slug is positional.
pking lp vibe status <slug> <operationId>  Poll a vibe edit.

pking lp set <slug> [--title <t>] [--file <path>] [--metadata-file <path>]
                                           Manually set content/metadata (no AI).
```

### Side pages

```
pking lp side list <slug>
pking lp side generate <slug> --type <type> [--count <n>] [--wait]
                                           Generate side-pages. Async.
pking lp side view <slug> <sideKey>
pking lp side edit <slug> <sideKey> [--instructions <text>]
pking lp side delete <slug> <sideKey> --destructive
pking lp side section <slug> <sideKey> --id <sectionId>
                                           [--content <text> | --file <path> | --instructions <text>]
pking lp side state <slug> <sideKey> [--publish | --unpublish]
pking lp side status <slug> <operationId>  Poll an async side-page op.
```

## Visuals — asset library

All library commands operate on the active brand. `upload` requires `--file
<path>`; `import-url` and `search-stock` take their target as a positional
arg (NOT a `--url` / `--query` flag).

```
pking visuals list [--type <image|video|document|link|lottie>] [--tags <csv>] [--search <q>] [--limit <n>]
pking visuals view <assetId>
pking visuals upload --file <path> [--name <n>] [--description <d>] [--tags <csv>]
pking visuals import-url <url> [--name <n>] [--tags <csv>]
pking visuals import-csv <file>            Bulk import (≤50 URLs; one per line or JSON array).
pking visuals tag <assetId> [--add <csv>] [--remove <csv>]
pking visuals delete <assetId>
pking visuals tags                         List all tags in use.
pking visuals suggest --context "<text>" [--limit <n>]
                                           AI-pick library assets for a caption.
pking visuals search-stock <query> [--platform <p>]
                                           Stock photo/video search (Unsplash + Pexels).
```

## Visuals — on a post

The post-scoped commands live under the SAME `visuals` namespace (NOT
`visuals-post`). `options` already returns library matches + card templates +
quote templates + stock results in one call — you do NOT need to upload an
asset first to get a carousel.

```
pking visuals options <postId> [--platform <p>] [--category <smart|card|quote|photo|video>]
                                           Numbered list of every candidate
                                           (library, cards, quotes, stock).
                                           Marks the recommended pick with ★.
                                           Caches index→pickArgs at
                                           ~/.postking/cache/visuals-<postId>-<platform>.json
                                           so the next `pick --pick <N>` works.
pking visuals regenerate <postId> [--load-external] [--platform <p>]
pking visuals pick <postId> --platform <p> --pick <N>
                                           Recommended path. N is the index
                                           printed by the most recent `options`
                                           call (cached for 24h).
pking visuals pick <postId> --platform <p> [--style <s> --variant <n> | --asset <id> | --slot <key>]
                                           Power-user override — explicit flags
                                           beat --pick when both are given.
pking visuals clear <postId> --platform <p>
pking visuals cards list <postId>
pking visuals cards edit <postId> --card <n> [--title <t>] [--body <t>] [--number <t>] [--rerender]
pking visuals cards set  <postId> --file <cards.json> [--rerender]
pking visuals carousel <postId> [--style <s>] [--variant <n>] [--title <t>]
                                           Render a multi-slide PDF from the post's cards.
```

### Recipe — "pick a visual for this post"

`options` is the discovery surface. It prints a `► Recommended` block, then
numbered sections (Library matches, Cards, Quotes, Stock photos, Stock
videos) with each row showing the rendered card text and an OSC-8
hyperlinked preview URL.

For card and quote templates, each row also prints the **resolved render
params** — the colors, avatar, fonts, and quote text that would actually
be drawn. Read these back to the user when they ask "what would this look
like" — you don't need to render a preview; the params ARE the spec.

The footer of every `options` call includes:

- A `Load more stock photos/videos: pking visuals regenerate <postId> --load-external` line — that's the canonical "show me more" path (no separate `--more` flag).
- A `To change brand colors or default avatar: <settingsUrl>` line — colors and avatar are NOT overridable per-pick. If the user wants different colors or a different avatar, they have to update brand settings in the dashboard at the URL printed there. Surface it verbatim.

```
pking visuals options <postId> --platform linkedin
# → user sees [1]…[N] with resolved colors/avatar/font under each template,
#   plus footer hints for "load more" and "change brand colors"
pking visuals pick <postId> --platform linkedin --pick 3
```

The `pick` call now renders quote/card template images **synchronously**
(typically 1–3 s) and writes the real PNG URL into `post.assets[0].url`,
so the dashboard shows it immediately. The response includes a `webUrl`
field — the dashboard URL where the user can review the post. Surface it
to the user when you confirm the pick.

The agent should NOT try to assemble `--style/--variant/--asset/--slot`
itself — that's the power-user path. `--pick <N>` is the canonical flow
because the CLI already cached every option's `pickArgs` from the
`options` call.

### Recipe — "make me a carousel" (no library asset required)

```
# 1. Confirm cards exist (auto-generated from post copy on creation).
pking visuals cards list <postId>

# 2. (Optional) tweak any card — title / body / stat number.
pking visuals cards edit <postId> --card 2 --body "New punchline" --rerender

# 3. Render the carousel PDF. Style is optional; defaults look fine.
pking visuals carousel <postId> --style glass-morphism --variant 2
```

If `cards list` returns an empty array, the post was created without
carousel-friendly content — write cards manually with
`pking visuals cards set <postId> --file cards.json` (a JSON array of
`{ number?, title?, body? }` objects), or regenerate the post with a
carousel-friendly theme via `pking posts generate`.

## API keys

```
pking keys list
pking keys create [--name "..."] [--scope <read|write>]
                                           Returns plaintext key once.
pking keys revoke <keyId>
```

---

## Error envelope

PostKing returns errors as JSON with consistent shape:

```json
{
  "error": "RATE_LIMITED",
  "message": "Too many requests",
  "retryAfter": 30,
  "checkoutUrl": "https://try.postking.app/billing?..."
}
```

Common codes:

| Code | Meaning | Action |
|---|---|---|
| `UNAUTHORIZED` | Invalid / expired token | `pking login` |
| `RATE_LIMITED` | Throttled | Wait `retryAfter` seconds |
| `INSUFFICIENT_CREDITS` | Out of plan credits | Surface `checkoutUrl`; stop |
| `FREE_CAP_REACHED` | Free-tier publish cap hit | Surface `checkoutUrl`; stop |
| `VALIDATION_ERROR` | Bad input | Show `message`, ask user to correct |
| `NOT_FOUND` | Bad id | Re-list and ask user to pick |

The CLI surfaces these as exit code != 0 with the JSON in stderr.
