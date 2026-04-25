# `pking` command reference

Full command tree for `postking-cli` v1.0.3+. Run `pking <group> --help` for canonical flag documentation.

Conventions:
- Items in `<angle>` are required positional arguments.
- Items in `[brackets]` are optional.
- Flags ending in `...` accept multiple values.

---

## Auth

```
pking login                                Device-code OAuth flow.
pking logout                               Clear local credentials.
pking register --email <e> --password <p>  Create a new PostKing account.
pking login-password --email <e> --password <p>
                                           Password login (legacy).
pking user me                              Show authenticated user.
pking user credits                         Show plan + remaining credits.
```

## Brand

```
pking brand list                           List brands; active is starred.
pking brand info                           Show active brand details.
pking brand create <name>                  Create a brand manually.
pking brand set <brandId>                  Switch active brand.
pking brand onboard <websiteUrl> [--name <n>]
                                           Crawl site, analyze audience,
                                           generate 10 themes. Async.
pking brand generate-themes                Regenerate themes for active brand.

pking brand themes list                    List content themes.
pking brand themes edit <themeId>          Edit a theme.
pking brand themes delete <themeId>        Delete a theme.
```

## Posts

```
pking posts list [--platform <p>] [--status <s>]
pking posts view <postId>
pking posts generate --platform <p> [--theme <t>] [--variations <n>]
pking posts generate-bulk --count <n> [--platforms <p1,p2,...>]
pking posts create --platform <p> --content "<text>"
pking posts approve <postId> [--variation <n>] [--schedule <iso>]
pking posts schedule <postId> --at <iso>
pking posts calendar                       Upcoming scheduled posts.
pking posts cancel <postId>
pking posts reschedule <postId> --at <iso>
pking posts delete <postId>
```

## Repurpose

```
pking repurpose <url> [--platforms <p1,p2,...>] [--text <override>]
                                           Turn a URL or text block into
                                           platform-tailored posts.
```

## Voice

```
pking voice list
pking voice rewrite                        Rewrite a draft in brand voice.
```

## Social

```
pking social check                         List connected social accounts.
pking social connect [--platform <p>]      Generate OAuth deep-link.
pking social disconnect <platform_or_id>
pking social connect-platform              Multi-step OAuth helper.
```

## Weekly schedule

```
pking weekly-schedule get
pking weekly-schedule set                  Interactive cadence editor.
pking weekly-schedule enable
pking weekly-schedule disable
pking weekly-schedule delete
pking weekly-schedule run-day <YYYY-MM-DD>
                                           Generate all posts for a day.
```

## Editor utilities

```
pking editor rewrite                       Rewrite text in brand voice.
pking editor humanize                      De-AI-ify text.
pking editor detect                        AI-detection score.
```

## Blogs

```
pking blogs list [--status <s>]
pking blogs view <articleId>
pking blogs create --title "..." --content "..."
pking blogs generate --topic "..." [--keyword "..."]
                                           Async draft generation.
pking blogs status <articleId>             Poll a generation job.
pking blogs publish <articleId>
pking blogs delete <articleId>

pking publications list
pking publications create --name "..." [--domain <id>]

pking authors list
pking authors create --name "..." [--bio "..."]

pking categories list
pking categories create --name "..." [--slug <s>]
```

## Jobs

```
pking jobs list                            All async background jobs.
```

## Domains

```
pking domains list
pking domains add <domain>
pking domains verify <domain>
pking domains delete <id>
pking domains connect <id>                 Attach domain to a publication.
```

## SEO pipeline

```
pking seo seeds <kw1> <kw2> ...            Register seed keywords.
pking seo generate                         Expand seeds → ~100 keywords. Async.
pking seo keywords                         List generated keywords.
pking seo categorize                       Tag intent (informational,
                                           commercial, navigational,
                                           transactional).
pking seo cluster                          Group keywords into topic pillars.
pking seo clusters list                    Show clusters.
pking seo roadmap                          Generate ~20 article topics from
                                           a cluster.
pking seo view <id>                        View a roadmap item.
pking seo edit <id>
pking seo delete <id>
pking seo write [--roadmap <id>]           Draft an article. Async.
pking seo gap                              Gap analysis.
pking seo competitor                       Competitor diff.
pking seo publish [--article <id>]
pking seo stats                            Roadmap progress.
```

## Landing pages

```
pking lp list
pking lp view <slug>
pking lp generate --topic "..." [--keyword "..."]
                                           Async; returns slug + operationId.
pking lp edit <slug> [--instructions "..."]
                                           Vibe edit (async) or manual flags.
pking lp publish <slug>
pking lp delete <slug>
pking lp regenerate <slug>
pking lp draft view <slug>                 Preview pending changes.
pking lp versions list <slug>
pking lp versions view <slug> <versionId>

pking lp vibe                              Start a vibe-edit session.
pking lp vibe status <slug> <operationId>  Poll a vibe edit.

pking lp set <slug>                        Set page-level config.
```

### Side pages

```
pking lp side list <slug>
pking lp side generate <slug>              Generate a new side page. Async.
pking lp side view <slug> <sideKey>
pking lp side edit <slug> <sideKey>
pking lp side delete <slug> <sideKey>
pking lp side section <slug> <sideKey>     Edit a specific section.
pking lp side state <slug> <sideKey>       Get/set side-page state.
pking lp side status <slug> <operationId>  Poll an async side-page op.
```

## Visuals (asset library)

```
pking visuals list [--tag <t>]
pking visuals view <assetId>
pking visuals upload <path>                Upload local image/video.
pking visuals import-url <url>             Import remote asset.
pking visuals import-csv <file>            Bulk import from CSV.
pking visuals tag <assetId> --tags <t1,t2,...>
pking visuals delete <assetId>
pking visuals tags                         List all tags.
pking visuals suggest                      AI-suggest assets for a brief.
pking visuals search-stock <query>         Search stock library.
```

## Visuals on a post

```
pking visuals-post options <postId>        Suggested visuals for a post.
pking visuals-post regenerate <postId>
pking visuals-post pick <postId> --asset <id>
pking visuals-post clear <postId>
pking visuals-post cards                   Card-style visual generator.
pking visuals-post cards list <postId>
pking visuals-post cards edit <postId>
pking visuals-post cards set <postId>
pking visuals-post carousel <postId>       Multi-slide carousel generator.
```

## API keys

```
pking keys list
pking keys create --name "..."             Returns plaintext key once.
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
