# 6. Complete the Queue and Report

← [index.md](../index.md) · prev: [05-route-and-write.md](./05-route-and-write.md)

## Complete useful-queue processing

Classification alone is not completion. Every run must **write or explicitly
skip** each email that is `USEFUL` and has a strong route to an eligible
`email inject` page.

Do **not**:

* stop after the newest or easiest items
* write only one maintenance row when multiple completed jobs exist
* skip older `USEFUL` mail because the lookback window is large
* treat “classified useful” as done without a Notion update or dedup match

For each `USEFUL` email with a destination:

1. Fetch full body (and attachments if present).
2. Read the target Notion page.
3. Check dedup identifiers (message ID, job number, invoice number, etc.).
4. Integrate into the existing page structure, **or**
5. Record as duplicate ignored with the matching identifier.

The completion report must account for every routed `USEFUL` email:

* **Written** — new or updated Notion content
* **Duplicate ignored** — already present under the same identifier
* **Skipped** — only when no eligible `email inject` target exists, or
  confidence is genuinely insufficient after full body + attachment review

A run that classifies dozens of useful property or insurance emails but
updates Notion for only a recent subset is incomplete.

---

## Completion report

After processing, return a concise report.

**Emails scanned** is every message fetched in the lookback window.
**Pinned** is mail from a `pinned_sender` address, which bypasses pre-filter and
classification ([00-pinned-senders.md](./00-pinned-senders.md)).
**Pre-filtered** is mail discarded in [02-prefilter.md](./02-prefilter.md) without
classification. The classification counts (useful, promotional, low value,
uncertain) apply only to mail that passed pre-filter — they sum to
`scanned − pinned − pre-filtered`, not to `scanned`.

**Pinned senders with no destination** must be listed explicitly, by sender
address — no `email inject` page claims them with an `email from` marker. This is
the only sanctioned reason for a pinned email not to reach Notion, so it is an
action item for the user, not a routine skip. If a pinned email is missing from
Notion for any other reason, the run is incorrect — say so under `Errors`, and
likewise when two pages claim the same sender.

Example:

```text
Emails scanned: 9146
Pinned senders (forced useful): 4
Pre-filtered (auto-replies / bounces): 142

Useful (classified): 96
Useful with Notion route: 59
  - Written to Notion: 12
  - Duplicates ignored: 37
  - Skipped after full review: 10

Promotional / spam: 1586
Low value: 1514
Uncertain: 5804

Full bodies read (useful candidates): 59
Attachments read: 8

Notion updates:
- Properties
  - Added front door install — $1,210 (job 2607023873)
  - Updated tenancy and council rates
- Insurance Policies
  - Updated AAMI renewal premium
- Patreon Creator Updates
  - Added 3 creator updates (money_or_life@creator.patreon.com)

Pinned senders with no destination (action required):
- news@creator.patreon.com — no email inject page claims this sender
  (add `email from: news@creator.patreon.com` to the intended page)

Skipped examples (promotional / low value):
- AAMI multi-policy promotional offer
- Qantas marketing newsletter
- Amazon shipment notifications

Pre-filtered examples (not classified):
- Mailer-daemon non-delivery report
- Out-of-office automatic reply

Errors: None
```

Include **Pre-filtered**, **Useful with Notion route**, and the written /
duplicate / skipped breakdown so it is obvious when mail left the pipeline
before classification and when classified useful mail was not ingested.

Do not include skipped promotional email content in Notion.
