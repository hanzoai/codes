# Hanzo Codes

Describe the work. Hanzo does it in your repo, on your machine, against your tests.

One binary. Your files, your git, your credentials. The whole Hanzo toolset is
attached the moment a session starts, and every capability it reaches is an
ordinary API call — so the same work runs with a terminal or without one.

## Install

Two ways in. The script needs nothing you do not already have; the second exists
because you may already live there.

```sh
curl -fsSL https://hanzo.sh | sh       # no runtime required
npm install -g hanzo                   # if node is already there
```

Both land the same binary. It is written in Rust and it is one file — no daemon
to run, no container to pull, and nothing to keep current but the file itself.
The npm package is a downloader around that same binary, not a second
implementation of it.

## A session

A trailing task runs headless; leave it off and you get an interactive session
instead. The command name is optional — `hanzo` with a task is already a session.

```
$ hanzo code "the metered list gates a surface nobody bills"
  reading spend.go, manifest/apps.go, plugin/…
  apps/auto was deleted; /v1/auto routes to automations
  removed the entry, ran the guard test
  ok  github.com/hanzoai/cloud  45.4s
```

An agent that pattern-matches your prompt onto a plausible diff will hand you
something that compiles and is wrong. This one goes and looks — at the routing
table, at the deleted package, at what the test actually asserts — and when the
evidence contradicts the request, it says so instead of shipping.

- **Your machine.** Your files, your git, your credentials. Nothing is uploaded
  to be indexed, and nothing is written that you did not see.
- **Your tests are the judge.** Not a confidence score. It runs them, shows the
  output, and a red suite is reported red.
- **It stops.** On anything hard to reverse — a force push, a production deploy,
  a rotated secret — it hands the decision back.

## Bring your own agent

The session is the product; the model inside it is your choice. Our `dev` agent
runs by default, and naming another is one word. Whichever you pick gets the same
toolset, the same identity and the same meter — so the bill, the audit trail and
the tools do not change when the model does.

```sh
hanzo code                             # dev, the default
hanzo code claude
hanzo code codex
hanzo dev                              # shorthand for `hanzo code dev`
hanzo desktop                          # the same session, aimed at the desktop
```

Model calls route through `api.hanzo.ai`, which is how a run can be metered and
streamed at all. `--no-route` sends them to the backend's own account instead,
and you get an unmetered session.

## The tools are already attached

A session starts with the Hanzo toolset connected. There is no extension to
install, no config file to edit and nothing to restart. If you want a bare model
with none of it, `--no-mcp` starts one.

```sh
hanzo code                             # toolset attached
hanzo code --no-mcp                    # the model on its own
hanzo code --project-mcp               # also the repo's own .mcp.json
```

That last one is off by default on purpose. A repository is untrusted input: any
MCP server it declares would run with your session's model key, so loading one is
a decision you make about a repo you trust, not something you inherit by cloning.

The toolset your session attaches runs beside you, on your machine. The cloud has
its own door for everything else — one address, an ordinary HTTP endpoint. Any
MCP client points at it — an editor, a CI job, another agent — and gets the same
tools under the same identity and the same cap.

```
POST https://api.hanzo.ai/v1/mcp
```

There is no second entry path. `/mcp` exists only to redirect to it.

## What it can reach

These are capabilities of the cloud, not features of an editor. Each answers on
its own path under `api.hanzo.ai`, each is a tool inside a session, and each is
plain HTTP outside of one. The ones a coding session leans on:

| Capability | Answers at | What it is |
|---|---|---|
| code | `/v1/code` | Search and symbols across your repos, for you and your agents. |
| lsp | `/v1/lsp` | Real language servers behind that — definitions, references, diagnostics — on an immutable checkout per org, repo and commit, jailed with no network. |
| exec | `/v1/exec` | The code interpreter: run a snippet in a sandbox, and move files in and out of the session that sandbox *is*. |
| sandboxes | `/v1/sandboxes` | The one compute primitive — a gVisor pod that runs somebody else's code. Every lifetime is the same object. |
| functions | `/v1/functions` | Your serverless code: publish it, call it over HTTP, watch every run and what it cost. |
| git | `/v1/git` | Git hosting for your org: create repos, clone, push, and see what they cost. |
| projects | `/v1/projects` | Where your sites live: create one, deploy a build, roll back to any release. |
| platform | `/v1/platform` | Hanzo PaaS: deploy containers to your own tenant namespace — builds, releases, environments, logs, custom domains. |
| deploy | `/v1/deploy` | Hanzo CD: see what each app is running, sync it, and roll back a bad release. |

That is nine of them. The CLI is generated from the same specification the API
serves, so all **182 products** and **2,379 operations** are already subcommands
of the binary you just installed — `hanzo <product> <verb>`, with no plugin to
add and no release to wait for.

## Push, and it ships

Hanzo hosts your git. A push is delivered to a signed door that starts a build for
every app tracking that repository and branch; when the build succeeds its image
is written to the cluster and rolled out. You wire nothing, and the record of what
is running has one home.

```sh
git remote add hanzo https://git.hanzo.ai/<org>/<repo>
git push hanzo main
```

A push that maps to no application builds nothing, and says so. That is the common
case and it is not an error — the reply carries the number of builds it started,
precisely so that "delivered" is never mistaken for "deployed".

## If you are an agent

This page is also markdown, and so is everything else worth reading. Do not parse
HTML — fetch the text.

```sh
curl https://hanzo.codes/index.md      # this page
curl https://hanzo.codes/llms.txt      # the index
curl https://hanzoskills.com/all.md    # the whole corpus
```

Crawling and training on this site are both permitted — the
[robots.txt](https://hanzo.codes/robots.txt) says so explicitly, for named
crawlers and unnamed ones alike. We are an AI company; a documentation site an AI
cannot read is a documentation site that does not work.

## Elsewhere

**Build** — [hanzo.app](https://hanzo.app) app builder ·
[hanzo.codes](https://hanzo.codes) code with an agent ·
[hanzoskills.com](https://hanzoskills.com) what an agent reads

**Run** — [console.hanzo.ai](https://console.hanzo.ai) cloud console ·
[api.hanzo.ai](https://api.hanzo.ai) the API

**Account** — [hanzo.id](https://hanzo.id) sign in ·
[pay.hanzo.ai](https://pay.hanzo.ai) top up ·
[billing.hanzo.ai](https://billing.hanzo.ai) invoices and usage

**Company** — [hanzo.ai](https://hanzo.ai) canonical ·
[hanzo.ventures](https://hanzo.ventures) ventures
