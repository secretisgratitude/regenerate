# HANDOFF — Regenerate / TrueFoundry Agent Harness Hackathon (2026-09-19)

_Repo: github.com/secretisgratitude/regenerate — **PUBLIC** as of 2026-09-19._

## Status: SUBMITTED. Waiting on announcements.

Schedule (PDT): submissions closed 4:00 PM · selected demos ~5:00 PM ·
judging and awards ~6:30 PM · event ends 8:00 PM. Starred times were a working
schedule; only the 4:00 deadline was confirmed by the event lead.

Submitted on HackerSquad with project name **Regenerate**, the full README
description, TrueFoundry checked, stack line
(MCP Streamable HTTP, Node.js, SQLite, OpenAI), git remote, and the recorded
demo. Two takes exist on their platform; **judges see the latest one (Take 2,
3:31 PM, 4:03)** — there is no take selector, only re-record.

## What was built

A mock payer with an immutable SQLite ledger, exposed to TrueForge over MCP
(Streamable HTTP) as three tools:

- `get_claim` — read
- `prepare_resubmission` — mints a server-side operation id and a sha256 of the
  canonicalized `{claim_id, amount}`; cannot write
- `submit_claim` — the ONLY shielded tool, the only one that writes

Plus one skill, `skills/safe-resubmission/SKILL.md`, carrying the retry policy
as portable instructions rather than buried prompt text. Loaded into the agent
from the public repo (skills load by git URL, folder, branch — which is why the
repo had to go public).

## The claim, stated correctly

TrueForge pauses on **every** shielded call, including retries — verified live,
two submits produced two separate pauses with different tool_call_ids. It does
NOT hand out reusable permissions.

The honest, narrower claim: **the harness cannot RELATE one approval to
another.** Each approval references one specific pending tool call, so the same
business operation can be approved twice and every yes is correct in isolation.
Nothing compares them.

Codex's framing, the best sentence produced:
*"The harness's unit of authorization is a tool invocation, while the
application's unit of correctness is a business operation."*

**Do not reintroduce the old framing** ("an agent can carry a stale approval to
a changed payload"). That was FALSE, was caught in adversarial review, and was
corrected before recording.

**Confirmed app-side by Cy (TrueFoundry) in person before recording:** relating
approvals is an application concern, not something TrueForge is likely to do.
That turned the boundary section from a hedge into a design decision.

## Late fix worth remembering

`tests/e2e.sh` asserted a global `COUNT(*) = 1` on the ledger. By beat 3 the
live demo had already committed a row, so the test printed **FAIL** on camera at
the exact moment it was meant to prove correctness. It could never have passed
in that position.

Fixed by scoping the assertion to the three operations the run itself creates,
and additionally checking the surviving row is the happy-path operation. It now
passes in any ledger state and says something stronger:
`three operations attempted, exactly one row written`.

## Open threads

- **Announcements pending.** If top 10, there is a live demo slot ~5:00 PM.
- **Q&A answers, ready:**
  - *"Why not in the harness?"* → "I asked. It's app-side. That's where I put it."
  - *"Isn't this Okta?"* → "Identity tells you who's asking. It doesn't tell you
    what you already answered."
  - Don't know → "I'd have to check before I tell you something wrong."
- **Question for Cy, unasked:** "You mentioned the human-vs-agent headers.
  That's who's asking. What I ran into is whether *this* approval is the same
  operation as one I already approved. App-side, or something the harness might
  carry?"

## What transfers, and what does not

**Transfers — the pattern, ~40 lines wherever there is a write endpoint:**
server mints the operation id; fingerprint the payload at prepare; re-verify at
commit; same fingerprint replays the receipt, different fingerprint refuses.
Relevant to ClaimRail, the YesOnUs OD sync, and the Medicare billing agent —
anywhere an agent drives a write. In claims a duplicate submission is an audit
finding, not a cleanup task.

**Does not transfer:** this repo. It is a mock payer with one synthetic claim.
Its job was to make an argument on camera. Do not extend it into a product.

**TrueForge itself:** watch, do not build on yet. Pinned 0.2.0, known approval
lifecycle bugs (#508 double-click, #494/#498 undeletable connectors and skills),
and it OOM-killed itself five times on this machine.

## The honest accounting

Max upside $2,000 at maybe 15%. Expected value roughly $300 for a full day.
Meanwhile **$75,377 across 21 claims sat untouched for 57 days** behind a Stedi
837P/835 enrollment for payer 011, CH00033 in EDISS, and the HETS attestation
for NPI 1700350485 — plus the N290 rendering provider reassignment that gates
all Medicare downstream.

That is the richest hour available anywhere, and it is a portal login and a
form. Next working session goes there, not here.

## Don't do

- Don't claim TrueForge lets an agent reuse an approval.
- Don't extend this repo.
- Don't re-record unless an announcement requires it — judges see the latest take.
- Don't let Claude start or poll TrueForge.
