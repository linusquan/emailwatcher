# 2. Pre-filter — Auto-replies and delivery noise

← [index.md](../index.md) · prev: [01-fetch.md](./01-fetch.md) · next: [03-classify.md](./03-classify.md)

> **Skip this stage** for mail from an address listed as `pinned_sender` in
> `emailwatcher.config`, except for genuine non-delivery reports. See
> [00-pinned-senders.md](./00-pinned-senders.md).

After fetch and **before** the quality gate, automatically discard messages that
are never candidates for Notion ingestion.

Pre-filtered mail is **not classified** — it does not receive a
`USEFUL` / `PROMOTIONAL` / `LOW_VALUE` / `UNCERTAIN` label, is not deep-read,
and is never written to Notion.

## Always pre-filter (discard without classifying)

Use sender, subject, and listing metadata only — **do not** fetch the full body
for pre-filtered messages.

* **Bounce / non-delivery** — e.g. `mailer-daemon@`, `postmaster@`, subjects
  or bodies indicating undeliverable, delivery failure, returned mail, or
  non-delivery report (NDR).
* **Out-of-office / auto-replies** — automatic replies, vacation responders,
  away messages, and similar system-generated acknowledgements with no
  transactional content.

## Do not pre-filter

When a message might contain durable facts (invoice, renewal, maintenance,
policy change, payment confirmation), **pass it through** to classification even
if the sender looks automated. When in doubt, classify — do not pre-filter.
