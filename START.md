# Start Instructions

To kick off a run, prompt the automation with:

```text
Read README.md in this repo. Load run parameters from emailwatcher.config
(timerange, inboxes) and the full processing/classification/routing rules
from Emailer-Agent.md. Then process the configured inboxes for the
configured lookback window and report results per the Completion Report
format in Emailer-Agent.md.
```

The automation must not skip step 1 — settings and rules always come from
these files, never from defaults baked into the prompt.
