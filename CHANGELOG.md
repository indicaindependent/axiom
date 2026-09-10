# Changelog

All notable changes to the Axiom specification. Dates are the day the behaviour went live
on the reference deployment. Entries describe mechanisms, never tuning values.

## 2026-09-09 — Smooth-run batch: the queue that starved, and the logs that did not exist

### Fixed
- **Wipe-queue starvation.** Approved message-wipe jobs sat untouched for more than a day
  with zero attempts. Root cause: the scheduled tick ran the full member-roster sync first,
  one database write per member, and exhausted the per-invocation subrequest budget before
  the wipe step was reached. The roster sync now writes in fixed-size batches, cutting its
  subrequest count by roughly two orders of magnitude, and the wipe step runs *before* the
  sync on every tick. See [OPERATIONS.md](docs/OPERATIONS.md#the-scheduled-tick).
- **Roster reconcile never completing.** A side effect of the same starvation: the
  "who has left the server" reconcile at the end of the sync had not been reached in weeks.
  The first completed tick after the fix flagged a backlog of leavers at once.
- **A detector whose finding was erased by a later guard.** The cross-account template
  detector fired on a two-account recruiting script, then a hiring carve-out downstream
  nulled the classification, so the escalation hold was never reached and no human was
  told. Scam-signal holds now short-circuit ahead of every carve-out. Escalate-only: the
  hold saves evidence and notifies the operator; it never enforces.
- **A tap handler that stripped its own buttons.** Multi-account ban proposals carried a
  different field shape from single-account ones; the guard rejected them as "no longer
  available" and removed the buttons, so nothing was ever banned from a list card.
- **Admin delete route bypassed the evidence locker.** The operator's targeted-delete
  endpoint deleted without an archive receipt, contradicting the locker law. It now
  fetches the message, archives it, and refuses to delete on any locker failure.

### Added
- **Operator-approved ban cards** with an immediate ban on tap and a *queued* history wipe
  that the scheduled tick drains one account per tick, evidence locker first, failing
  closed per channel. The tap itself now also drains a small number of wipe jobs
  immediately in the background so the operator sees movement within seconds.
- **Queue-age tripwire.** An approved wipe job that remains untouched past an age threshold
  produces a single operator notification per stale set, re-arming after a quiet period.
  Proven with a control: an artificially aged job was flagged, then the check returned
  clear once it was removed.
- **Lease lock on the wipe tick** so concurrent runners cannot double-process a job;
  the lock carries a dead-man expiry and is released in a `finally`.
- **Structured logging enabled** on the reference deployment. Before this the worker had
  no persisted logs at all, which is why a day-long stall was invisible. The scheduled tick
  now logs one line per stage with counts.
- **Redirect ladder for two deterministic solicitation species in the general channel** —
  recruiting-plus-DM-funnel and off-platform commercial adverts. First offence: archive,
  delete, private redirect note, no strike. Second: the same plus a short timeout. Third
  and beyond: a longer timeout and an operator ping for the ban decision. Staff exempt;
  general channel only; fails closed on the locker. This is the operator's explicit,
  documented exception to the "deterministic rules escalate only" rule — see
  [ENFORCEMENT-LADDER.md](docs/ENFORCEMENT-LADDER.md).
- **Same-author window.** A funnel split across two messages ("looking for a partner…"
  then, later, "let's talk in DM") evades any single-message rule. When one message
  carries only a partial signal, the author's recent messages in the channel are joined
  and re-detected; on a hit every joined message is archived, then deleted, as one offence.
- **Begging species** (personal money, gift or purchase requests aimed at members).
  Requires two independent signals — an *ask* phrase **and** a money-or-channel term — so
  builder talk that shares vocabulary ("anyone feeling generous with code review?",
  "I can't afford the pro plan so I use the free tier") does not trigger it.
- **Weekly dry replay.** Once a week the deterministic detectors are re-run over the
  previous week's un-actioned general-channel rows and the operator receives a summary of
  what *would* fire today. Nothing is actioned. First run on real data: every known missed
  spam message was recovered and no legitimate message was flagged.
- **Scam lexicon and cross-account template memory** (escalate-only). The same normalised
  script appearing from a second account is a signal even when the classifier grades it
  clean; a small lexicon of resale/proxy/"training provided, DM me" markers corroborates.

### Changed
- Wipe step ordering in the scheduled tick (wipes first, then roster sync, then tripwires).
- Ban-card tap: fires the ban with zero-second Discord history purge so nothing is deleted
  before it is archived; the archive-then-wipe happens through the queue.

## 2026-09-04 — Evidence locker law
- Every deletion path archives message content and attachments to object storage first
  and **fails closed** — no receipt, no delete, operator notified instead.
- Prior-reversal guard: a member with a prior overturned action is never auto-actioned by
  the advert layer again. A clean classifier grade can no longer be overridden by a regex.
