# Contributing to Axiom

Axiom is an autonomous Discord security and moderation system. This repository publishes its
**architecture**, not a deployable copy, and its calibration is withheld on purpose.
Contributions here shape what Axiom does next.

## Two lanes

Some things can be discussed in the open. Some cannot, because documenting a detector is a
manual for evading it.

| Lane | What belongs there | Where |
| :--- | :--- | :--- |
| **Public** | ideas, new commands, the wording of Axiom's messages, documentation, reports that Axiom acted on something it should have left alone | issues and pull requests on this repository |
| **Restricted** | thresholds, guard patterns, classifier prompts, model choices, timings, and any way around Axiom's detection | a private security advisory ([SECURITY.md](SECURITY.md)) |

If a public thread starts heading into the restricted lane, a maintainer will move it and
close the public thread with a note. That is not a rejection.

## How to suggest something

Open an issue and pick a template:

- **Idea**: something Axiom should do, or do differently.
- **Bug**: something that does not work as documented.
- **False positive**: Axiom acted on a message that was legitimate.

False-positive reports are the most valuable thing a member can send. Each confirmed one
becomes a test case: the informative test set for a detector is the legitimate traffic it
must leave alone ([FALSE-POSITIVES.md](../docs/FALSE-POSITIVES.md)).

**Describe, do not paste.** Never include another member's messages, usernames, IDs or
personal details. Describe what happened instead.

## How a change moves

1. **Issue.** Someone suggests it.
2. **Triage.** A maintainer gives it a class and a lane.
3. **Discussion.** Anyone can comment. Reactions help decide what is built first.
4. **Design.** Behaviour changes get a short written design first: the problem, the change,
   who could be wrongly caught, how it is tested, how it is undone.
5. **Pull request.** One focused change, reviewed against the checklist in the pull-request
   template.
6. **Shadow mode.** Behaviour changes run log-only before they act on anyone.
7. **Release.** Recorded the way [GOVERNANCE.md](../docs/GOVERNANCE.md) requires: dated,
   attributed, forward-only. Credited in the [CHANGELOG](../CHANGELOG.md).

## Classes of change

| Class | Examples | Review |
| :--- | :--- | :--- |
| A. Docs and wording | README, help text, the message a member receives | 1 reviewer |
| B. Commands and tools | a new command, a moderator view, a report format | 1 reviewer and the security owner |
| C. Behaviour | anything that changes what Axiom flags, removes, times out or escalates | 2 reviewers and the security owner, then shadow mode |
| D. Calibration and security | tuning, evasion fixes | restricted lane only |
| E. Emergency | Axiom misfiring at scale or being actively evaded | the security owner acts at once; reviewed afterwards |

Votes and reactions set priorities. The security owner makes the final call on security.

## Ground rules

- Never publish a threshold, pattern, prompt, model name, identifier, token or secret, in an
  issue, a pull request, a commit or a comment.
- Every change must keep Axiom's design commitments: fail closed on enforcement and open on
  speech, no single signal decides, evidence before action, the ladder never bans, policy
  changes are forward-only.
- One focused change per pull request. Say what changed and why.
- Be kind. See the [Code of Conduct](CODE_OF_CONDUCT.md).

By contributing you agree your work is MIT-licensed under this project.
