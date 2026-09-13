# Processing pipeline

Canonical run flow for a scheduled EmailWatcher agent. Aligns with `index.md` + `rules/` and [project-context.md](../project-context.md).

Diagram: [processing-pipeline.excalidraw](./diagrams/processing-pipeline.excalidraw).

---

## Stages

### 0. Startup (mandatory)

1. Read `README.md`.
2. Load `emailwatcher.config` (`timerange`, `inboxes`).
3. Load `index.md` **and every file under `rules/`** (full rules).
4. Do **not** use defaults from the automation prompt.

### 1. Fetch

- Query each configured inbox for messages in the lookback window (`timerange` in `emailwatcher.config`; currently **`3 day`**).
- Capture: sender, subject, received time, body, links, attachment names/types, `webLink` / permalink.
- **Deep read:** for anything that might be USEFUL, fetch the **full body** (previews truncate durable fields). Property/maintenance threads: read the conversation, not only the latest message.

### 1b. Pinned-sender bypass

Mail whose sender matches a `pinned_sender` entry in `emailwatcher.config` leaves the
normal path here. It skips pre-filter (except genuine NDRs) and the quality gate, is
treated as USEFUL, and routes to whichever `email inject` page claims the sender with an
`email from` marker.

Config holds addresses only — no Notion page names live in the repo. The six-month test
and conservative filtering do **not** apply. Deduplication, deep read, attachment
extraction, and integrate-don't-dump do. The only sanctioned non-write outcome is "no page
claims this sender", reported as an action item; two claiming pages is an error.

Rules: `rules/00-pinned-senders.md`.

### 2. Pre-filter

Before classification, automatically discard messages that are never candidates for
Notion ingestion — using sender, subject, and listing metadata only (no full-body fetch):

- Bounce / non-delivery reports (`mailer-daemon@`, `postmaster@`, NDR).
- Out-of-office / auto-replies with no transactional content.

Pre-filtered mail is **not classified**, not deep-read, and never written to Notion. When
a message might contain durable facts, it passes through to classification even if the
sender looks automated — when in doubt, classify rather than pre-filter.

Diagram: [processing-pipeline.excalidraw](./diagrams/processing-pipeline.excalidraw).

### 3. Quality gate

Classify every email that survives pre-filter before any Notion write:

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

### 4. Attachments (USEFUL only)

If USEFUL and attachments exist: download, open, and extract authoritative facts. Reporting Notion updates with `Attachments read: 0` for USEFUL mail that had attachments is a processing failure.

### 5. Discover & route

1. Search Notion for `email inject` targets.
2. Read candidates; match on subject, sender, entities, refs, semantic fit.
3. **High confidence required** — one keyword is not enough.
4. No strong match → **do not write**.

Diagram: [notion-routing.excalidraw](./diagrams/notion-routing.excalidraw).

### 6. Integrate

- Read the target page first.
- Update records / dates / statuses; maintain tables; add actions when needed.
- Deduplicate (message id, refs, amounts, dates, attachment names, etc.).
- Prefer newest authoritative values; keep history only when useful.
- Light provenance: sender, date, subject, attachment name, reference, email link.

### 7. Completion report

Emit counts and outcomes per `rules/06-report.md`: scanned, pre-filtered, useful / promotional /
low-value / uncertain, attachments read, Notion updates, skips, duplicates, errors.

---

## Failure / skip modes (by design)

| Situation | Expected behavior |
|---|---|
| Pinned sender, claimed by a page | Always write (dedup still applies) |
| Pinned sender, unclaimed | Skip + report as action item — the only sanctioned pinned skip |
| Pinned sender, claimed by 2+ pages | Do not write — report under `Errors` |
| Bounce / auto-reply | Pre-filter — discard without classifying |
| Uncertain durability | Skip write |
| No `email inject` match | Skip write |
| Weak keyword-only match | Skip write |
| Promotional / low-value | Skip write |
| Duplicate of existing knowledge | Ignore or update in place |
| Mixed transactional + promo | Keep facts only |

These are **correct outcomes**, not outages.
