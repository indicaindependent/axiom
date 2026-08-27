# FALSE POSITIVES

### The design document that matters most
A moderation system is judged by the community it moderates, and communities do not
resent enforcement. They resent being flagged for a word.

## THE ASYMMETRY THAT DRIVES EVERY DECISION HERE

| Error | Who notices | Cost |
| :--- | :--- | :--- |
| Missed violation | moderators, later | one message survives longer than it should |
| Wrong flag | the member, immediately | trust in the system, and their willingness to post |
A missed violation is a delay. A wrong flag is a relationship.
Every guard below exists because that asymmetry was taken seriously rather than
assumed away.

## WHY SINGLE-SIGNAL MATCHING CANNOT WORK
A naive filter escalates on presence. Mentions a payment app, flag. Contains a link,
flag. Mentions a price, flag.
In a technical community that is catastrophic, because the vocabulary of solicitation
is also the vocabulary of ordinary shop talk:

> "the API costs about the same as hosting it yourself"
> "we moved off WhatsApp for support, the webhooks were unreliable"
> "that provider charges per seat, which killed it for us"
Every one trips a keyword filter. Every one is the conversation the community exists
to have. A system that punishes them does not make the room safer — it makes the room
quieter, and teaches members that the bot is an obstacle to be routed around.

## THE REPLACEMENT: CONJUNCTION GATING
Escalation requires **co-occurrence**, not presence:

```
escalate  <-  funnel_signal   AND  (link OR promotional_intent OR earnings_framing)
escalate  <-  earnings_hook   AND  (funnel OR promotional_intent OR invite_behaviour)
```
A single signal, alone, does nothing.
So "we pay for the API" passes without friction, while "DM me on Telegram, I'll show
you the API that made me money" resolves immediately. Same vocabulary. Different
structure. **The structure is the signal.**

## THE GUARD SUITE
Each of these is a named, separately-reasoned component. They are listed by what they
protect rather than how they work.

### Technical-context guard
Security work has a vocabulary that overlaps heavily with content policy: exploit,
penetration, payload, injection. A community that discusses application security
cannot function under a filter that reads those words literally. This guard
establishes technical context before any content judgement is allowed to stand.

### Reported-speech guard
**The most important guard in the system.** A member quoting abuse in order to report
it is not committing abuse. Neither is a moderator discussing a case, or someone
describing what was said to them.
A classifier reading only the text sees the abusive string and flags it. The effect of
getting this wrong is severe and self-reinforcing: **it punishes the person raising the
alarm**, which teaches a community to stop reporting. Most human moderation teams get
this wrong at least once.

### Hiring and collaboration guard
"Looking for a backend dev for a paid project" carries commercial intent, a rate, and
often a contact channel — a perfect score on every solicitation signal, and entirely
legitimate in a builder community. Recruitment is distinguished from advertising as a
first-class category rather than surviving on a lucky threshold.

### Free-software guard
Sharing something free is the opposite of solicitation, yet it looks identical to a
signal counter: a link, a project name, enthusiasm. Absence of a commercial hook is
treated as evidence, not as a missing feature.

### Exemption gate
Some accounts and contexts are legitimately permitted to promote. Exemptions are
explicit and checked, rather than being emergent from thresholds.

### Deterministic guards
An entire non-AI layer dedicated to false-positive suppression, so that during a model
outage the system does not become *more* punitive as it becomes less capable. That
inversion is a common failure in layered designs and it is guarded against directly.

## GRADUATED RESPONSE, NOT A BINARY
Two further mechanisms exist purely to make being wrong cheap:
**First-flag confidence floor.** A member's *first* flag requires **higher** confidence
than a subsequent one. The reasoning is proportionality — a first encounter with the
system sets a member's expectation of it forever, and a wrong first flag costs more
than a wrong second.
**Low-confidence, minor-severity nudge.** Where certainty is low and severity is
minor, the outcome is a private nudge. No strike, no public mark, no record of
punishment. The member simply learns the norm.
Combined with **ephemeral-first delivery** — corrections are private by default — the
cost of a wrong call falls close to zero, which is the only honest way to run a
detector that will sometimes be wrong.

## OFF-TOPIC IS NOT A VIOLATION
Stated explicitly in the design. Being in the wrong channel is a routing problem, and
the correct response is a redirect. Treating misplacement as misconduct is how
moderation systems acquire a reputation for pettiness.

## A RECURRING IMPLEMENTATION TRAP
Matching a fragment inside a longer word produces confident nonsense — a term of art
can contain a flagged sequence entirely by accident, and the match looks like a real
hit in a log. Pattern boundaries must be anchored, and every pattern list needs testing
against ordinary domain vocabulary.
**Testing a detector only against things it should catch measures nothing.** The
informative test set is the legitimate traffic it must leave alone.

## WHAT THIS DOES NOT SOLVE
Conjunction gating and the guard suite raise the cost of evasion. They do not eliminate
it. An adversary who understands the structure can construct a message satisfying no
conjunction.
That is precisely why the calibration behind these rules — the thresholds, the floors,
the guard patterns — appears nowhere in this repository.

