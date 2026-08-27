# SECURITY SCANNER
A read-only web-security audit, invoked from chat. It fetches, inspects and grades — it
never writes, never authenticates against a target, and never attempts exploitation.

## WHAT IT AUDITS

| Audit | Looks at |
| :--- | :--- |
| **Security headers** | transport security, content policy, frame options, content-type handling, referrer policy, permissions policy |
| **Content Security Policy quality** | not merely whether a CSP exists, but whether it is meaningful |
| **Cookies** | flags and attributes on what the target sets |
| **Forms** | how inputs are exposed and submitted |
| **Exposed secrets** | credential-shaped material reachable in served content |
| **Backend fingerprint** | what the target discloses about its own stack |
| **Path probing** | a bounded check for commonly exposed paths |
Output is a letter grade from `A+` down to `F`, so a non-specialist gets an
interpretable answer rather than a header dump.

##  THE SPA-FALLBACK CONTROL PROBE
The most interesting thing in this component, and the reason it belongs in a repository
about detection quality.
Path probing has an obvious failure mode: a single-page application answers **every**
URL with `200 OK` and its shell page. A naive prober therefore reports that every
sensitive path it tried exists — a page of confident false findings.
The scanner sends a **control probe** to a path that cannot plausibly exist. If that
returns success, the target is a catch-all, and every other positive result from path
probing is discarded as meaningless.
**That is a negative control.** It establishes what "not found" looks like on this
specific target before trusting any "found". It is the same discipline as the external
liveness check in [FAILURE-MODES](FAILURE-MODES.md): prove the instrument can register
absence before believing its presence.

## SAFETY PROPERTIES

| Property | Why |
| :--- | :--- |
| **Read-only** | no writes, no auth attempts, no exploitation |
| **Bounded reads** | response bodies are capped, so a hostile or enormous target cannot exhaust the scanner |
| **Timed fetches** | every request has a deadline; a slow target cannot hold the scan open |
| **Normalised targets** | input is normalised before use rather than passed through |
| **Bounded probe set** | a fixed, small path list, not a wordlist crawl |
The combination matters: it is a scanner you can point at your own site from a chat
window without wondering whether you just launched an attack.

## AVAILABILITY
The gateway that routes scan requests holds both a primary and a **fallback** scanner
endpoint, so a single deployment problem does not remove the capability. Failover is
designed rather than incidental — the same posture as the model fallbacks in
[CLASSIFICATION](CLASSIFICATION.md).

## WHAT IS NOT PUBLISHED
The probe path list, the grading weights and cut-offs, the secret-detection patterns, and
the response and time caps. A published pattern set is a checklist for hiding from it.

