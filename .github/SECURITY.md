# SECURITY POLICY

## REPORTING
If you believe you have found a security issue in Axiom — including an evasion
technique that defeats its detection — please report it privately rather than
opening a public issue.
Open a GitHub security advisory on this repository, or contact the maintainer
directly. Please include what you observed, how to reproduce it, and what you
expected instead.

## SCOPE
This repository is a **technical specification**, not a deployable implementation.
Reports about the specification are welcome; there is no running service here to
test against.
Particularly valuable:

- a described mechanism that would not work as documented
- an architectural weakness the [threat model](../docs/THREAT-MODEL.md) does not
  already acknowledge
- **anything in this repository that should not have been published** — a value,
  identifier, threshold or pattern that slipped past sanitization
That last category is the most useful report we can receive.

## WHAT IS DELIBERATELY ABSENT
This repository withholds detection calibration on purpose: thresholds, limits,
confidence cut-offs, backstop patterns, classifier prompts and scheduled timings.
**Documentation of a detector is a manual for evading it.** The architecture is
published so the design can be evaluated; the calibration is withheld so it cannot
be circumvented. Requests to publish specific values will be declined.

## ACKNOWLEDGEMENT
Reporters are credited unless they prefer otherwise.
