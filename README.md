# hanzo.codes

The coding surface of the Hanzo cloud: what `hanzo code` is, what it can reach,
and how to install it.

Static files, no build. `public/` **is** the site — what is committed is what a
stranger is served. Published to the Hanzo Sites plane by
[`.hanzo/workflows/deploy.yml`](.hanzo/workflows/deploy.yml) on every push to
`main`. Cloudflare is DNS and CDN only.

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

Don't do it by hand. Push to `main` **on `git.hanzo.ai`** — that is what runs CI;
GitHub is a mirror the forge writes to.

`bin/gates` runs first, in the job that publishes, so nothing publishes ungated.
Then the shared `site` action uploads `public/` to the plane. Then the workflow
re-fetches the live host and fails unless the bytes being served are the bytes it
just published, that both hostnames answer for every file, and that a crawler
reading `/robots.txt` is actually welcomed.

The route, the response headers and the certificate are not in this repo — they
are `universe` (`charts/app/values/hanzo/static-sites.yaml` and
`infra/k8s/ingress/wildcard-certs.yaml`). A `_headers` file here would be read by
nothing. See [LLM.md](LLM.md).

## Related

- [hanzoskills.com](https://hanzoskills.com) — the deep corpus, 537 documents in markdown
- [hanzo.sh](https://hanzo.sh) — the installer
- [github.com/hanzoai/cli](https://github.com/hanzoai/cli) — the binary this page is about
