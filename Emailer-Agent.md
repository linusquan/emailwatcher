# Email Digestor → Notion Knowledge Ingest

## Goal

Process the inboxes and lookback window defined in `emailwatcher.config` and convert genuinely useful information from emails and attachments into structured, long-term Notion knowledge.

Notion is a curated knowledge base, **not an email archive**.

Only high-value, factual, transactional, actionable, or reference-worthy information should be written into Notion.

Marketing, promotional content, newsletters, generic announcements, and low-value email noise must not be injected.

---

## 1. Fetch Emails

Read emails from the inboxes listed in `emailwatcher.config`, received within the configured lookback window.

For each email inspect:

* Sender
* Subject
* Received date/time
* Email body
* Links
* Attachment names and types
* Web link / permalink to the message itself (for later provenance in Notion)

Do not decide relevance from the subject line alone.

### Deep read — required before any Notion write

Listing or preview text alone is **not enough** to classify or extract knowledge.

For every email that might be `USEFUL`, fetch and read the **full message body**
before deciding what to write. Previews and snippets often truncate the fields
that matter most (amounts, job titles, policy numbers, dates).

This is **mandatory** when any of the following appear in the sender, subject,
or preview — even if the subject looks generic (e.g. a property address only):

* maintenance, repair, work order, invoice approved, completed job
* quote, receipt, payment, statement, renewal, cancellation
* lease, tenancy, rates, direct debit
* insurance, policy, registration
* attachment present on a property-manager, insurer, council, or tradie email

**Example of why this matters:**

> Subject: `Maintenance Notification: Invoice Approved for Completed Job at 21 Tori Street...`
> Preview ends at: `Job Number`
> Full body contains: `Job Title: front door install`, `Invoice Amount: $1,210.00`

Without the full body, durable cost and work details are invisible.

### Property and maintenance threads

Emails about the same property often reuse generic subjects such as
`21 Tori St, Box Hill NSW 2765` or `FW: 21 Tori St, Box Hill NSW 2765`.

When a property address, managing agent, or maintenance provider appears:

1. Read the **full thread or conversation**, not just the newest message.
2. Follow forwards and replies — quotes and approvals may be in different messages.
3. Do not assume one maintenance write-up covers all jobs at that address.

---

# 2. Content Quality Gate — Apply Before Any Notion Write

Before processing an email further, classify it as one of:

* `USEFUL`
* `PROMOTIONAL`
* `LOW_VALUE`
* `UNCERTAIN`

Only `USEFUL` emails may update Notion.

If uncertain whether an email contains durable knowledge, default to **not inserting it**.

## USEFUL

An email is useful when it contains information such as:

* a real policy or contract
* renewal notice
* invoice or receipt
* insurance quote
* account statement
* booking confirmation
* payment confirmation
* important service notice
* change to an existing agreement or product
* expiry date
* due date
* premium or price change
* policy or account reference
* claim information
* official correspondence
* tax or financial records
* maintenance / repair information
* appointment or reservation details
* personally relevant regulatory or administrative information
* documents or attachments that may need to be referenced later
* an action the user genuinely needs to take

Example:

> AAMI landlord insurance renewal notice showing the new premium, policy number and renewal date.

→ `USEFUL`

---

## PROMOTIONAL

Do NOT ingest:

* sales promotions
* discounts
* special offers
* upselling
* cross-selling
* advertising
* marketing campaigns
* generic product recommendations
* "you may also like"
* seasonal promotions
* loyalty promotions
* competitions
* referral campaigns
* promotional credit-card offers
* promotional insurance offers unrelated to an existing policy or genuine requested quote

Example:

> "Save 20% when you add car insurance today!"

→ `PROMOTIONAL`

Do not write it to Notion.

---

## LOW_VALUE

Do NOT ingest emails that technically relate to a topic but provide no meaningful new information.

Examples:

* "Thanks for being our customer"
* generic welcome emails
* generic reminders with no important dates or details
* marketing-style educational material
* newsletters
* weekly/monthly company news
* social notifications
* survey requests
* generic usage tips
* duplicated notifications
* repeated emails containing no new information
* boilerplate correspondence
* package or shipping tracking updates with no delay, exception, or delivery problem

