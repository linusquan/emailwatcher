# Email Digestor → Notion Knowledge Ingest

## Goal

Process the inboxes and lookback window defined in `emailwatcher.config` and convert genuinely useful information from emails and attachments into structured, long-term Notion knowledge.

Notion is a curated knowledge base, **not an email archive**.

Only high-value, factual, transactional, actionable, or reference-worthy information should be written into Notion.

Marketing, promotional content, newsletters, generic announcements, and low-value email noise must not be injected.

---

## How to load these rules

This file is a spine, not the whole spec. **Read every file under `rules/`
before processing any mail** — they are mandatory instructions, not optional
background reference. Do not begin fetching until all seven are loaded.

---

## Processing pipeline

Each run follows this order:

| # | Stage | Rules |
|---|---|---|
| 0 | **Pinned senders** — addresses listed in `emailwatcher.config` bypass stages 2–3 and route by name | [rules/00-pinned-senders.md](./rules/00-pinned-senders.md) |
| 1 | **Fetch** — load messages in the configured lookback window; deep-read full bodies and threads | [rules/01-fetch.md](./rules/01-fetch.md) |
| 2 | **Pre-filter** — discard auto-replies and delivery noise **before** classification | [rules/02-prefilter.md](./rules/02-prefilter.md) |
| 3 | **Classify** — quality gate (`USEFUL` / `PROMOTIONAL` / `LOW_VALUE` / `UNCERTAIN`) | [rules/03-classify.md](./rules/03-classify.md) |
| 4 | **Attachments** — open and extract, for `USEFUL` candidates only | [rules/04-attachments.md](./rules/04-attachments.md) |
| 5 | **Route & integrate** — match `email inject` targets, deduplicate, write Notion | [rules/05-route-and-write.md](./rules/05-route-and-write.md) |
| 6 | **Report** — complete the useful queue, then report counts and outcomes | [rules/06-report.md](./rules/06-report.md) |

---

## Global rules

These apply at every stage and override anything in the stage files —
**except for pinned senders**, which override these in turn.

> **Precedence:** `pinned_sender` (config) > global rules > stage rules.
>
> Mail from an address listed as `pinned_sender` in `emailwatcher.config` is
> exempt from both global rules below. Do not apply the six-month test to it
> and do not skip it for being uncertain or newsletter-shaped. The user pinned
> the sender; that decision is already made. The only reason a pinned email may
> fail to reach Notion is that its designated page does not exist.
> See [rules/00-pinned-senders.md](./rules/00-pinned-senders.md).

### Notion cleanliness

The knowledge base must stay high-signal.

Before every Notion write **that is not from a pinned sender**, ask:

> "Would the user reasonably want to find this information six months from now?"

If the answer is no, do not insert it.

Good Notion knowledge:

```text
Policy renewal: 27 Sep 2026
Premium: $1,720
Excess: $800
Policy number: HPL12345
```

Bad Notion content:

```text
Great news!
As one of our valued customers,
you can save 10% when you...
```

### Conservative filtering

Applies to all mail **except pinned senders**.

When deciding between:

```text
possibly useful
```

and:

```text
probably noise
```

choose:

```text
skip
```

The goal is not maximum email coverage.

The goal is a **clean, trustworthy personal knowledge base**.

False-positive insertion is worse than missing a low-value email.
