## What this changes

<!-- One focused change. What, and why. -->

## Class

- [ ] A. Docs and wording
- [ ] B. Commands and tools
- [ ] C. Behaviour (needs a design note and shadow mode)
- [ ] E. Emergency (reviewed after the fact)

Class D (calibration and security) never comes through a public pull request.

## Reviewer checklist

Every box must be answered before approval.

- [ ] **Fail closed on enforcement, fail open on speech.** If this breaks, Axiom stops acting rather than acting wrongly.
- [ ] **No single signal decides.** Any new action still needs more than one independent signal.
- [ ] **Evidence before action.** Evidence is saved before anything is removed or anyone is notified.
- [ ] **The ladder never bans.** A human still makes every ban decision.
- [ ] **Forward-only.** Past decisions stay judged by the policy in force at the time.
- [ ] **Tested against legitimate traffic.** There is a set of harmless messages this must leave alone, and it passed.
- [ ] **Nothing secret published.** No thresholds, patterns, prompts, model names, identifiers or tokens.
- [ ] **Undo plan.** The exact way to revert this is written below.
- [ ] **Who could be wrongly caught** is written below.

## Who could be wrongly caught

## How it was tested

## How to undo it

## Credit

<!-- Who suggested it, who built it. Leave blank to stay unnamed. -->
