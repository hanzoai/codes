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

**1. Cloudflare prepends to robots.txt. It does not replace it.**

Measured on the live host after the first deploy: the served `/robots.txt` is our
file with a `# BEGIN Cloudflare Managed content` block glued on **above** it. So
the same document says both things —

```
# BEGIN Cloudflare Managed content
User-agent: *
Content-Signal: search=yes,ai-train=no,use=reference
...
User-agent: ClaudeBot
Disallow: /
# END Cloudflare Managed Content

  (our file starts here)
User-agent: *
Content-Signal: search=yes, ai-input=yes, ai-train=yes
...
User-agent: ClaudeBot
Allow: /
```

Which of those a crawler honours is then a question about that crawler's parser,
not about our intent. Most implementations merge same-agent groups and let the
least restrictive rule win at equal path length, so `Allow: /` probably wins — but
"probably" is not a position an AI company should take about whether AI may read
its documentation, and the `ai-train=no` signal sits above ours regardless.

No change in this repo can remove that block. The zone setting can: Cloudflare
dashboard → `hanzo.codes` → **AI Scrapers and Crawlers** → turn off the managed
`robots.txt`. The deploy workflow's last step fetches the live file, fails, and
names that fix, so the state cannot go quietly wrong again.

`hanzo cloudflare zones list` answers `503 cloudflare is not connected for this
org`, so this needs somebody with dashboard access.

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
