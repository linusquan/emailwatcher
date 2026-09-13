---
name: sync-system-design
description: After merges to main that change EmailWatcher behavior or architecture, update docs/system-design/ (README, architecture.md, pipeline.md, four Excalidraw diagrams) to match. Skip when only docs/system-design/ changed or nothing architectural changed. Never edit emailwatcher.config or processing rules; never merge the follow-up PR.
---

# Sync system design docs

Use when a merge to `main` may have changed how EmailWatcher works or is structured. Target directory: **`docs/system-design/`**.

## When to run

**Update** when the merged PR touched any of:

- `index.md`
- `rules/*.md`
- `emailwatcher.config`
- `README.md`
- Application code (if any is added later)

**Do nothing** (prevent loops) when:

- The merge **only** changed files under `docs/system-design/`, or
- Nothing architectural or behavioral changed (typos, `.gitignore`, etc.)

## What to update

Keep these in sync with the repo and with each other:

| Artifact | Path |
|---|---|
| Index | `docs/system-design/README.md` |
| Architecture prose | `docs/system-design/architecture.md` |
| Pipeline prose | `docs/system-design/pipeline.md` |
| System context diagram | `docs/system-design/diagrams/system-context.excalidraw` |
| Processing pipeline diagram | `docs/system-design/diagrams/processing-pipeline.excalidraw` |
| Quality gate diagram | `docs/system-design/diagrams/quality-gate.excalidraw` |
| Notion routing diagram | `docs/system-design/diagrams/notion-routing.excalidraw` |

Use `excalidraw-diagram` skill to improve the diagramming.

## Rules

1. **Read first** — `README.md`, `emailwatcher.config`, `index.md`, all `rules/*.md`, and existing `docs/system-design/` before editing.
2. **Match reality** — Document instruction-driven Cursor automations + MCPs (Gmail, Outlook, Notion). Do not invent servers, queues, or databases not in the repo.
3. **Do not change processing rules** — Do not edit `index.md` or `rules/*.md` classification, routing, dedup, or report rules as part of this sync.
4. **Do not change run parameters** — Do not edit `emailwatcher.config` (`timerange`, `inboxes`, `pinned_sender`).
5. **Diagrams + prose together** — When behavior or config surface changes (e.g. lookback window), update both markdown and the relevant `.excalidraw` labels in the same PR.
6. **Changelog** — Add a dated row to the README changelog when you make substantive updates.
7. **Open a PR, do not merge** — Create a follow-up PR on a `cursor/` branch; leave it for human review.

## Workflow

1. Confirm the merged diff is in scope (see “When to run”).
2. Identify what changed (config values, new automations, pipeline steps, integrations).
3. Update matching markdown and diagrams.
4. Commit on `cursor/<descriptive-name>-<suffix>`, push, open PR against `main`.
5. In the PR description, list which merge triggered the sync and what design artifacts changed.

## Reference

- Operational detail: root `README.md`, `index.md`, `rules/*.md`, `emailwatcher.config`
- Design space index: `docs/system-design/README.md`
