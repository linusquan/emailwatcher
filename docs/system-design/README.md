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
| [diagrams/processing-pipeline.excalidraw](./diagrams/processing-pipeline.excalidraw) | Fetch → pre-filter → classify → route → integrate → report |
| [diagrams/quality-gate.excalidraw](./diagrams/quality-gate.excalidraw) | USEFUL / PROMOTIONAL / LOW_VALUE / UNCERTAIN decisions |
| [diagrams/notion-routing.excalidraw](./diagrams/notion-routing.excalidraw) | `email inject` discovery, match confidence, integrate vs skip |

---

## Design principles

1. **Repo files are the structured prompt** — every run loads `README.md`, `emailwatcher.config`, `index.md`, and all six `rules/*.md` stage files.
2. **Notion is curated knowledge, not an email archive** — false positives cost more than misses.
3. **Conservative by default** — skip when uncertain; only `USEFUL` may write.
4. **Strong routing** — writes only to pages marked `email inject`, and only on high-confidence match.
5. **Integrate, don’t dump** — update structured records; never paste whole emails.
6. **Deduplicate across scheduled runs** — prefer update over insert.

---

## How to maintain this space

- Prefer **one diagram per concern**; keep labels short and aligned with the stage names in `rules/`.
- When behavior changes in the repo or project context, update the matching markdown **and** Excalidraw in the same change.
- Do not invent runtime components that are not in the repo or Cursor automation — this system is instruction-driven agents + MCPs, not application servers.
- Open `.excalidraw` files in [Excalidraw](https://excalidraw.com) (or any Excalidraw-compatible viewer) to edit.

---

## Changelog

| Date | Change |
|---|---|
| 2026-09-13 | Lookback widened to `timerange = 3 day`; architecture, pipeline and the fetch diagram label updated. |
| 2026-09-13 | Pinned senders: new `pinned_sender` config key (addresses only) and `rules/00-pinned-senders.md`. Listed addresses bypass pre-filter and the quality gate, and are exempt from the six-month test and conservative filtering. Destinations are declared in Notion via an `email from` marker on an `email inject` page — no Notion page names in the repo. An unclaimed pinned sender is the only sanctioned non-write; two claims is an error. Report gains pinned counts and a "no destination" action list. Diagrams updated: processing-pipeline gains the green bypass arc (fetch → attachments), quality-gate and notion-routing gain pinned annotations. |
| 2026-09-13 | Behavior spec split: `Emailer-Agent.md` → `index.md` (spine: goal, pipeline order, global rules) + six stage files under `rules/`. Citations changed from `§N` to file paths; `README.md`/`START.md` load instructions updated. |
| 2026-09-13 | Sync after Pre-filter stage added (PR #6): document new pre-filter step (bounces/non-delivery, out-of-office auto-replies discarded before classification), renumber pipeline stages, update processing-pipeline and quality-gate diagrams. |
| 2026-09-13 | Sync after lookback change: document `timerange = 1 day`, Email Watcher + Design Sync automations, pipeline diagram fetch label. |
| 2026-09-13 | Initial system-design space: overview, pipeline docs, four Excalidraw diagrams. |
