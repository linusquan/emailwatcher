# 1. Fetch Emails

← [index.md](../index.md) · next: [02-prefilter.md](./02-prefilter.md)

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

## Deep read — required before any Notion write

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
* tax invoice, purchase confirmation, subscription charge or renewal, order
  confirmation for devices or hardware
* attachment present on a property-manager, insurer, council, or tradie email
* attachment that may be a receipt or invoice (PDF, image) on a merchant or
  subscription email

**Example of why this matters:**

> Subject: `Maintenance Notification: Invoice Approved for Completed Job at 21 Tori Street...`
> Preview ends at: `Job Number`
> Full body contains: `Job Title: front door install`, `Invoice Amount: $1,210.00`

Without the full body, durable cost and work details are invisible.

## Property and maintenance threads

Emails about the same property often reuse generic subjects such as
`21 Tori St, Box Hill NSW 2765` or `FW: 21 Tori St, Box Hill NSW 2765`.

When a property address, managing agent, or maintenance provider appears:

1. Read the **full thread or conversation**, not just the newest message.
2. Follow forwards and replies — quotes and approvals may be in different messages.
3. Do not assume one maintenance write-up covers all jobs at that address.
