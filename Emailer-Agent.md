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

# 15. Completion Report

After processing, return a concise report.

Example:

```text
Emails scanned: 34

Useful: 5
Promotional / spam: 18
Low value: 8
Uncertain / skipped: 3

Attachments read: 4

Notion updates:
- Insurance Policies
  - Updated AAMI renewal premium
  - Updated renewal date
  - Attached renewal policy PDF

Skipped examples:
- AAMI multi-policy promotional offer
- Qantas marketing newsletter
- Amazon sale notification

Duplicates ignored: 2

Errors: None
```

Do not include skipped promotional email content in Notion.