# AXIOM

### Autonomous Discord security and channel moderation, built on the assumption that its own AI will fail
Axiom reads a live Discord gateway, classifies what it sees, and acts — warning,
timing out, escalating or staying silent. That part is unremarkable; plenty of bots
classify messages.
What makes Axiom different is the **four independent layers underneath the
classifier**, each able to carry the system when the one above it stops working.

---

## THE FOUR LAYERS

| | Layer | What it does | What it survives |
| :--- | :--- | :--- | :--- |
| **1** | Primary classifier | Workers AI model, confidence-scored, category-tagged | normal operation |
| **2** | Fallback classifier | a second configurable model | the primary model being unavailable |
| **3** | Deterministic backstop | rules-based, **no AI involved at all** | total inference outage |
| **4** | Retry queue | out-of-band sweep on separate hardware | gaps left while the above were down |
Layer 3 is the unusual one: when every model is unreachable, coverage **degrades**
instead of disappearing. Layer 4 is rarer still — it runs outside the worker, on a
different machine, and its only job is to close the holes an outage left behind.

> A moderation system that assumes its classifier always answers is a moderation
> system with a single point of failure it has not noticed.

---

## WHAT IT ACTUALLY DOES

- **Text classification** with confidence scores and named categories
- **Image classification** through a separate vision model, with its own fallback
- **Graduated verdicts** rather than allow/deny — see [CLASSIFICATION](docs/CLASSIFICATION.md)
- **Conjunction-gated signals** so legitimate conversation is not punished for
  vocabulary — see [FALSE-POSITIVES](docs/FALSE-POSITIVES.md), which is the design
  document most worth reading
- **Raid detection** on join behaviour and account age
- **Spam windowing** with configurable limits
- **Two separate strike ladders**, so a promotional infraction does not accumulate
  against a harassment record
- **Evidence retention**, so any action can be audited after the fact
- **Ephemeral-first correction** — the default is a private nudge, not public shaming
- **Trust tiers** derived from real Discord permission bits
- **Weekly security brief**

---

## DOCUMENTATION

| Document | What it covers |
| :--- | :--- |
| [ARCHITECTURE](docs/ARCHITECTURE.md) | components, data flow, the two-database split |
| [THREAT-MODEL](docs/THREAT-MODEL.md) | what it defends against, and what it deliberately does not |
| [CLASSIFICATION](docs/CLASSIFICATION.md) | the verdict taxonomy and the borderline rule |
| [FALSE-POSITIVES](docs/FALSE-POSITIVES.md) | why single-signal matching fails, and what replaces it |
| [FAILURE-MODES](docs/FAILURE-MODES.md) | what happens when each layer breaks |
| [TRUST-TIERS](docs/TRUST-TIERS.md) | who can invoke what |
| [COMMANDS](docs/COMMANDS.md) | the command surface, by required tier |
| [CONFIGURATION](docs/CONFIGURATION.md) | operator-tunable behaviour |
| [GOVERNANCE](docs/GOVERNANCE.md) | how moderation policy changes are recorded |
| [PRIVACY](docs/PRIVACY.md) | what is retained, for how long, and who can read it |

---

## ON THIS REPOSITORY
**This is a technical specification, not a deployable implementation.**
That is deliberate, and for a moderation system it is also a security requirement.
Documentation of a detector is a manual for evading it, so this repository
**publishes the architecture and withholds the calibration**:

- every threshold, limit, window and confidence cut-off is described by its
  *purpose*, never its value
- the deterministic backstop's trigger patterns are described by their *role*,
  never enumerated
- classifier prompts are not published
- all examples are synthetic
You can see exactly how it is built and why it works. You cannot derive the numbers
you would need to slip past it. Both of those are on purpose.

## LICENSE
See [LICENSE](LICENSE).
