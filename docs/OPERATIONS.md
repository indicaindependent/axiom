# Operations: the scheduled tick, queues, and self-checks

Axiom does its slow work on a scheduled tick. This document describes what the tick does,
in what order, and the mechanisms that keep it honest — most of which exist because of a
specific failure described in the [changelog](../CHANGELOG.md).

## The scheduled tick

Order matters. The tick runs:

1. **Wipe queue** — drain approved message-wipe jobs, one account per tick, evidence
   locker first, failing closed per channel. Guarded by a lease lock with a dead-man expiry.
2. **Roster sync** — refresh the member roster from the gateway, writing in fixed-size
   batches, then reconcile who has left.
3. **Queue-age tripwire** — see below.
4. **Weekly dry replay** — on its weekly slot only.

The wipe step runs first because the roster sync is the expensive stage. When it ran
first and wrote one row per member, it exhausted the invocation's subrequest budget on
large servers and the wipe step was never reached; approved jobs sat for more than a day
with zero attempts, and — because the worker had no persisted logs — nothing said so.

## Batched writes

The roster sync submits its upserts as database batches rather than one statement per
member. Same SQL, same result, roughly two orders of magnitude fewer subrequests. Any
loop that writes per-row on a large collection is a candidate for the same starvation and
should be batched.

## The wipe queue

- A ban-card tap bans immediately with a zero-second history purge (so the platform never
  deletes anything before it is archived) and enqueues a guild-wide wipe job.
- The tap also drains a small number of jobs immediately in the background so the operator
  sees progress within seconds; the tick finishes the rest.
- Each job archives every message it will remove **before** removing any, per channel,
  and aborts that channel if the archive fails.
- Completion produces one operator notification per account.
- An operator endpoint drains one job on demand and can report queue state read-only.

## Queue-age tripwire

An approved job that remains untouched past an age threshold means something upstream is
starving the queue again. The tripwire notifies the operator **once per stale set** and
re-arms after a quiet period, so a stalled queue produces one message, not one per tick.

It was proven with a control before it went live: an artificially aged job was inserted,
the check reported it with its age, the job was removed, and the check returned clear.
A safety check that has never been shown to fire is not a safety check.

## Lease lock

Concurrent runners of the wipe tick (a scheduled tick and an on-demand drain overlapping)
could process the same job twice. The tick takes a short lease before draining and
releases it in a `finally`; a second runner finding the lease held returns immediately
and reports that it was locked.

## Logging

The reference deployment runs with persisted structured logs enabled. Each tick stage
logs one line with its counts (`ok`, totals, pages, leavers flagged). The absence of logs
is a defect in itself: a day-long stall is invisible without them.

## Weekly dry replay

Once a week the deterministic detectors are re-run over the previous week's general-channel
rows that were **not** actioned, and the operator receives a summary: how many rows were
scanned, how many would fire today, by species, with a short sample. Nothing is actioned,
no strikes are written, and the cross-account template memory runs in dry mode so the
replay cannot pollute it.

This exists because the most dangerous failure in a moderation pipeline is the quiet one:
a detector that fires and is then erased by a downstream guard, so the message is graded
clean and no human is told. The replay makes that class of miss visible within a week.
The same endpoint is available to the operator on demand.

## Operator endpoints

All operator endpoints are bearer-secret gated and are called from an operator-controlled
host, never from a browser. They return structured JSON, refuse on a wrong key, and — where
they delete — go through the evidence locker and fail closed exactly like the automated
paths. A targeted-delete route that bypassed the locker was found and corrected; there is
no longer any delete path in Axiom that does not archive first.
