# CLASSIFICATION

## AN ENSEMBLE, NOT A CLASSIFIER
Classification runs across **several independent models spanning two modalities**:

| Role | Coverage |
| :--- | :--- |
| Text | multiple instruct-tuned models of differing size and architecture, including a mixture-of-experts model and a large dense model |
| Vision | **two independent image models**, so image classification has its own redundancy rather than borrowing the text path's |
| Escalation | an external reasoning call for genuinely hard cases |
| Fallback | every primary has a configured alternate |
Two consequences worth naming:
**Image moderation is a first-class path, not a bolt-on.** A violation placed in an
image is not a cheap way past the system, because the vision layer has its own primary
and its own fallback.
**Model choice is configuration.** Swapping one is a behavioural change that produces no
code diff at all, which is exactly why [GOVERNANCE](GOVERNANCE.md) requires a record for
it.
Model identifiers are deliberately not published. Knowing which model judges a
message is directly useful to someone trying to write beneath it.

## VERDICTS ARE GRADUATED
Most systems answer allow-or-deny, which forces every ambiguous case into one of two
wrong outcomes. The promotional path here resolves to three states:

| Verdict | Meaning | Typical response |
| :--- | :--- | :--- |
| `hard` | unambiguous commercial solicitation | removal, ladder increment, operator log |
| `soft` | promotional in character but plausibly in good faith | redirect to the right channel, private note |
| clean | no promotional character | no action |
The middle state is the entire point. It lets the system act on something without
treating a member as a spammer.

## THE BORDERLINE FLOOR

> **A message that is promotional but borderline floors at `soft`. It may never fall
> through to clean.**
An explicit, dated policy decision rather than emergent behaviour. It encodes a
judgement about which error is cheaper:

- A borderline case wrongly **cleared** teaches every observer the rule is optional, and
  the next message is bolder.
- A borderline case wrongly **redirected** costs one member one moment of mild friction,
  in private, with an explanation.
The second is recoverable. The first compounds.

## SIGNAL FAMILIES
Described by role. No patterns, weights or thresholds.

| Family | Observes |
| :--- | :--- |
| commercial intent | whether something is being offered in exchange for money |
| link presence | whether an outbound destination is attached |
| contact-channel mention | whether the message points somewhere private or off-platform |
| audience targeting | whether the message addresses the room rather than a person |
| invite behaviour | whether the message trades in server access |
| earnings framing | whether an outcome is advertised to be emulated |
**No single family escalates alone.** See [FALSE-POSITIVES](FALSE-POSITIVES.md).

## ABUSE CATEGORIES
Abuse-type content is classified separately from the promotional path, with named
categories, independent confidence scoring, and its own strike ladder. The separation is
deliberate: a promotional message is usually *misplaced*, whereas abuse is usually
*intended*, and the appropriate response differs in kind rather than degree.

## CONFIDENCE
Every classification carries a confidence score, retained with the evidence, so a later
review can distinguish a marginal call from an obvious one.
**Thresholds are not published and are not uniform across categories.** A score that
justifies action on one category does not justify it on another. Nor is the first-flag
floor published — see [FALSE-POSITIVES](FALSE-POSITIVES.md) for what it does.

