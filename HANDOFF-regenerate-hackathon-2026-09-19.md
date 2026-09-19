# HANDOFF — Regenerate / TrueFoundry Agent Harness Hackathon (2026-09-19 13:12)

_Scope: `repo:regenerate` (github.com/secretisgratitude/regenerate, PRIVATE)_

## Goal

Win a top-3 prize at the TrueFoundry Agent Harness Hackathon. Single judging
axis: "Best use of TrueFoundry/TrueForge." Submission is a 3-5 minute recorded
video plus a public repo. **Hard deadline 4:00 PM; freeze building at 3:30.**

## Status

**BUILD: done and proven. Nothing left to build.**

- Mock payer with SQLite ledger, payload-hash binding, server-minted operation
  ids. Ledger rows immutable (UPDATE/DELETE triggers verified firing).
- MCP server over Streamable HTTP, three tools:
  `get_claim` (read), `prepare_resubmission` (mints id + sha256, cannot write),
  `submit_claim` (the ONLY shielded tool, writes).
- 4 unit tests pass. `tests/e2e.sh` proves all four scenarios over real MCP.
- TrueForge agent `regenerate` created (id `01m2xfgr0zxkxwtfkecz5m808g`),
  connector `two-key-claims` registered, OpenAI provider connected ($50 event
  credit), sandbox enabled.
- **Full loop verified live through the UI twice today**: agent read the claim
  via MCP, computed in the sandbox (Code Mode / Python), paused on the shielded
  tool, approval committed ONE row (`RCPT-018DDA2E`), retry replayed the SAME
  receipt with no second row.
- Beat 3 (tampered amount refused) verified over real MCP, zero rows written.

**NOT DONE:**
- Zero full rehearsals with narration and a timer.
- Nothing recorded.
- Repo still PRIVATE. Must be public to submit — **this is a gated action,
  ask Eric before flipping.**
- Nothing submitted on HackerSquad.

## Files touched

- `server/ledger.mjs` — the payer. propose/approve/deny/submit, hash check at
  commit, atomic operation-status + ledger-row write.
- `server/mcp-server.mjs` — three MCP tools + /approve /deny /commit /health.
- `server/fixtures.mjs` — one synthetic claim, CLM-75377, $75,377, code A8.
- `tests/ledger.test.mjs` — four deterministic scenarios.
- `tests/e2e.sh` — same four over real MCP + HTTP. The rehearsal script.
- `preflight.sh` — 11 checks. Run before EVERY take. `SKIP_CONNECTOR_CHECK=1`
  skips the expensive one.
- `reset-demo.sh` — clean ledger between takes. **Non-negotiable.**
- `README.md` — the honest framing, before/after diagram, boundaries section.
- `BEAT-SHEET.md` — timed script, think/decide/remember structure.
- `RUN-THE-DEMO.md` — exact commands and paste-text.
- `REHEARSE.md` — four-run rehearsal plan and delivery notes.
- `AGENT-SETUP.md` — config to recreate the agent if needed.

## Key decisions

- **Named "Regenerate"** over Two-Key. Two-Key points at multisig (prior art);
  Regenerate names the actual insight — agents re-derive rather than replay.
- **Shield on `submit_claim`, not on the proposing tool.** Adopted from a Codex
  plan. Originally the shield sat on the safe tool while the real commit went
  over plain HTTP outside the harness — a judge would have caught that.
- **CORRECTED THE CENTRAL CLAIM after adversarial review.** The original framing
  said an agent could carry a stale approval to a changed payload. **That is
  FALSE.** TrueForge's approval references one specific pending tool call; a
  retry is a NEW call raising its OWN pause (verified: two submits, two
  tool_call_ids, two pauses). The honest, narrower, defensible claim: the
  harness cannot RELATE one approval to another. Every yes is correct in
  isolation; nothing compares them. **Do not reintroduce the old framing.**
- **No subagents.** Three independent reviews agreed: no parallel branch exists,
  and nondeterminism hurts a short recording.
- **One fixture, not several.** The demo is about the approval boundary, not
  claims processing.
- **Three-brain result on the win-gap:** Codex won over two Claude brains. Don't
  just *narrate* the constraint — **stage it.** Deliberately approve a changed
  amount on camera and let the payer refuse it. Evidence beats assertion.
  Codex's line, best sentence anyone produced today:
  *"The harness authorizes a tool call. My application cares about a business
  operation. Those aren't the same unit."*

## Open threads / blockers

- **THE MACHINE IS THE RISK, NOT THE CODE.** TrueForge has been OOM-killed ~5
  times. Root cause investigated: 25 days uptime, 73M swapouts, ~400 MB free
  with ~8 GB stuck "inactive". It dies on *allocation spikes*, and the biggest
  spike is the **sandbox spawn — which fires on every run and is a scored
  capability.** Fixed so far: killed 4 stuck Cursor extension hosts pegged at
  100% CPU (two for 16 days; load 82 → 2), killed the YesOnUs `next-server`
  (798 MB), closed apps. Still pending: `sudo purge`, quit Preview.
- **Claude must NOT start TrueForge or poll it.** Two of the deaths were
  Claude's own writes, one was its preflight. Eric runs TrueForge in his own
  terminal; Claude reads only when asked.
- Claude's Chrome extension cannot reach localhost — cannot drive the demo UI.
  Eric drives, Claude verifies ledger state between beats.
- Agent sometimes detours (inspects tool schemas, tried `pip install` once).
  Mitigated with `preload: true` and explicit instructions. Say "it's checking
  the tool contract" and carry on — do not restart a take for this.
- Stale `ledger-test` connector from last night is still listed. Connectors
  can't be deleted (#494). Harmless; don't let it confuse you on camera.

## Next step

Run `sudo purge`, quit Preview, restart TrueForge in Eric's own terminal, then
do one full narrated rehearsal against `REHEARSE.md` with a timer.

## Don't do

- **Don't claim TrueForge lets an agent reuse an approval.** It doesn't. That
  framing was caught and corrected; reintroducing it gets the video dismissed
  by the only audience that matters.
- **Don't let Claude launch TrueForge** — its process launches get reaped, and
  its writes have killed it under memory pressure.
- **Don't record without `./reset-demo.sh` first.** A dirty ledger already
  produced TWO rows in testing, which destroys the "one row" punchline.
- **Don't double-click Approve.** Bug #508 cancels the turn it just approved.
- **Don't upgrade TrueForge.** Pinned 0.2.0; PRs #815/#807 change the approval
  lifecycle.
- **Don't add another MCP server, subagents, or a second fixture.** All
  considered and rejected — new dependencies risk the one thing that works.
- **Don't say "moves money."** It's a local SQLite mock ledger.
- **Don't build more.** Presentation is the weakest axis and the only one left.
