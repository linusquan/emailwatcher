# Architecture overview

EmailWatcher is an **instruction-driven Cursor Cloud automation**, not a deployed application server. Behavior lives in repo markdown/config; execution uses MCP integrations (Gmail, Outlook, Notion) inside scheduled agent runs.

```text
┌─────────────────────────────────────────────────────────────┐
│                     Cursor Cloud                             │
│  Scheduled automation “Email Watcher”                        │
│  → agent run loads repo instructions → MCP tool calls        │
└───────────────┬─────────────────────┬───────────────────────┘
                │                     │
       fetch mail (MCPs)      search/update (MCP)
                │                     │
        ┌───────┴───────┐      ┌──────┴──────┐
        │ Gmail inbox   │      │   Notion    │
        │ Outlook inbox │      │ email inject│
        └───────────────┘      └─────────────┘
```

See also: [system-context.excalidraw](./diagrams/system-context.excalidraw).

---

## Components

| Component | Kind | Role |
|---|---|---|
| **Email Watcher automation** | Scheduled runner | Processes inboxes per repo instructions on a schedule |
| **Design Sync automation** | Merge-triggered runner | After merges to `main`, updates `docs/system-design/` when architecture/behavior files change |
| **Agent run** | Ephemeral worker | Reads instructions, fetches mail, classifies, writes Notion, reports |
| **`README.md`** | Entry index | Points the agent at config and behavior spec |
| **`emailwatcher.config`** | Run parameters | `timerange`, `inboxes` |
| **`index.md`** | Behavior spine | Goal, pipeline order, global cleanliness / conservative-filtering rules, links to `rules/` |
| **`rules/*.md`** | Behavior spec | Six stage files: fetch, pre-filter, classify, attachments, route & write, report |
| **`START.md`** | Kickoff prompt | Canonical automation start text |
| **Gmail MCP** | Email source | Inbox `liquansyd@gmail.com` |
| **Outlook MCP** | Email source | Inbox `liquan1992@outlook.com` |
| **Notion MCP** | Knowledge sink | Only pages/databases marked `email inject` |

There is **no** custom app runtime, database, or message queue in-repo. Persistence of knowledge is Notion; provenance prefers message `webLink` / deep links.

---

## Trust & write boundaries

| Boundary | Rule |
|---|---|
| Config | Always from repo files — never baked automation defaults |
| Ingress | Only listed inboxes; only messages inside `timerange` |
| Pinned senders | `pinned_sender` addresses bypass pre-filter + quality gate and route to the `email inject` page claiming them via `email from`; only an unclaimed sender may block the write |
| Pre-filter | Bounces / non-delivery / auto-replies discarded before classification; never deep-read, never written |
| Classification | Must complete before any Notion write, unless the sender is pinned |
| Egress | Only `USEFUL` (or pinned) → Notion; PROMOTIONAL / LOW_VALUE / UNCERTAIN → no write |
| Targets | Only Notion pages with `email inject` marker |
| Routing | High-confidence document match required; weak keyword match → skip |
| Content | Integrate into existing structure; strip marketing / boilerplate |

---

## Data flow (logical)

1. **Load** — README → config → `index.md` → all `rules/*.md`.
2. **Fetch** — per inbox, messages in lookback window (full body when possibly useful).
3. **Pre-filter** — discard bounces/non-delivery and out-of-office auto-replies before classification (metadata only, no full-body fetch, never written to Notion).
4. **Classify** — quality gate labels each remaining message.
5. **Enrich** — for USEFUL: attachments, thread context where required (e.g. property/maintenance).
6. **Route** — discover `email inject` targets; pick best strong match or skip.
7. **Integrate** — dedupe, prefer newest authoritative values, record light provenance.
8. **Report** — completion report per `rules/06-report.md`.

Detail: [pipeline.md](./pipeline.md) and [processing-pipeline.excalidraw](./diagrams/processing-pipeline.excalidraw).

---

## Configuration surface

| Key | Source | Effect |
|---|---|---|
| `timerange` | `emailwatcher.config` | Lookback window for fetch |
| `inboxes` | `emailwatcher.config` | Allowed mailbox addresses |
| `pinned_sender` | `emailwatcher.config` | Always-useful sender addresses (no destinations) |
| Pinned destinations | Notion metadata | `email from: <address>` on an `email inject` page |
| Classification & routing | `rules/03-classify.md`, `rules/05-route-and-write.md` | Gate, attachments, Notion rules |
| Eligible Notion pages | Notion metadata | Tag/property/text `email inject` |

### Current run parameters (`emailwatcher.config`)

| Key | Current value |
|---|---|
| `timerange` | `3 day` |
| `inboxes` | `liquansyd@gmail.com`, `liquan1992@outlook.com` |
| `pinned_sender` | `money_or_life@creator.patreon.com` |

The fetch stage only retrieves messages received within this lookback window (`1 year` → `1 day` in PR #2; widened to `3 day` since).

---

## Non-goals (architecture)

- Email archival or full-text mail search in Notion
- Automatic creation of new Notion destinations without `email inject`
- Guaranteed ingest of ambiguous mail (skip is correct)
- Changing application code on routine processing runs
