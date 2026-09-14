# 3. Content Quality Gate — Apply Before Any Notion Write

← [index.md](../index.md) · prev: [02-prefilter.md](./02-prefilter.md) · next: [04-attachments.md](./04-attachments.md)

> **Skip this stage** for mail from an address listed as `pinned_sender` in
> `emailwatcher.config` — treat it as `USEFUL` without evaluating it. In
> particular, the LOW_VALUE entries for newsletters and recurring updates do
> **not** apply to pinned senders. See
> [00-pinned-senders.md](./00-pinned-senders.md).

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

### Tax deduction candidates

Treat as `USEFUL` when the email or its attachments contain **factual purchase or
charge records** that may be relevant for tax record-keeping — not when it is
only marketing.

**Ingest (extract facts only):**

* invoices and receipts (including PDF/image attachments)
* subscription renewal or charge confirmations (amount, vendor, billing period)
* device or hardware purchase confirmations and tax invoices
* payment confirmations that include what was bought, when, and for how much

**Extract and record as candidates — do not decide deductibility:**

* amount (and currency)
* vendor / merchant
* purchase or charge date
* description of what was bought or subscribed to
* payment method, when stated
* invoice or order number, when present

**Do not give tax advice.** Do not assert that something is deductible, claim a
deduction percentage, or recommend a tax treatment. Record extracted facts as
**tax-deduction candidates** for the user to review at tax time.

**Classify as `PROMOTIONAL` (not ingest):** sales pitches, discount offers,
unused coupon codes, "save on your next purchase", upsell campaigns — even when
they mention subscriptions or devices.

**Classify as `UNCERTAIN` (skip Notion):** the email might be purchase-related
but lacks enough facts to record a candidate (no amount, no clear vendor, no
date, or only a vague "your order" with no detail). Prefer skip over guessing.

Example:

> Apple tax invoice for Magic Trackpad — A$179, order W1540167101, 26 Aug 2026.

→ `USEFUL` (tax-deduction candidate — record facts, not deductibility)

Example:

> "Renew Cursor Pro today and save 20% on your annual plan!"

→ `PROMOTIONAL`

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

## Transactional content beats marketing content

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
