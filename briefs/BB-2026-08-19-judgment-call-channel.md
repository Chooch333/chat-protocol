# Build Brief — BB-2026-08-19-judgment-call-channel

**Git home:** Chooch333/chat-protocol · `briefs/BB-2026-08-19-judgment-call-channel.md`
**Project State plan:** `1641e970-27b9-4d04-ae31-d52325d031c3` on `context-database`

**What this is:** Extend the existing Q-channel so build chats can post *disclosures* — anything they'd otherwise tell Charles in chat that isn't a blocking question and isn't just "done" — with a copy/paste ID Charles carries to a design chat for optional review.

**What I'll do:** Add one new origin type and one new terminal status to the existing `build_questions` tables, teach build chats to route all non-"done" surfacings here, and teach DA chats to surface them at session start. No new tables.

**What you'll do:** Ship this as a queued plan the orchestrator (or a build chat) executes.

---

## Restate — what you're after

When a build chat finishes a brief, it decides its own calls and moves on — the protocol says it should. But it often still has *something to tell you*: a call it made on its own authority, a loose end it noticed, an "FYI this was left open." Today that lands in a chat closeout you have to read in the moment, or gets buried in a fork log or a tagged Decision. None of it gives you a **pointer you can hand to a design chat** the way the Q-channel does for blocking questions.

**The rule, in your words:** if the build is going to tell you *anything* other than "it's done and ready to log the build log," that thing goes on the comms table — not into a chat message. You are not defining what qualifies; the build already decides that. The channel is simply the destination for "something the build wants Charles to see."

**The goal behind the goal:** keep yourself out of the build loop (calls get *made*, not asked) while getting one low-effort, non-blocking surface to audit whatever the build flags — and catch the occasional bad call before it rots the database.

## Acceptance criteria (what proves this worked)

1. A build chat can post a disclosure and hand you a single prefixed ID (e.g. `CBR-J-003`).
2. That ID pasted into a DA chat surfaces the full disclosure — what it is, why, what it touched — with no relay of content by you.
3. Disclosures **never** appear in the "open questions blocking this build" list a DA chat runs at session start. They live in a separate "Calls to review" list.
4. Review is optional and non-blocking: an un-reviewed disclosure holds up nothing.
5. A DA chat can mark a disclosure reviewed (agree / disagree-and-corrected) in one step, and that disposition is recorded.

---

## Design decision — reuse the Q-channel, don't build a new one

**Verified by checking the live DB:** `build_questions` + `build_question_messages` already carry everything a disclosure needs — project binding, `plan_id` binding to the originating brief, prefixed globally-unique `display_id`, free-text `context` body, `tags` array, append-only message thread, `linked_decision` pointer. Only 2 rows exist total, both resolved — no legacy volume to disturb.

`status` and `origin` are plain `text` with no database-level value restriction (**verified** — no enum, defaults `'open'`/`'build'`). New values are added by convention + the MCP layer, not a migration. Cheaper, and nothing existing breaks.

**A disclosure is a Q-channel row with:**
- `origin = 'build-judgment'` (new value — marks it a disclosure, not a blocking question)
- `status = 'noted'` on creation (new value — posted, non-blocking, awaiting optional review)
- a `judgment-call` tag

Because it's born in `noted` and never in `open`, every existing "what's blocking" query (which filters `status IN ('open','discussing')`) ignores it automatically. **That's the entire isolation mechanism.**

### Scope of the channel (Fork 2 — resolved)
The channel holds **anything a build would otherwise surface to Charles that is neither a blocking question nor the clean "done, logged, nothing to flag" case.** The build is not asked to classify severity or judge what's "design-touching" — if it has something to say beyond "done," it posts it here. This is a build-to-Charles disclosure inbox, slightly broader than "judgment calls only": it also catches "heads up, X is still open" even when X wasn't a decision the build made.

### Status lifecycle for a disclosure
```
noted                → posted by the build, non-blocking, awaiting optional review
  → reviewed-agree      → a DA chat looked, concurs, done
  → reviewed-corrected  → a DA chat disagreed and made a correcting decision
```
`reviewed-corrected` reuses the existing rule that a resolving state must name its decision (`linked_decision` + `resolved_decision_project`) — a disagreement always leaves a trail to the fix. `reviewed-agree` needs no decision.

### Prefix (Fork 1 — resolved: parallel `-J-`)
Disclosures get a per-project `-J-` series (`CBR-J-001`, `ABO-J-001`) parallel to the `-Q-` question series. Same globally-unique discipline; the ID alone tells you which kind it is.

