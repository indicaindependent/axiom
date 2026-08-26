# FAILURE MODES

### Designing for the classifier being unavailable

Most moderation systems have one answer to "what if the model is down?" — nothing
gets moderated, and nobody finds out until later.

Axiom treats inference failure as an expected operating condition rather than an
incident.

## THE LADDER OF DEGRADATION

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/indicaindependent/axiom/main/assets/diagrams/degradation-ladder-dark.svg">
  <img src="https://raw.githubusercontent.com/indicaindependent/axiom/main/assets/diagrams/degradation-ladder-light.svg" alt="Ladder of degradation: primary model unavailable, fallback model classifies, losing some accuracy. Both models unavailable, deterministic backstop, losing nuance while unambiguous cases are still caught. Total inference outage, evaluation queues for retry, losing immediacy but not coverage. Worker unavailable, gateway session persists, losing interaction handling until it returns. Gateway disconnect, worker still serves interactions, losing live evaluation until the session resumes. Both unavailable, evidence and ladders intact in the database, losing live moderation until restored." width="100%">
</picture>

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/indicaindependent/axiom/main/assets/diagrams/defence-in-depth-dark.svg">
  <img src="https://raw.githubusercontent.com/indicaindependent/axiom/main/assets/diagrams/defence-in-depth-light.svg" alt="Four independent moderation layers and what survives when each fails. Layer one, primary classifier: language-model judgement on message content; on failure a configured fallback model takes over. Layer two, vision classifier with its own fallback; on failure image coverage degrades while text coverage is unaffected. Layer three, deterministic backstop: rules-based enforcement requiring no AI, so only a total system failure removes it. Layer four, outage retry queue: re-processes messages missed while classifiers were down. No thresholds or confidence values shown." width="100%">
</picture>

*The table below is the accessible equivalent of this figure, not a caption for it.*


| Failure | What still works | What is lost |
| :--- | :--- | :--- |
| Primary model unavailable | fallback model classifies | some accuracy |
| Both models unavailable | deterministic backstop | nuance; unambiguous cases still caught |
| Total inference outage | evaluation queues for retry | immediacy, not coverage |
| Worker unavailable | gateway session persists | interaction handling until it returns |
| Gateway disconnect | worker still serves interactions | live evaluation until the session resumes |
| Both unavailable | evidence and ladders intact in the database | live moderation until restored |

The important row is the third. **An inference outage costs immediacy rather than
coverage**, because anything the classifier could not evaluate is recorded as
unevaluated and swept afterwards.

## THE RETRY QUEUE

A recurring out-of-band job re-evaluates anything that could not be classified while
models were unavailable.

Two design notes matter more than the mechanism:

**It runs on separate infrastructure from the worker.** A worker-side retry cannot
run when the worker is the thing that is unavailable.

**An empty queue is the healthy state.** Under normal operation the job reports
nothing pending. That is success rather than idleness — worth knowing before someone
reads its log and concludes it never runs.

Its cadence is not published. See the note on omitted timings in
[ARCHITECTURE](ARCHITECTURE.md).

## FAIL-CLOSED GUARDS

Several paths fail closed rather than proceeding on incomplete information. The
governing principle:

> **Fail closed on enforcement. Fail open on speech.**

An error in the evaluation path must not silently allow content through — it queues.
But an error in a detector must not silently punish a member either. Those two
requirements point in opposite directions, so each site resolves them explicitly
rather than inheriting one global default.

## ENGINEERING DISCIPLINE THIS PROJECT APPLIES

The failure modes that cost the most are not crashes. They are instruments that
return a plausible, well-formed value for something they never measured — because
nothing surfaces as an error, and the wrong answer is indistinguishable from a right
one at the point of use.

Five checks are applied throughout as a result:

**1. A wrapped write must be proven to succeed, not merely proven not to crash.**
Instrumentation is wrapped so it can never break the path it observes. But a wrapped
write that cannot succeed under real conditions yields a guard that detects a problem
and records nothing. Wrapped paths are exercised under the conditions they will
actually run in.

**2. An empty result is only evidence once the source is confirmed.** Querying the
wrong store, or with insufficient permission, commonly returns success with nothing
in it — which reads as "this has never happened." Absence is checked against the
right store before it is believed.

**3. A guard must be proven to fire.** Defining a check and invoking it are separate
facts. A source-level search can confirm the first while the second was never wired.

**4. A threshold must sit below the failure it exists to catch.** An alarm calibrated
just above its own trigger condition stays silent through the exact event it was
built for, while manufacturing confidence that something is being watched.

**5. A status code is not a meaning.** A rejection at the network edge can look
exactly like an authentication failure. A success code proves a request was accepted,
never that it did what was intended. State is read back.

## WHAT THIS BUYS

None of the above makes the classifier more accurate. It makes the **system's
behaviour predictable when the classifier is not available** — which, over a long
enough operating window, matters more.