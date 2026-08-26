# FALSE POSITIVES

### The design document most worth reading
Most people who dislike moderation bots do not dislike moderation. They dislike
being flagged for a word.

## THE PROBLEM WITH SINGLE-SIGNAL MATCHING
A naive filter escalates when it sees a signal. Mentions a payment app? Flag.
Contains a link? Flag. Mentions a price? Flag.
In a technical community that is catastrophic, because **the vocabulary of
solicitation is also the vocabulary of ordinary shop talk**:

> "the API costs about the same as hosting it yourself"
> "we moved off WhatsApp for support, the webhooks were unreliable"
> "Telegram's bot API is nicer than Discord's for this"
> "that provider charges per seat, which killed it for us"
Every one of those trips a keyword filter. Every one is exactly the conversation the
community exists to have. A system that punishes them does not make the room safer;
it makes the room quieter, and it teaches members that the bot is an obstacle.

## THE REPLACEMENT: CONJUNCTION GATING
Axiom's escalation rules require **co-occurrence**, not presence.
A contact-channel mention is not evidence of solicitation. A contact-channel
mention **together with** selling intent or an outbound link is. Expressed as
structure rather than values:

```
escalate  <-  funnel_signal  AND  (link OR promotional_intent OR earnings_framing)
escalate  <-  earnings_hook  AND  (funnel OR promotional_intent OR invite_behaviour)
```
The single signal, alone, does nothing.
This is why someone can say "we pay for the API" without consequence, while
"DM me on Telegram, I'll show you the API that made me money" resolves immediately.
Same vocabulary. Different structure. **The structure is the signal.**

## RATE AND STRUCTURE ARE LOAD-BEARING; PHRASING IS CORROBORATION
A parallel rule applies to behavioural detection. Volume and burst are primary
evidence. Wording is supporting evidence only, never sufficient on its own.
The reason is concrete: a phrase common among spammers is also used by ordinary
members, so a phrasing gate alone flags real people. Any detection design that can
be defeated or triggered by word choice is measuring vocabulary and calling it
intent.

## SUBSTRING MATCHING IS A FALSE-POSITIVE GENERATOR

A specific, recurring implementation trap, documented because it is easy to
reintroduce.
Matching a fragment inside a longer word produces confident nonsense. A term of art
can contain a flagged sequence entirely by accident, and the match will look like a
real hit in a log. Pattern boundaries must be anchored, and any pattern list needs
testing against ordinary domain vocabulary — not only against known-bad examples.
**Testing a detector only on things it should catch measures nothing.** The
informative test set is the legitimate traffic it must leave alone.

## THE ASYMMETRY THAT DRIVES ALL OF THIS

| Error | Who notices | Cost |
| :--- | :--- | :--- |
| False negative | moderators, later | one message survives longer than it should |
| False positive | the member, immediately, in public | trust in the system, and their willingness to post |
A missed violation is a delay. A wrong flag is a relationship.
That asymmetry is why corrections are **ephemeral by default** — a private nudge, so
that being wrong costs the member nothing publicly — and why borderline promotional
cases are **redirected rather than punished**.

## WHAT THIS DOES NOT SOLVE
Conjunction gating raises the cost of evasion. It does not eliminate it. An
adversary who understands the structure can construct a message that satisfies no
conjunction — which is precisely why the numeric calibration behind these rules is
not published, and why the deterministic backstop's patterns are not enumerated
anywhere in this repository.