These should not pollute the knowledge base.

---

## UNCERTAIN

If the email could be relevant but does not clearly contain useful long-term information:

Do not update Notion automatically.

Prefer skipping it over injecting questionable content.

---

# 3. Transactional Content Beats Marketing Content

Some emails contain both useful and promotional content.

Extract only the useful portion.

Example:

```text
Your landlord policy renews on 27 September.
New premium: $1,750.

Bundle your car insurance today and save 10%.
```

Keep:

```text
Renewal date: 27 September
New premium: $1,750
```

Ignore:

```text
Bundle your car insurance today and save 10%.
```

Never copy marketing boilerplate simply because it appears inside an otherwise useful email.

---

# 4. Read Attachments

For `USEFUL` emails with attachments:

1. Download the attachments.
2. Open and read the actual attachment content.
3. Do not rely only on the email body or filename.
4. Extract durable information from the attachment.
5. Preserve the attachment where useful as supporting evidence.

**Attachments are not optional for ingestion.** If an email is classified
`USEFUL` and has attachments, the run is incomplete until each attachment
has been opened and checked for authoritative facts (amounts, dates,
reference numbers, scope of work). Reporting `Attachments read: 0` while
writing Notion updates from `USEFUL` mail is a processing failure.

Priority attachment types for property and finance mail:

* quotes and estimates (including tradie / agent forwards)
* invoices and payment receipts
* lease and tenancy documents
* policy schedules, certificates, renewal packs

Relevant attachment types can include:

* PDF
* DOCX
* XLSX
* image
* invoice
* statement
* quote
* policy schedule
* renewal document
* receipt
* contract
* booking confirmation

If an email looks promotional but contains an attached authoritative document, inspect the document before deciding whether to discard the email.

---

# 5. Discover Notion Injection Targets

Search Notion for pages/documents intended to receive email-derived information.

A page is eligible for automatic email injection only when it contains the marker:

`email inject`

This may appear as a tag, property, or explicit metadata text.

Examples:

```text
Tags: email inject
```

or a database tag equivalent.

Do not automatically write email content into arbitrary Notion pages.

---

# 6. Route the Email to the Correct Document

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

If no existing `email inject` document is a strong match:

**do not inject the content into an unrelated document.**

---

# 7. High Confidence Required Before Writing

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

# 8. Update Existing Knowledge, Do Not Dump Emails

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

# 9. Deduplication

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

# 10. Prefer New Authoritative Information

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

# 11. Preserve Source Traceability

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

---

# 12. Attachment Handling

If an attachment is useful:

* attach it to the relevant Notion section where supported;
* use a meaningful filename;
* extract its important information into visible page content.

The user should be able to understand the current state without opening every attachment.

Do NOT store promotional brochures or generic marketing PDFs.

---

# 13. Notion Cleanliness Rule

The knowledge base must stay high-signal.

Before every Notion write, ask:

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

---

# 14. Conservative Filtering

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

---

# 15. Complete Useful-Queue Processing

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

# 16. Completion Report

After processing, return a concise report.

Example:

```text
Emails scanned: 9146

Useful (classified): 96
Useful with Notion route: 59
  - Written to Notion: 12
  - Duplicates ignored: 37
  - Skipped after full review: 10

Promotional / spam: 1586
Low value: 1514
Uncertain: 5950

Full bodies read (useful candidates): 59
Attachments read: 8

Notion updates:
- Properties
  - Added front door install — $1,210 (job 2607023873)
  - Updated tenancy and council rates
- Insurance Policies
  - Updated AAMI renewal premium

Skipped examples (promotional / low value):
- AAMI multi-policy promotional offer
- Qantas marketing newsletter
- Amazon shipment notifications

Errors: None
```

Include **Useful with Notion route** and the written / duplicate / skipped
breakdown so it is obvious when classified useful mail was not ingested.

Do not include skipped promotional email content in Notion.