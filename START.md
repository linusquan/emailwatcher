# Start Instructions

To kick off a run, prompt the automation with:

```text
Read README.md in this repo. Load run parameters from emailwatcher.config
(timerange, inboxes, pinned_sender), then read index.md and every file under rules/ for the
full processing/classification/routing rules. index.md is only a spine — the
rules/ files are mandatory, not optional reference. Then process the
configured inboxes for the configured lookback window and report results per
the Completion Report format in rules/06-report.md.
```

The automation must not skip step 1 — settings and rules always come from
these files, never from defaults baked into the prompt. A run that loads
`index.md` but not the six `rules/` files is missing most of the spec.
