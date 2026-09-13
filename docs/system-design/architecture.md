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
| **Cursor automation** | Scheduler / runner | Triggers agent runs against the repo on a schedule |
| **Agent run** | Ephemeral worker | Reads instructions, fetches mail, classifies, writes Notion, reports |
| **`README.md`** | Entry index | Points the agent at config and behavior spec |
| **`emailwatcher.config`** | Run parameters | `timerange`, `inboxes` |
| **`Emailer-Agent.md`** | Behavior spec | Classification, attachments, routing, dedup, report format |
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
| Classification | Must complete before any Notion write |
| Egress | Only `USEFUL` → Notion; PROMOTIONAL / LOW_VALUE / UNCERTAIN → no write |
| Targets | Only Notion pages with `email inject` marker |
| Routing | High-confidence document match required; weak keyword match → skip |
| Content | Integrate into existing structure; strip marketing / boilerplate |

---

## Data flow (logical)

1. **Load** — README → config → Emailer-Agent rules.
2. **Fetch** — per inbox, messages in lookback window (full body when possibly useful).
3. **Classify** — quality gate labels each message.
4. **Enrich** — for USEFUL: attachments, thread context where required (e.g. property/maintenance).
5. **Route** — discover `email inject` targets; pick best strong match or skip.
6. **Integrate** — dedupe, prefer newest authoritative values, record light provenance.
7. **Report** — completion report per Emailer-Agent §15.

Detail: [pipeline.md](./pipeline.md) and [processing-pipeline.excalidraw](./diagrams/processing-pipeline.excalidraw).

---

## Configuration surface

| Key | Source | Effect |
|---|---|---|
| `timerange` | `emailwatcher.config` | Lookback window for fetch |
| `inboxes` | `emailwatcher.config` | Allowed mailbox addresses |
| Classification & routing | `Emailer-Agent.md` | Gate, attachments, Notion rules |
| Eligible Notion pages | Notion metadata | Tag/property/text `email inject` |

Operational snapshot of current values: [project-context.md](../project-context.md).

---

## Non-goals (architecture)

- Email archival or full-text mail search in Notion
- Automatic creation of new Notion destinations without `email inject`
- Guaranteed ingest of ambiguous mail (skip is correct)
- Changing application code on routine processing runs