### Escalation (Fork 3 — resolved: no push)
Pure pull-inbox. It never nags you. If a build hits something it genuinely can't decide, that's a blocking question (`open`), not a disclosure — the two stay cleanly separated. Builds print their `-J-` IDs at close, so unreviewed items are in your session record even though nothing pushes.

---

## How each side behaves

### Build chat, at brief close (new exit step)
Any time the build would otherwise tell Charles something beyond "done and logged," it posts one disclosure row instead: title, the what/why/what-it-touched in `context`, `plan_id` = the brief, tag `judgment-call`. It then prints the IDs in its closeout:
> *"Posted for design review: `CBR-J-003`, `CBR-J-004`. Carry these to a design chat if you want them reviewed."*
The only thing that does NOT get posted is the clean done-nothing-to-flag close.

### DA chat, at session start (extends the existing Q-channel check)
Already pulls open questions on its project. Now also pulls `origin='build-judgment' AND status='noted'` and lists them under a distinct **"Calls to review"** heading, separated from blocking questions. For each: read the thread, then mark `reviewed-agree`, or disagree → log a correcting Decision → mark `reviewed-corrected` with the link. Unreviewed `noted` items are shown but never gate the session.

---

## Scope

**In scope:**
- New convention values (`origin='build-judgment'`; statuses `noted` / `reviewed-agree` / `reviewed-corrected`; `-J-` prefix).
- MCP tool support on Project State for posting and disposing disclosures.
- Doctrine: one paragraph in the build-chat exit routine, one in the DA SKILL session-start routine, and a Q-channel extension in PROTOCOL.md.

**Out of scope (pre-answered):**
- No new tables — reuse `build_questions` / `build_question_messages`.
- No change to blocking-question behavior — this sits alongside, untouched.
- Not a notification system — pull inbox, checked when you bring an ID.
- No agent self-review — humans / DA chats only.

---

## Build sequence (MCP-first, for the executor)

1. **Project State MCP — extend write path.** Support `origin='build-judgment'` + initial `noted` status (or add a thin `post_judgment_call` wrapper). Params: `project`, `plan_id`, `title`, `context`, `tags` (defaults include `judgment-call`). Returns the `CBR-J-00N` display_id. Repo: `Chooch333/project-state-mcp`. **S.**
2. **Project State MCP — extend read path.** `list_judgment_calls(project, status='noted')` (or a filter arg on the existing question-list tool) so DA pulls the review inbox in one call. **S.**
3. **Project State MCP — disposition tool.** `dispose_judgment_call(display_id, disposition, linked_decision?, resolved_decision_project?)`, disposition ∈ {`reviewed-agree`, `reviewed-corrected`}; enforce `reviewed-corrected` requires a linked decision (mirror the resolve-needs-decision rule). Appends a `build_question_messages` row for the trail. **S–M.**
4. **`-J-` counter.** Wherever the `-Q-` display_id is minted per project, add a parallel `-J-` sequence. **S.**
5. **Doctrine — build side.** Add an exit-routine paragraph to `Chooch333/agent-library` (`skills/execute-build-task/` and/or `skills/orchestrate-build/`): at close, route anything-beyond-done to the disclosure channel and print the IDs. **S.**
6. **Doctrine — DA + protocol.** Add a session-start paragraph to `roles/design-assist/SKILL.md` ("Calls to review" pull + disposition). Extend the PROTOCOL.md Q-channel section to name the `build-judgment` origin and the `noted → reviewed-*` lifecycle, and to state the routing rule (anything a build would tell Charles other than "done" goes here). **S.**
7. **Verify live.** Post one real disclosure from a test build, carry its `-J-` ID into a DA chat, confirm it surfaces under "Calls to review" and disposes correctly. **S.**

**No schema migration needed** (verified: status/origin are unconstrained text). Enforcing allowed values with a CHECK constraint later is optional hardening, not part of this build.

## Gated items (executor resolves before coding)

- **Where the `-J-` counter lives.** Whether the `-Q-` display_id is minted in server code or a DB trigger is not verified from the design chat. The executor reads `project-state-mcp` first and places the `-J-` counter accordingly. Gated on repo access the executor has; not a design blocker.

## Completeness report

All eleven domains Answered or N/A. Forks closed: prefix (`-J-`), channel scope (anything-beyond-done), escalation (no push). No open items remain — build-ready.
