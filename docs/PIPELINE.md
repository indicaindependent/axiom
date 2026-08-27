# PIPELINE

## A MESSAGE, END TO END

```
message arrives on the gateway listener
  |
  |-- should_feed?          watched channel · human author · not a webhook
  |                         not the bot itself · not an exempt account
  |                         (anything else is dropped here and never seen)
  v
forwarded to the engine
  |
  |-- PHASE 1  HARVEST      recorded to the harvest queue before any judgement
  |
  |-- tier lookup           is the author exempt from evaluation?
  |-- scope check           is this channel in moderated scope?
  |
  |-- PHASE 2  JUDGE        ensemble classification, text and image
  |                         falls through fallback models, then the
  |                         deterministic backstop
  |
  |-- category resolver     graduated category and severity assignment
  |-- guard suite           every false-positive guard may CLEAR a verdict
  |-- backstop reconcile    deterministic rules may RAISE a verdict
  |-- confidence gating     first-flag floor · low-confidence nudge path
  |
  |-- EVIDENCE LOCKER       the record is written BEFORE any action fires
  |
  |-- action selection      none · ephemeral nudge · notice · warn
  |                         timeout · remove · escalate
  |-- ladder update         the appropriate strike ladder, by category
  |-- operator log          to the configured log channel
  v
done
```

## THE ORDERING IS THE DESIGN
Four properties of that sequence are load-bearing.
**Harvest precedes judgement.** A message is durably recorded before anything tries to
classify it. That is what makes the retry queue in
[FAILURE-MODES](FAILURE-MODES.md) possible at all — an outage leaves a queue of known
work rather than a silent gap.
**Evidence precedes action.** If the operator notification fails, the record still
exists. A notification is not a record, and a system that acts before it writes can
enforce something it cannot later explain.
**Guards can clear, the backstop can only raise.** Deterministic rules may escalate a
verdict the models scored too low; they may never clear one the models raised. Meanwhile
the false-positive guards may clear. The two directions are deliberately asymmetric:
mechanical certainty is allowed to accuse, and contextual understanding is allowed to
excuse.
**Filtering happens at the earliest possible point.** The listener drops anything
unfeedable before it leaves the process. Nothing downstream can act on a bot message,
a webhook, or a channel outside scope, because those never arrive.

## BACKFILL — RE-JUDGING HISTORY WITHOUT RE-JUDGING EVERYTHING
The engine exposes a bounded, cursor-paginated backfill with **distinct phases**, so
history can be re-examined for one concern at a time rather than wholesale.

| Property | Why it matters |
| :--- | :--- |
| **Phased** | a policy change to one category re-judges only that category |
| **Cursor-paginated** | resumable; a failed run continues instead of restarting |
| **Batched** | each call bounded, so a long backfill cannot become its own incident |
| **Reports remaining** | progress is observable rather than inferred from silence |
| **Driven externally** | the driver runs on operator hardware, not as a worker loop |
Backfill interacts with [GOVERNANCE](GOVERNANCE.md) in a way worth stating: policy
changes are **forward-only**, so backfill is used to recover *missing* verdicts, never
to retroactively re-score messages under a policy that did not exist when they were
posted.

## WHAT IS NOT PUBLISHED
Batch sizes, page limits, freshness cutoffs, the exempt-account set, and the specific
channels in scope. The shape of the pipeline is the useful part; its tuning is not.

