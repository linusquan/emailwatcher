# Email Watcher — Configuration

Settings and run parameters for the email reading agent. The agent reads new
mail from the configured inboxes and writes anything durable/important into
the external knowledge store (Notion). This file is the entry point the
automation should read first to find its settings.

## Files

| File | Purpose |
|---|---|
| `README.md` | This file — index of settings and structure. |
| `emailwatcher.config` | All run parameters (lookback window, inboxes). |
| `index.md` | Behavior spine: goal, pipeline order, global rules, links into `rules/`. |
| `rules/*.md` | Full behavior spec, one file per pipeline stage (see below). |
| `START.md` | Prompt to give the automation to kick off a run. |
| `docs/system-design/` | Architecture overview, pipeline notes, and Excalidraw diagrams. |

## Run Parameters (`emailwatcher.config`)

Plain text, `key = value` per line.

| Key | Format | Example |
|---|---|---|
| `timerange` | `<number> <unit>` | `1 day`, `1 week`, `1 month`, `1 year` |
| `inboxes` | comma-separated email addresses | `a@example.com, b@example.com` |
| `pinned_sender` | one email address, repeatable | `x@example.com` |

### Pinned senders

`pinned_sender` lines force a sender's mail through to Notion. Pinned mail skips
pre-filter and the quality gate, and is exempt from the six-month test and
conservative filtering — newsletter-shaped content is expected and accepted.

The config says **which senders matter**; Notion says **where their mail goes**.
No Notion page names live in this repo. A destination page claims a sender by
carrying an `email from` marker next to its `email inject` marker:

```text
Tags: email inject
email from: money_or_life@creator.patreon.com
```

So adding a pinned sender is two edits: one config line here, one marker in
Notion. Routing is then a lookup — the claiming page, or nothing.

**The only sanctioned reason a pinned email does not reach Notion is that no
page claims its sender.** That is reported as an action item. Two pages claiming
the same sender is an error, not a tiebreak. Deduplication still applies. Full
rules: `rules/00-pinned-senders.md`.

The agent should read this file at the start of each run and only fetch
messages from the listed inboxes received within the lookback window.

Current values: see `emailwatcher.config`.

### Destination store

Notion — only pages/databases tagged `email inject` are eligible targets.
See `rules/05-route-and-write.md` for full routing and write rules.

## Behavior spec (`index.md` + `rules/`)

`index.md` is a spine, not the whole spec. The automation must read **every**
file under `rules/` before processing mail.

| File | Stage |
|---|---|
| `rules/00-pinned-senders.md` | Always-useful senders from config; what they bypass |
| `rules/01-fetch.md` | Fetch, deep read, property/maintenance threads |
| `rules/02-prefilter.md` | Discard bounces and auto-replies before classification |
| `rules/03-classify.md` | Quality gate; transactional beats marketing |
| `rules/04-attachments.md` | Open attachments; handling in Notion |
| `rules/05-route-and-write.md` | `email inject` targets, confidence, integrate, dedup, provenance |
| `rules/06-report.md` | Complete the useful queue; completion report format |

## How the automation should use this

1. Read `emailwatcher.config` for the lookback window and inboxes.
2. Read `index.md`, then every file under `rules/`, for the full processing/classification/routing logic.
3. Apply that logic to the configured inboxes for the configured window.

See `START.md` for the prompt to kick off a run.

## Cursor automations

| Automation | Trigger | Role |
|---|---|---|
| **Email Watcher** | Scheduled | Process inboxes per `emailwatcher.config`, `index.md`, and `rules/` |
| **Design Sync** | Pull request merged → `main` | Keep `docs/system-design/` aligned when behavior or architecture files change |

Design Sync skips merges that only touch `docs/system-design/` to avoid update loops. It opens a follow-up PR but does not merge it.
