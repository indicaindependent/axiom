# FAILURE MODES

### Two real failures, published in full
Most projects document what they do. This document covers two things that went wrong in
production, because the fixes are the most interesting engineering in the system and
because a security tool that only publishes its successes has not earned trust.

---

## FAILURE ONE — THE HEALTH CHECK THAT BLINDED THE SUPERVISOR
**112 unnecessary restarts in 24 hours, and a real fault made invisible.**

### The setup
The gateway listener runs as a resident process, supervised by a keeper that restarts it
if it looks dead. The keeper's liveness test was: *has the listener's log gone stale?*
That test rested on an assumption — that the listener emitted a heartbeat line on a
fixed interval.

### What actually happened
The heartbeat had been **removed** from the listener days earlier, because it was itself
causing a reconnect churn loop. The keeper was left depending on a signal that no longer
existed.
The listener writes to its log only on startup and when it flags something. So:

```
quiet channel
  -> log goes stale
    -> keeper concludes the process is dead
      -> keeper restarts it
        -> the restart itself writes fresh log lines
          -> log looks healthy again
            -> twenty minutes later, identical. Forever.
```

### The part that made it serious
Not the wasted restarts. This, from the fix's own header:

> "Worse, a REAL zombie became indistinguishable from that noise, so the supervisor was
> effectively blind."
A genuinely hung process produces exactly the same symptom as a quiet channel. Once
restarts became routine, the one signal that could have revealed a real fault had no
information left in it. **The monitor was not merely useless; it was actively
concealing.**

### The fix, which touches the listener zero times
The replacement is a **black-box check that asks a question about the world** rather
than about the process:

> Does Discord hold a **feedable** message newer than our newest moderation record?
>
> - quiet channel — healthy, however long it stays quiet
> - unprocessed human traffic — zombie
This cannot be fooled by a restart, because a restart does not create moderation
records. It cannot be fooled by silence, because silence is now a valid healthy state.
And "feedable" mirrors the listener's own filter exactly — watched channel, human
author, not a webhook, not the bot itself, not an exempt account. That detail is
load-bearing: other bots post periodically, so **counting bot messages would
manufacture a permanent false gap and rebuild the exact loop this replaces.**

### The generalisable lesson
**An instrument that measures the wrong thing is worse than no instrument, because it
hides real failure inside its own noise.**
Three corollaries applied throughout this system:

1. A monitor must be proven to distinguish the failure it exists to catch from normal
   operation. Proving it does not crash is not the same thing.
2. A monitor that depends on the monitored process emitting something is coupled to it,
   and that coupling silently rots when the process changes.
3. Prefer a check whose evidence is produced by neither the monitor nor the monitored
   process. Here, Discord and the moderation record are both third parties.

---

## FAILURE TWO — THE PERMANENT HOLE, AND A REFUSAL TO PUNISH RETROACTIVELY

### The setup
When the classifier is unreachable, the engine fails **closed**: the message is recorded
as unjudged, archived with an explicit unjudged marker, and no action is taken.
That is the correct first half. A moderation system must never silently allow content
through because its own inference failed.

### What was missing
Nothing ever looked at those records again.

> "A transient model outage created a **permanent** unmoderated hole. Ten real messages
> accumulated that way, including an unauthorised @everyone recruiting for an external
> hackathon that no layer ever looked at."
Fail-closed without a retry is not fail-closed. It is **deferred failure with a paper
trail** — the system records precisely what it failed to do and then never does it.

### The fix, and the decision inside it
A retry queue runs out-of-band, on separate hardware from the worker, and distinguishes
two cases:

| Age of the unjudged message | Treatment |
| :--- | :--- |
| **fresh** | re-submit normally — full enforcement is still appropriate |
| **stale** | re-submit in **observe-only** mode: a verdict is *recorded*, no punishment fires |
The reasoning, verbatim from the implementation:

> "Retroactive timeouts days later are arbitrary; the missing verdict is the actual bug."
**That is an ethical decision encoded in code.** The system had a genuine gap. Closing
it by punishing people days after the fact would fix the metric and not the problem —
the member's experience would be an unexplained sanction for something long forgotten,
caused by an outage that was never their fault. So the verdict is recovered for the
record, and the punishment is deliberately forfeited.

### Three properties that make it safe

- **Idempotent.** Rows are marked once handled, so nothing is retried forever.
- **Self-healing without thrash.** If the classifier is *still* unreachable, the row is
  left untouched for the next cycle rather than being consumed and lost.
- **Bounded.** Each run processes a capped number of rows, so recovery from a long
  outage cannot become its own incident.

---

## THE LADDER OF DEGRADATION

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/indicaindependent/axiom/main/assets/diagrams/degradation-ladder-dark.svg">
  <img src="https://raw.githubusercontent.com/indicaindependent/axiom/main/assets/diagrams/degradation-ladder-light.svg" alt="Ladder of degradation: primary model unavailable, fallback model classifies, losing some accuracy. Both models unavailable, deterministic backstop, losing nuance while unambiguous cases are still caught. Total inference outage, evaluation queues for retry, losing immediacy but not coverage. Worker unavailable, gateway session persists. Gateway disconnect, worker still serves interactions. Both unavailable, evidence and ladders intact in the database." width="100%">
</picture>

What the two fixes buy, stated as a table:

| Failure | What still works | What is lost |
| :--- | :--- | :--- |
| A primary model unavailable | fallback model classifies | some accuracy |
| All models unavailable | deterministic backstop | nuance; obvious cases still caught |
| Total inference outage | messages queue, retry sweeps them | immediacy, **not** coverage |
| Worker unavailable | resident gateway session persists | interaction handling until it returns |
| Gateway disconnected | worker still serves interactions | live evaluation until resume |
| Both unavailable | evidence and ladders intact in the database | live moderation until restored |
| Supervisor misjudges liveness | external check disagrees with it | nothing — that is the point |

## FAIL CLOSED ON ENFORCEMENT, FAIL OPEN ON SPEECH
The governing principle, and the reason the two halves of failure one and two point in
opposite directions:

- An error in the **evaluation** path must not silently allow content through. It
  queues.
- An error in a **detector** must not silently punish a member. It stands down.
Those requirements conflict, so each site resolves them explicitly rather than
inheriting a single global default.

## WHAT NONE OF THIS ACHIEVES
None of it makes the classifier more accurate. It makes the system's behaviour
**predictable when the classifier is not available** — which, over a long enough
operating window, matters more than peak accuracy.

