# Chat Protocol for the Project State Database

**Read this before reading from or writing to Project State.** This document defines the shared vocabulary, ticket formats, workflow, and engagement rules every chat uses when working with the database. Follow it as written. If something isn't covered here, ask in chat before acting.

**Location:** `https://github.com/Chooch333/chat-protocol/blob/main/PROTOCOL.md`

---

## The seven buckets

Every item that goes into Project State fits one of these. Use the plain-English name on tickets. Never invent new categories; if something doesn't fit, use **Note** and we'll sort it out.

1. **Decision** — a choice that's been made, with the reason why. Example: *"Using Supabase MCP for DDL rather than copy/paste SQL, because it removes a manual step."*
2. **Next step** — a concrete action someone is going to take. Example: *"Wire Google Places API key into Vercel env vars for the Family Trip App."*
3. **Open question** — something unresolved that's blocking or shaping progress. Example: *"Is Joe producing the Ridgeworks operating agreement, or is that on Charles?"*
4. **Learning** — a retrospective observation worth remembering for next time. Example: *"Claude Code prompts need explicit Preserve/Verify sections or it wanders off-scope."*
5. **Assumption** — something being treated as true for now, but might need to be revisited. Example: *"Assume 80% LTV on BRRRR refi until the lender confirms."*
6. **Status** — a narrative snapshot of where a project is right now. Example: *"Family Trip App: vibe planning environment stripped; ready for clean rebuild once Places API key is in place."*
7. **Note** — the catch-all. Quick capture when it doesn't cleanly fit yet. Gets promoted into one of the six above later.

### Priority on Next steps

Use the tag `urgent` to elevate a next step. The dashboard shows urgent next steps in their own sub-section. No other priority levels — either it's urgent or it isn't.

---

## Two ticket types — both live as artifacts

**Every time Claude plans to act on the database, it produces a ticket artifact.** The artifact opens beside the chat; the chat itself stays for discussion. Conversational questions ("should we do X or Y?", "what do you think about Z?") stay in chat — they do not go on tickets.

### Log Ticket

For anything going *into* the database. One ticket per log session, regardless of how many items or what types. If six items across four buckets are going in at once, they all sit on one ticket.

**Template:**

```
# Log Ticket — [YYYY-MM-DD] — [short session topic]

**What this is:** [one-line plain-English summary of what's being logged and why]

**What I'll do:** [plain English — e.g., "I'll log 4 items to the Family Trip App project in Project State: 2 decisions, 1 next step, 1 learning. All tagged 'vibe-planning'."]

**What you'll do:** [e.g., "Review the entries below, edit any wording, approve. Nothing else." — or specify what's needed from you]

---

## Entries to log

### 1. [Bucket] — [headline in plain English]
**Body:** [what will go in the database, in plain English]
**Project:** [which project this belongs to]
**Tags:** [comma-separated tags]

### 2. [Bucket] — [headline]
**Body:** [...]
**Project:** [...]
**Tags:** [...]

[repeat for each item]

---

## Result (fill in after execution)

**What got logged:** [bullet list of entries with their returned IDs]
**Where:** [project name(s) in Project State]
```

### Fix Ticket

For anything that breaks — and the fix. Break and solution live on the same ticket; the resolution section gets filled in after the fix lands.

**Template:**

```
# Fix Ticket — [YYYY-MM-DD] — [short description of what broke]

**What broke (plain English):** [the symptom — what Charles is seeing, or what stopped working]

**What I think is happening:** [hypothesis in plain English, not jargon]

**What I want to try:** [the proposed fix, described as actions not code]

**What could go wrong:** [risks, side effects, anything else that might be affected]

**What you'll do:** [review and approve, provide info, run a command, etc.]

---

## Resolution (fill in after)

**What we actually did:** [...]
**Did it work:** [yes / no / partially — with note]
**Learning to capture:** [if applicable — this often becomes a Learning entry in a future Log Ticket]
```

---

## Workflow

Every session that touches Project State follows this loop:

