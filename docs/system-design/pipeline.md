# Processing pipeline

Canonical run flow for a scheduled EmailWatcher agent. Aligns with `Emailer-Agent.md` and [project-context.md](../project-context.md).

Diagram: [processing-pipeline.excalidraw](./diagrams/processing-pipeline.excalidraw).

---

## Stages

### 0. Startup (mandatory)

1. Read `README.md`.
2. Load `emailwatcher.config` (`timerange`, `inboxes`).
3. Load `Emailer-Agent.md` (full rules).
4. Do **not** use defaults from the automation prompt.

### 1. Fetch

- Query each configured inbox for messages in the lookback window.
- Capture: sender, subject, received time, body, links, attachment names/types, `webLink` / permalink.
- **Deep read:** for anything that might be USEFUL, fetch the **full body** (previews truncate durable fields). Property/maintenance threads: read the conversation, not only the latest message.

### 2. Quality gate

Classify every email before any Notion write:

| Label | Notion write? |
|---|---|
| `USEFUL` | Yes (if strong route) |
| `PROMOTIONAL` | No |
| `LOW_VALUE` | No |
| `UNCERTAIN` | No (prefer skip) |

Rules of thumb:

- Do not classify from subject alone.
- Transactional beats marketing — extract only durable facts from mixed emails.
- Six-month test: would the user want this later? If no → skip.
- When unsure → skip.

Diagram: [quality-gate.excalidraw](./diagrams/quality-gate.excalidraw).

### 3. Attachments (USEFUL only)

If USEFUL and attachments exist: download, open, and extract authoritative facts. Reporting Notion updates with `Attachments read: 0` for USEFUL mail that had attachments is a processing failure.

### 4. Discover & route

1. Search Notion for `email inject` targets.
2. Read candidates; match on subject, sender, entities, refs, semantic fit.
3. **High confidence required** — one keyword is not enough.
4. No strong match → **do not write**.

Diagram: [notion-routing.excalidraw](./diagrams/notion-routing.excalidraw).

### 5. Integrate

- Read the target page first.
- Update records / dates / statuses; maintain tables; add actions when needed.
- Deduplicate (message id, refs, amounts, dates, attachment names, etc.).
- Prefer newest authoritative values; keep history only when useful.
- Light provenance: sender, date, subject, attachment name, reference, email link.

### 6. Completion report

Emit counts and outcomes per Emailer-Agent §15: scanned, useful / promotional / low-value / uncertain, attachments read, Notion updates, skips, duplicates, errors.

---

## Failure / skip modes (by design)

| Situation | Expected behavior |
|---|---|
| Uncertain durability | Skip write |
| No `email inject` match | Skip write |
| Weak keyword-only match | Skip write |
| Promotional / low-value | Skip write |
| Duplicate of existing knowledge | Ignore or update in place |
| Mixed transactional + promo | Keep facts only |

These are **correct outcomes**, not outages.
