# GOVERNANCE

### How moderation policy changes are recorded
A moderation system's behaviour is a policy position, not just an implementation.
Axiom treats changes to that position as versioned decisions.

## THE PRACTICE
Every tuning decision that alters enforcement is recorded at the point of change
with three properties:

| Property | Why |
| :--- | :--- |
| **Dated** | so a past decision can be evaluated against what was known then |
| **Attributed** | so there is an accountable author, never an anonymous default |
| **Forward-only** | so a change of policy does not rewrite history |
In practice this appears as dated, attributed annotations at the site of each
decision, in the form:

```
<DATE> — <who approved it>. No retroactive effect: forward-only.
```

## WHY FORWARD-ONLY IS THE LOAD-BEARING PART
If tightening a rule retroactively re-scored past messages, then:

- a member's standing could worsen without them doing anything
- past enforcement decisions would stop being reproducible
- the evidence record would describe a policy that did not exist when the action
  was taken
So a change applies to what happens next. **Prior decisions remain judged by the
policy that was actually in force**, and the evidence record retains the model and
confidence that produced them.

## WHAT REQUIRES A RECORD

- any change to a verdict rule or its ordering
- any change to how signals combine
- **any model swap**, primary or fallback — this is the easiest change to make and
  the easiest to forget, because it alters behaviour with no code diff at all
- any change to the rule text the classifier reasons against
- any threshold change

## WHY THIS IS UNUSUAL AND WHY IT MATTERS
Most bots have a commit history. A commit history tells you *what changed*. It does
not tell you *who decided*, *when the decision took effect*, or whether it applied
backwards.
For a system that takes action against people, those three are the difference
between an auditable decision and an unexplained one. When a member asks why they
were actioned, the answer should be a dated policy and a retained confidence score
— not a shrug and a diff.

## THE UNDERLYING PRINCIPLE

> A thing asserted is not a thing checked.
An enforcement claim is only as good as the record behind it. Governance is what
makes the record answer questions after the fact instead of merely existing.
