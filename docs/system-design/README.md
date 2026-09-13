# EmailWatcher — System Design

Living system design for EmailWatcher. Keep diagrams (Excalidraw) and prose (Markdown) in sync when architecture, integrations, or processing rules change.

**Companion:** [Project context](../project-context.md) (operational source of truth for purpose, config, and rules).  
**Repo:** [github.com/linusquan/emailwatcher](https://github.com/linusquan/emailwatcher)

---

## Map

| Doc / diagram | What it covers |
|---|---|
| [Architecture overview](./architecture.md) | Components, trust boundaries, data stores, constraints |
| [Processing pipeline](./pipeline.md) | End-to-end run steps, quality gate, Notion write path |
| [diagrams/system-context.excalidraw](./diagrams/system-context.excalidraw) | Actors ↔ EmailWatcher ↔ Gmail / Outlook / Notion / Cursor |
| [diagrams/processing-pipeline.excalidraw](./diagrams/processing-pipeline.excalidraw) | Fetch → classify → route → integrate → report |
| [diagrams/quality-gate.excalidraw](./diagrams/quality-gate.excalidraw) | USEFUL / PROMOTIONAL / LOW_VALUE / UNCERTAIN decisions |
| [diagrams/notion-routing.excalidraw](./diagrams/notion-routing.excalidraw) | `email inject` discovery, match confidence, integrate vs skip |

---

## Design principles

1. **Repo files are the structured prompt** — every run loads `README.md`, `emailwatcher.config`, and `Emailer-Agent.md`.
2. **Notion is curated knowledge, not an email archive** — false positives cost more than misses.
3. **Conservative by default** — skip when uncertain; only `USEFUL` may write.
4. **Strong routing** — writes only to pages marked `email inject`, and only on high-confidence match.
5. **Integrate, don’t dump** — update structured records; never paste whole emails.
6. **Deduplicate across scheduled runs** — prefer update over insert.

---

## How to maintain this space

- Prefer **one diagram per concern**; keep labels short and aligned with names in `Emailer-Agent.md`.
- When behavior changes in the repo or project context, update the matching markdown **and** Excalidraw in the same change.
- Do not invent runtime components that are not in the repo or Cursor automation — this system is instruction-driven agents + MCPs, not application servers.
- Open `.excalidraw` files in [Excalidraw](https://excalidraw.com) (or any Excalidraw-compatible viewer) to edit.

---

## Changelog

| Date | Change |
|---|---|
| 2026-09-13 | Sync after lookback change: document `timerange = 1 day`, Email Watcher + Design Sync automations, pipeline diagram fetch label. |
| 2026-09-13 | Initial system-design space: overview, pipeline docs, four Excalidraw diagrams. |
