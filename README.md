# AXIOM

> [!NOTE]
> This repository publishes the **architecture** of a system running in production. It is a
> specification, not a deployable copy. The calibration is deliberately withheld — no
> thresholds, confidence cut-offs, guard patterns, classifier prompts or schedules — because
> documentation of a detector is a manual for evading it. Identifiers are placeholdered.


### Autonomous Discord security and channel moderation
Axiom reads a live Discord gateway, classifies what it sees across text and images,
and acts. Plenty of bots do that.
Two things separate this one, and neither is the classifier.

---

## 1. IT IS BUILT NOT TO PUNISH THE INNOCENT
Most people who dislike moderation bots do not dislike moderation. They dislike being
flagged for a word.
In a technical community that failure is near-guaranteed, because the vocabulary of
solicitation is also the vocabulary of ordinary shop talk. "The API costs about the
same as hosting it" is not an advertisement. "We moved off WhatsApp for support" is not
a funnel. A keyword filter cannot tell the difference, so it punishes the conversation
the community exists to have.
Axiom carries a suite of named, individually-engineered guards whose only purpose is
declining to act:

| Guard | What it protects |
| :--- | :--- |
| **Technical-context guard** | security and pentest vocabulary is not adult content |
| **Reported-speech guard** | quoting abuse in order to report it is not abuse |
| **Hiring guard** | a job post is not an advertisement |
| **Free-software guard** | sharing something free is not promotion |
| **Exemption gate** | declared exemptions are honoured |
| **Deterministic guards** | a whole non-AI layer dedicated to this alone |
| **First-flag confidence floor** | a first offence requires *higher* confidence than a repeat |
| **Low-confidence nudge** | uncertainty produces a private nudge, never a punishment |
Escalation requires **co-occurrence**, never presence. A contact-channel mention alone
does nothing; a contact-channel mention *together with* selling intent resolves
immediately. Same vocabulary, different structure — and the structure is the signal.
The reported-speech distinction is one that human moderation teams routinely get
wrong. See [FALSE-POSITIVES](docs/FALSE-POSITIVES.md).

## 2. IT ASSUMES ITS OWN AI WILL FAIL

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/indicaindependent/axiom/main/assets/diagrams/defence-in-depth-dark.svg">
  <img src="https://raw.githubusercontent.com/indicaindependent/axiom/main/assets/diagrams/defence-in-depth-light.svg" alt="Four independent layers. Primary classifier with a configured fallback. Vision classifier with its own fallback, so image failure does not affect text coverage. Deterministic backstop requiring no AI at all. Outage retry queue that re-processes anything missed while classifiers were down." width="100%">
</picture>


| | Layer | Survives |
| :--- | :--- | :--- |
| **1** | model ensemble — several text models, two independent vision models | normal operation |
| **2** | configurable fallback models | a primary being unavailable |
| **3** | deterministic backstop, **no AI at all** | total inference outage |
| **4** | retry queue on separate hardware | gaps left while the above were down |
Layer 3 means coverage *degrades* instead of disappearing. Layer 4 exists because a
worker-side retry cannot run when the worker is the thing that is broken.
Both were built in response to measured failures, and both failures are published in
full in [FAILURE-MODES](docs/FAILURE-MODES.md) — including the outage that left ten
messages permanently unjudged, and the health check that made a dead process invisible
for 112 restarts. Those two documents are the ones worth your time.

---

## COMPONENTS

```
moderation engine    resident Discord gateway + scheduled worker + two databases
research gateway     authenticated per-consumer research API over a corpus store
security scanner     read-only web-security audit, invoked from chat
```
The gateway listener runs as a resident process on operator hardware, because a Worker
cannot hold a resumable websocket across cold starts. Its own description of itself:

> The brain does all judgment; this feeder is dumb plumbing. Auto-reconnect, resume,
> backoff. **Never acts, never stores.**

## DOCUMENTATION

| | Document |
| :--- | :--- |
| | [FALSE-POSITIVES](docs/FALSE-POSITIVES.md) — the guard suite, and why single-signal matching fails |
| | [FAILURE-MODES](docs/FAILURE-MODES.md) — two real failures, published in full |
| | [ARCHITECTURE](docs/ARCHITECTURE.md) — components, data flow, the two-database split |
| | [PIPELINE](docs/PIPELINE.md) — harvest, judge, evidence, act — and backfill |
| | [CLASSIFICATION](docs/CLASSIFICATION.md) — the ensemble and the verdict taxonomy |
| | [THREAT-MODEL](docs/THREAT-MODEL.md) — scope, non-scope, acknowledged trade-offs |
| | [TRUST-TIERS](docs/TRUST-TIERS.md) — authority derived from Discord permission bits |
| | [COMMANDS](docs/COMMANDS.md) — the command surface by required tier |
| | [CONFIGURATION](docs/CONFIGURATION.md) — operator-tunable behaviour |
| | [GOVERNANCE](docs/GOVERNANCE.md) — dated, attributed, forward-only policy changes |
| | [PRIVACY](docs/PRIVACY.md) — what is retained, and who can reach it |
| | [SCANNER](docs/SCANNER.md) — the companion security scanner |
| | [RESEARCH-GATEWAY](docs/RESEARCH-GATEWAY.md) — the corpus and research API |

---

## ON THIS REPOSITORY
**This is a technical specification, not a deployable implementation** — and for a
detector that is a security requirement rather than a convenience.
Documentation of a detector is a manual for evading it. So this repository
**publishes the architecture and withholds the calibration**: no thresholds, limits,
confidence floors, windows, model identifiers, guard patterns, prompts or scheduled
cadences. No real identifiers of any kind. Every example is synthetic.
You can judge exactly how it is built and why it works. You cannot derive what you
would need to slip past it.

## LICENSE
See [LICENSE](LICENSE).