1. **Identify** — Claude identifies what needs to be logged, or what broke.
2. **Propose** — Claude produces the appropriate ticket as an artifact. Plain English. Specific about what Claude will do and what Charles needs to do.
3. **Review** — Charles reviews in chat. Edits wording, adjusts buckets, removes items, approves.
4. **Execute** — Once approved, Claude runs the actual Project State commands (or ships the fix).
5. **Close the ticket** — Claude updates the Result/Resolution section on the artifact with what actually happened.

---

## Ticket closure rule

**A ticket closes clean, or it doesn't close.** If a caveat, problem, or unresolved item comes up during execution, Claude does **not** write it into the Result/Resolution section as an open caveat. Instead:

1. Claude pauses and raises the caveat in chat.
2. Charles and Claude discuss it until it's resolved.
3. Once resolved, Claude completes the Result/Resolution with the outcome reflecting that resolution.

Tickets have no "open items" field. If something is open, the ticket isn't done. Reports are conclusions, not status updates with loose ends.

---

## How to engage during builds

These rules govern how Claude structures every chat response when working with Charles on build, project, or planning work — construction projects, app development, Ridgeworks operations, Project State logging, workflow and process design, or any substantive deliverable.

**Context scope.** For casual conversation, brainstorming, emotional or reflective topics, quick factual questions, or any non-build chat, Claude uses natural conversational style. The rules below are tools for managing complexity in build work — they should not be applied to contexts where they add friction without value.

1. **One "Needs from you" section per response.** Every response requiring input ends with a single numbered list under that exact label. No decisions or approvals buried anywhere else in the response. If nothing's needed, no section — the response just ends.

2. **Every choice comes with a recommendation.** When presenting options, Claude picks one and says why. Charles can approve in a word or override. No open-ended "what do you think?" without a proposed answer.

3. **One layer, not two.** No "flags + questions" or "quick one + bigger one" splits. Everything requiring engagement goes on the same numbered list.

4. **Choices go in Needs from you, not just questions.** Anything Claude decided without asking — including what was left out, excluded, or chosen over an alternative — goes in Needs from you. If it's a judgment call Charles might want to reverse, surface it as a numbered item. Only descriptive commentary (context, explanations of what was done, how something works) stays in the body.

5. **Questions phrased as closed choices.** "Add now, or wait?" not "What do you think about this?" Closed questions are faster to answer.

6. **Visual separator.** A horizontal rule (`---`) immediately precedes every Needs from you section, so it's consistently and clearly separated from the surrounding discussion. No exceptions.

---

## Standing rules (database behavior)

- **No logging without a ticket.** Even a single-item log goes on a Log Ticket artifact.
- **One artifact per session.** Batch all log entries onto one Log Ticket. Don't spawn multiple artifacts for the same log session.
- **Plain English on tickets.** Technical syntax (SQL, function calls, MCP commands) belongs in the execution step, not on the ticket Charles is reviewing.
- **Chat questions stay in chat.** Clarifying questions, trade-off discussions, and open-ended thinking happen in the chat thread — not on artifacts.
- **Dashboard structure.** Any command center or dashboard that surfaces Project State data must have a section for each of the seven buckets, with urgent next steps broken out as their own sub-section under Next steps.
- **Before the first database action of any chat, Claude reads this document.**

---

## Appendix — Bucket to MCP tool mapping (for Claude)

This is for Claude's reference when executing after ticket approval. Charles does not need to read this.

| Bucket         | Project State tool            |
|----------------|-------------------------------|
| Decision       | `log_decision`                |
| Next step      | `add_next_move`               |
| Open question  | `add_blocker`                 |
| Learning       | `add_lesson`                  |
| Assumption     | `add_assumption`              |
| Status         | `write_status_snapshot`       |
| Note           | `add_note`                    |

For supersessions, use `supersede_decision` (decisions) or `update_plan_content` (plans). Tags go in via `add_tags` or at creation time. Always verify the project is registered with `list_projects` before logging; if not, `create_project` first.
