# 5. Route and Write to Notion

← [index.md](../index.md) · prev: [04-attachments.md](./04-attachments.md) · next: [06-report.md](./06-report.md)

> **Pinned senders route differently.** Their destination is whichever
> `email inject` page claims the sender via an `email from` marker, so discovery
> and the confidence test below are replaced by a lookup — claimed, or not. See
> [00-pinned-senders.md](./00-pinned-senders.md). Everything from
> *Update existing knowledge* onward (integrate, dedup, provenance) still applies.

## Discover Notion injection targets

Search Notion for pages/documents intended to receive email-derived information.

A page is eligible for automatic email injection only when it contains the marker:

`email inject`

This may appear as a tag, property, or explicit metadata text.

Examples:

```text
Tags: email inject
```

or a database tag equivalent.

A page may additionally claim specific senders with an `email from` marker, which
makes it the destination for those addresses when they are listed as
`pinned_sender` in `emailwatcher.config`:

```text
Tags: email inject
email from: money_or_life@creator.patreon.com
```

`email from` has no effect for senders that are not pinned in config.

Do not automatically write email content into arbitrary Notion pages.

---

## Route the email to the correct document

For every `USEFUL` email:

1. Search eligible `email inject` Notion documents.
2. Read candidate document titles and existing content.
3. Determine whether the email clearly belongs to one of them.
4. Select the best matching destination.

Use:

* subject
* sender
* extracted entities
* document type
* existing Notion content
* policy/account/reference numbers
* semantic similarity
* keyword matching

as routing signals.

Examples:

```text
AAMI renewal notice
→ Insurance Policies
```

```text
Council rates notice
→ Property / Council Rates document
```

```text
Car registration renewal
→ Vehicle document
```

```text
Receipt, tax invoice, subscription charge, or device purchase confirmation
→ Tax Deductions
```

Only when the `email inject` page **Tax Deductions** exists and the email is a
clear tax-deduction **candidate** (see [03-classify.md](./03-classify.md)). Write
extracted purchase facts into the existing table or structure — status as
candidate, not as a decided deduction. Do not add tax advice or deductibility
claims to Notion.

If no `email inject` page clearly matches tax, deductions, expenses,
subscriptions, or devices:

**do not invent a dump page** — skip Notion and note in the completion report
that no eligible destination was found.

---

If no existing `email inject` document is a strong match:

**do not inject the content into an unrelated document.**

---

## High confidence required before writing

Writing to Notion requires a strong match between:

```text
email
        +
attachment contents
        +
target document purpose
```

Do not make routing decisions merely because one keyword happens to match.

For example:

An email saying:

> "Get travel insurance before your next holiday"

contains the word `insurance`, but must **not** be inserted into `Insurance Policies`.

An email saying:

> "Your AAMI landlord insurance policy HPL12345 renews on 27 September"

clearly belongs there.

---

## Update existing knowledge, do not dump emails

Before modifying an eligible target page:

**read the existing page first.**

Integrate new information into the existing structure.

Prefer:

* updating existing records
* refreshing dates
* replacing outdated values
* adding a genuinely new policy / quote / record
* maintaining comparison tables
* updating statuses
* adding required actions
* preserving useful historical values

Do NOT:

* append whole email bodies
* copy signatures
* copy legal boilerplate unnecessarily
* copy promotional sections
* create repeated "Email received..." sections
* create duplicate entries
* create one Notion section per email when the information belongs in an existing structured record

The result should look like a maintained knowledge document.

---

## Deduplication

The workflow may run repeatedly.

Before writing, determine whether the information has already been processed.

Possible identifiers include:

* message ID
* sender
* subject
* received timestamp
* policy number
* invoice number
* quote number
* account number
* booking reference
* attachment filename
* effective date

Do not create duplicates.

If an email contains newer information for an existing record, update the record rather than adding a duplicate.

---

## Prefer new authoritative information

If newer authoritative information conflicts with existing information:

Use the newest authoritative source as the current value.

Examples:

```text
Old premium: $1,580
Renewal premium: $1,720
```

Current premium should become:

```text
$1,720
```

Retain historical values only when they provide useful comparison or audit history.

---

## Preserve source traceability

Important extracted knowledge should retain lightweight provenance.

Where appropriate record:

* sender
* received date
* email subject
* attachment filename
* reference number
* link to the original email (Outlook `webLink` / deep link opening the message directly)

For example:

```text
Source: AAMI renewal email — 12 Sep 2026
Email: https://outlook.office.com/mail/deeplink/...
Attachment: Landlord-Renewal-2026.pdf
```

Do not overwhelm the Notion document with email metadata.

The knowledge itself is more important than the email envelope.
