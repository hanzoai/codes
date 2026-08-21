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
`#232323` rules, `#0a0a0a` panels. `--faint` is `#7a7a75`, one step lighter than
hanzoskills.com's, which measures under 4.5:1 on both grounds.

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

`og.png` is `@hanzo/logo` `dist/og/og-image.png`, 1200x630, carried so shares
render the mark instead of nothing.

If the mark changes upstream, re-copy from `/home/z/work/hanzo/logo/dist/` and
`/home/z/work/hanzo/logo/svg/`. Do not redraw the paths by hand.

## The mark sheet

A hidden `<svg>` of `<symbol>`s at the top of `<body>` holds every mark the page
repeats — Hanzo, Claude, Codex, Cursor, OpenAI, plus a chevron and a document
glyph. The agent bar renders twice for its loop, so inlining the paths would
carry Claude's 2.5KB body four times; `<use href="#m-…">` carries it once.

Sources, all copied verbatim, none redrawn:

| Symbol | From |
|---|---|
| `m-hanzo` | `hanzo/logo/dist/logo-menubar.svg` (canonical 7-path, `currentColor`) |
| `m-claude`, `m-codex`, `m-cursor`, `m-openai` | `hanzo.ai/public/providers/<n>.svg` |

`fill="white"` on the provider files becomes `currentColor` so a chip inherits
its own colour. Geometry is untouched. Regenerate with the script that reads
those files rather than pasting paths by hand.

**OpenClaw has no mark anywhere in the estate**, so `m-openclaw` is an `OC`
monogram plate. `hanzo.ai/scripts/gen-marks.mjs` sets that policy: a monogram is
the honest fallback, a look-alike pictograph is not. If a real OpenClaw mark
lands, replace the plate.

## Controls

Radius and button treatment come from hanzo.ai, read off the live page rather
than guessed:

| | hanzo.ai | here |
|---|---|---|
| button radius | `9999px` (pill) | `--pill: 999px` |
| button height | 44px | 44px |
| horizontal padding | 28px | 1.75rem |
| fill / outline pair | white on black / `neutral-700` hairline | `--ink` / `--line-hi` |
| card radius | 24px | `--soft-r: 16px` (menu, `pre`, `.term`) |

The page keeps its own mono uppercase lettering; only the shape is borrowed.
Every button was `border-radius: 0` before this.

`.hero` must not carry `overflow:hidden` — it clips the setup menu where it
drops past the hero edge. The dot field is `inset:0` and never spills, so
nothing needs the clip.

The table scroller is the parent `.scroll`, not the `<table>`. `min-width` on
the element that also owns `overflow-x` makes that element wider than the page;
that shape overflowed 604px into a 390px viewport.

## The agent list

The bar and the setup menu name only clients we actually support. Verified, not
assumed:

- Skills targets, from `hanzo/skill` `src/cli.ts`: `~/.claude/skills` (Claude
  Code), `~/.agents/skills` (Codex, OpenClaw), `~/.cursor/skills` (Cursor),
  `~/.hanzo/bot/skills` (Hanzo Bot).
- Session backends, from `hanzo code --help`: `dev`, `claude`, `codex`. Those
  three and no others.
- The model API answers `/v1/chat/completions` and `/v1/messages`, so the OpenAI
  and Anthropic SDKs point at it unchanged.

**Hermes is not on the list.** The only Hermes in the estate is the React Native
JS engine. There is no Hermes agent to support, and a mark for one would be a
claim we cannot back.

Each menu row copies a prompt written for that tool — its own registration
command, its own skills directory, its own way to confirm. The syntax was read
from the installed binaries (`claude mcp add --help`, `codex mcp add --help`),
not from memory. `codex mcp add` takes `--bearer-token-env-var`; `claude mcp
add` takes `--header`. They are not interchangeable.

## The capability names, and why the plural is still here

HIP-0139 §2.4 "refuses a pair differing only in number and keeps the singular",
and it is quoted in `manifest/apps.go` itself where `bot`/`bots` were folded into
one singular row. The direction is settled: capability names go singular.

It has **not** landed for sandbox. Measured, not read:

```
gh api repos/hanzo-inc/cloud/contents/manifest/apps.go   -> {Name: "sandboxes", Prefixes: []string{"/v1/sandboxes"}}
curl -o /dev/null -w '%{http_code}' api.hanzo.ai/v1/sandbox    -> 404
curl -o /dev/null -w '%{http_code}' api.hanzo.ai/v1/sandboxes  -> 403
```

403 means the route exists and wants a credential. 404 means nothing is there. So
`/v1/sandbox` on this page would be a dead address, and there is no mechanical
plural alias in the router either — `grep -nE 'plural|singular|TrimSuffix' manifest/`
finds two comments and no code.

The page therefore prints `sandboxes`, and the deploy workflow watches that one
pair on every publish: it warns when `/v1/sandbox` starts answering, or when the
plural stops. When the rename lands, the deploy says so. Do not "fix" the copy
from a doc — re-run the probe.

Probing all nine stems was tried first and removed. A prefix-only app has no
handler at its own root, so `/v1/code`, `/v1/git`, `/v1/platform`, `/v1/deploy`
and `/v1/lsp` answer 404 while being entirely alive underneath — `deploy`
registers fourteen explicit sub-paths and no bare stem at all. Five false
warnings per deploy is worse than no check, because it teaches everyone to skip
the warnings that matter.

## Why CI is in .github/workflows and not .hanzo/workflows

The estate rule is that CI lives in `.hanzo/workflows` on the git.hanzo.ai runner
fleet. The rule exists because the default runner label
`hanzo-build-linux-amd64` is advertised only by forge runners, so a job asking
for it on github.com waits out a 24-hour timeout instead of failing.

Neither half of that applies here:

- This workflow asks for `ubuntu-latest`, not the forge label, so it cannot hang
  waiting for a runner that does not exist.
- This repository exists **only** on GitHub. There is no forge mirror, so a file
  under `.hanzo/workflows` would have no executor at all.
- Deploying a Cloudflare assets Worker is not a container build. It needs
  `wrangler deploy` and a Cloudflare token, and the only copy of that token is in
  GitHub org secrets — proven, because the publish step succeeds.

Measured: this workflow has published the live site repeatedly, 40-80s a run.

The cautionary tale is `hanzoai/hanzo.sh`, whose `.github/workflows/deploy.yml`
was deleted with the message "Actions are disabled on this repository, so these
files cannot run". `gh api repos/hanzoai/hanzo.sh/actions/permissions` answers
`{"enabled":true}`, and no `.hanzo/` directory exists on its `origin/main`. The
premise was wrong and it left that host with no deploy path, which is the exact
failure its own README warns about. Do not repeat it here without first checking
that something can actually run the replacement.
