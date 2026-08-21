# LLM.md — hanzo.codes

## What this is

One static page and its machine-readable twins. No framework, no bundler, no
`package.json`. `public/` is the deployed artifact verbatim.

If you are tempted to add a build step, don't. The page is 20KB of hand-written
HTML with the CSS inline; a toolchain would add a way for the repo and the live
site to disagree, which is the only failure this repo has to prevent.

## Where the site comes from

There was no source repo. hanzo.codes was live on Cloudflare and its source was
in neither the `hanzoai` GitHub org, the `hanzo-inc` org, nor the forge at
git.hanzo.ai — checked by name (`codes`, `code`, `hanzo.codes`, `codes-site`), by
forge org listing, and by grepping the estate for the literal domain. `hanzoai/code`
is the VS Code fork, not this. `hanzoai/web` is archived. So this repo is new, and
the live page it replaces was rebuilt from what it served plus what the source
says.

## Two rules that are not obvious

**1. The live robots.txt is not necessarily the one in this repo.**

Cloudflare's "AI Scrapers and Crawlers" zone setting injects a managed
`robots.txt` that disallows ClaudeBot, GPTBot, CCBot, Google-Extended, Bytespider,
Amazonbot, Applebot-Extended and meta-externalagent, and sets
`Content-Signal: ai-train=no`. That is what this domain served. Shipping a file
does not switch it off — the toggle is in the dashboard, on the `hanzo.codes`
zone. The deploy workflow's last step re-fetches the live `robots.txt` and fails
with the fix named if the managed one is winning, so the state can never be
silently wrong again.

`hanzo cloudflare zones list` currently answers
`503 cloudflare is not connected for this org`, so the toggle needs somebody with
dashboard access.

**2. Claims are checked, not remembered.**

The page states facts about the CLI and the cloud. Before changing one, read the
source — `hanzoai/cli` `src/main.rs` for commands and flags, `hanzoai/cloud`
`manifest/apps.go` for capability addresses, `hanzoai/cloud` `public.yaml` for
their descriptions. Two claims on the previous page were false (`brew install
hanzoai/tap/hanzo`, and an `/install` path that answered 404) and both had been
live for a while, because nothing ever asked.

Specific traps found the last time:

- The capability is **`sandboxes`**, plural. The Go package and the prose are
  singular. `/v1/sandbox` is not an address.
- `deploy` does not own `/v1/deploy/*` wholesale — it declares 14 explicit
  prefixes.
- The toolset a `hanzo code` session attaches by default is a **local
  `hanzo-mcp` process**, not `api.hanzo.ai/v1/mcp`. Same tools, different thing.
  Do not write that a session is wired to the cloud door by default.
- `POST /v1/git-webhook` is a tombstone that answers 410. Do not cite it.
- "A push is the deploy" overstates. A push is delivered to a signed door which
  *starts a build* for every app tracking that repo and branch; a reconciler
  applies the image when the build succeeds. A push matching no app builds
  nothing, and that is the common case.

## Palette

Monochrome, from `@hanzo/tokens` (`hsl(0 0% n)` throughout) as `@hanzo/brand`
paints dark. `#000` ground, `#f4f4f2` ink, `#9c9c98` soft, `#6a6a66` faint,
`#232323` rules, `#0a0a0a` panels. Identical to hanzoskills.com, deliberately.

The predecessor page used `#faf8f4` cream and `#c2410c` orange. There is no
orange in the Hanzo palette. The deploy workflow rejects any hex outside the
monochrome set, so a stray accent colour fails the build rather than shipping.

## The mark

`@hanzo/logo` v1.0.20. Two copies are inlined rather than linked: the static
5-path mark in the nav, and the animated one in the hero
(`svg/hanzo-mark-animated.svg` — origami fold-in on load, one 3-D turn on hover,
squash on press, pure CSS, and inert under `prefers-reduced-motion`). Inlined
because the page must render with zero network requests beyond itself, and
because the CSP on a static Worker gives nothing for free.

If the mark changes upstream, re-copy from `/home/z/work/hanzo/logo/dist/` and
`/home/z/work/hanzo/logo/svg/`. Do not redraw the paths by hand.
