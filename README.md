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
| `Emailer-Agent.md` | Full behavior spec: classification, deep-read rules, attachments, routing, dedup, complete useful-queue processing, reporting. |
| `START.md` | Prompt to give the automation to kick off a run. |
| `docs/system-design/` | Architecture overview, pipeline notes, and Excalidraw diagrams. |

## Run Parameters (`emailwatcher.config`)

Plain text, `key = value` per line.

| Key | Format | Example |
|---|---|---|
| `timerange` | `<number> <unit>` | `1 day`, `1 week`, `1 month`, `1 year` |
| `inboxes` | comma-separated email addresses | `a@example.com, b@example.com` |

The agent should read this file at the start of each run and only fetch
messages from the listed inboxes received within the lookback window.

Current values: see `emailwatcher.config`.

### Destination store

Notion — only pages/databases tagged `email inject` are eligible targets.
See `Emailer-Agent.md` for full routing and write rules.

## How the automation should use this

1. Read `emailwatcher.config` for the lookback window and inboxes.
2. Read `Emailer-Agent.md` for the full processing/classification/routing logic.
3. Apply that logic to the configured inboxes for the configured window.

See `START.md` for the prompt to kick off a run.
