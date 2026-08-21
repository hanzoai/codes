# hanzo.codes

The coding surface of the Hanzo cloud: what `hanzo code` is, what it can reach,
and how to install it.

Static files, no build. `public/` **is** the site — what is committed is what a
stranger is served. Published to Cloudflare as the assets-only Worker
`hanzo-codes` by [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml)
on every push to `main`.

```
public/
  index.html    the page
  index.md      the same page as markdown, for machines
  llms.txt      the index an agent reads first
  robots.txt    every crawler is welcome, and it says so explicitly
  sitemap.xml
  favicon.svg
  install       hands straight to hanzo.sh; the installer has one home
```

## Design

Monochrome, dark, mono-first — the same instrument panel as
[hanzoskills.com](https://hanzoskills.com), because they are two faces of one
product. The palette is `@hanzo/tokens` (a zero-saturation zinc ramp) as
`@hanzo/brand` paints it for dark: `#000` ground, `#f4f4f2` ink, hairline
`#232323` rules. The mark is `@hanzo/logo` — the animated one in the hero, which
folds in on load, turns once on hover, and holds still for anyone whose OS asked
it to.

There is no light variant on purpose. The dot field and the hairlines only read
on black.

## Content

Every claim on the page is checked against source, not remembered:

| Claim | Checked against |
|---|---|
| `hanzo code`, `hanzo dev`, `hanzo desktop`, `--no-mcp`, `--project-mcp`, `--no-route` | `hanzoai/cli` `src/main.rs`, `src/commands/code/` |
| The capability names, addresses and descriptions | `hanzoai/cloud` `manifest/apps.go` and `public.yaml` |
| `POST /v1/mcp` is the one door | `hanzoai/cloud` `manifest/door/door.go`, `fleet/mcp.go` |
| What a push actually triggers | `hanzoai/cloud` `apps/platform/hook.go`, `apps/platform/push.go` |
| Install lines | `hanzoai/hanzo.sh`, `hanzoai/cli` `npm/package.json` |

Two things the old page claimed that are not true, and are gone:
`brew install hanzoai/tap/hanzo` (that formula does not exist — the tap ships
`hanzo-dev`), and `curl hanzo.codes/install | sh` pointing at a 404. The install
path now exists and delegates to `hanzo.sh`.

## Deploying

Don't do it by hand. Push to `main`.

The workflow publishes, purges the edge, then re-fetches the live host and fails
unless the bytes being served are the bytes it just published — and separately
asserts that the live `robots.txt` is ours. Cloudflare can inject a managed
`robots.txt` at the zone level that disallows ClaudeBot, GPTBot, CCBot and the
rest; this domain served exactly that before this repo existed. A file in git
does not beat a zone setting, so the job asks the live host every time and names
the dashboard toggle when the answer is wrong.

Break-glass: `npx wrangler@3 deploy`. Afterwards `main` and the live host are two
different states again, so re-run the workflow.

## Related

- [hanzoskills.com](https://hanzoskills.com) — the deep corpus, 537 documents in markdown
- [hanzo.sh](https://hanzo.sh) — the installer
- [github.com/hanzoai/cli](https://github.com/hanzoai/cli) — the binary this page is about
