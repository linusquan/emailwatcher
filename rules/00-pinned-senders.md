# 0. Pinned Senders — Process Regardless

← [index.md](../index.md) · next: [01-fetch.md](./01-fetch.md)

Some senders are known-good in advance. Their mail must reach Notion whenever a
destination exists for it. **The only acceptable reason a pinned sender's email
does not land in Notion is that its designated Notion page does not exist.**

## Where the list comes from

`emailwatcher.config`, one `pinned_sender` line per address:

```text
pinned_sender = money_or_life@creator.patreon.com
```

Match the sender address **exactly** (case-insensitive). Do not fuzzy-match the
sender, and do not match on subject text — a pinned entry is an address, not a
pattern. If the address does not match exactly, the message is ordinary mail and
goes through the normal pipeline.

The config says **which senders matter**. It does not say where their mail goes —
the repo has no knowledge of Notion page names, and must not acquire any.
Destinations are declared in Notion, by the destination itself.

## Where the destination comes from

A Notion page claims a pinned sender by carrying the marker `email from`
alongside its existing `email inject` marker. As with `email inject`, this may be
a tag, a property, or explicit metadata text:

```text
Tags: email inject
email from: money_or_life@creator.patreon.com
```

One page may claim several addresses (comma-separated). Adding a new destination
is a Notion edit — no repo change.

Rules for resolving the claim:

* **No page claims the sender** → skip and report (see below). This is the
  expected, sanctioned outcome when a destination has not been created yet.
* **Exactly one page claims it** → that is the destination. No further judgment.
* **More than one page claims it** → do not guess and do not write. Report under
  `Errors` naming every claiming page; the ambiguity is a configuration mistake
  for the user to resolve in Notion.
* **A page claims a sender that is not in `pinned_sender`** → the sender is not
  pinned. The claim is inert; process the mail through the normal pipeline. The
  config list is what activates pinning.
* **A page carries `email from` but not `email inject`** → treat it as no claim.
  `email inject` remains the single gate on what may be written to.

## What is waived

A pinned sender's mail **skips these entirely**:

| Normally | For pinned senders |
|---|---|
| Pre-filter ([02](./02-prefilter.md)) | Skipped — but see the bounce exception below |
| Quality gate ([03](./03-classify.md)) | Skipped — treated as `USEFUL`, never labelled `PROMOTIONAL` / `LOW_VALUE` / `UNCERTAIN` |
| Six-month test ([index.md](../index.md)) | Waived — do not ask whether the user would want this in six months; they already said yes by pinning the sender |
| Conservative filtering ([index.md](../index.md)) | Waived — "when in doubt, skip" does **not** apply |
| High-confidence routing match ([05](./05-route-and-write.md)) | Waived — the destination declares itself via `email from`, so there is nothing to infer |

Newsletter-shaped content is **expected** here. The fact that a pinned email
reads like a newsletter, a creator update, or a promotional-looking digest is
not a reason to skip it. That is the entire point of pinning the sender.

## What still applies

| Stage | Why it stays |
|---|---|
| Non-delivery reports ([02](./02-prefilter.md)) | A bounce is still a bounce. Discard only genuine `mailer-daemon@` / `postmaster@` / NDR messages *about* the pinned sender. Never discard mail *from* the pinned address itself. |
| Deep read ([01](./01-fetch.md)) | Always fetch the full body. Previews truncate the durable content. |
| Attachments ([04](./04-attachments.md)) | Open and extract as normal. |
| Integrate, don't dump ([05](./05-route-and-write.md)) | Still write structured knowledge into the existing page — never paste the whole email. |
| Deduplication ([05](./05-route-and-write.md)) | Still applies. The run is scheduled and repeats; a duplicate is not a new fact. |

## Routing is a lookup, not a judgment

For each pinned email:

1. Search the `email inject` pages for one whose `email from` marker lists the
   sender address.
2. **Exactly one claim** → deep-read, dedup, integrate. Write it.
3. **No claim** → skip, and report under *Pinned senders with no destination*
   (see [06-report.md](./06-report.md)) so the user can add the marker to a page.
4. **Multiple claims** → do not write; report under `Errors`.

Do not substitute a page because it seems topically related — a page that has not
claimed the sender is not a destination, however relevant it looks. Do not
decline to write because the match feels weak; a claim is a claim, and there is
no confidence judgment to make.
