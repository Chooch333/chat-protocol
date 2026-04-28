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
6. **Status** — a narrative snapshot of where a project is right now. Status also absorbs handoff observations — pattern-level notes, unresolved undercurrents, and context that didn't fit a numbered bucket but matters for the next session. Example: *"Family Trip App: vibe planning environment stripped; ready for clean rebuild once Places API key is in place. Worth flagging: Charles kept circling back to whether the vibe buttons should persist — might signal the mental model isn't settled yet."*
7. **Note** — the catch-all. Quick capture when it doesn't cleanly fit yet. Gets promoted into one of the six above later.

### Priority on Next steps

Use the tag `urgent` to elevate a next step. The dashboard shows urgent next steps in their own sub-section. No other priority levels — either it's urgent or it isn't.

---

## Sessions

A **session** is a unit of work within a chat that ends with a Session Log. A chat can contain one or more sessions. Each Session Log is a milestone: future sessions reach back to prior logs as baseline and do not rewrite them. If a later session changes something a prior session decided, it supersedes — it doesn't rewrite the earlier log. "Log this state and keep working" is a session boundary.

---

## Three ticket types — all live as artifacts

**Every time Claude plans to act on the database, it produces a ticket artifact.** The artifact opens beside the chat; the chat itself stays for discussion. Conversational questions ("should we do X or Y?", "what do you think about Z?") stay in chat — they do not go on tickets.

### Session Log

For anything going *into* the database at a session boundary. One Session Log per session, regardless of how many items or what types. If six items across four buckets are going in at once, they all sit on one Session Log.

Every Session Log includes a status snapshot and a next-action line, even if the session feels light. The status snapshot is where handoff observations live — see the Status bucket description above.

**Template:**

```
# Session Log — [YYYY-MM-DD] — [short session topic]

**What this is:** [one-line summary of the session and what's being logged]

**What I'll do:** [plain English — e.g., "Log 4 items to Family Trip App: 2 decisions, 1 next step, 1 learning. Write a status snapshot."]

**What you'll do:** [e.g., "Review entries below, edit wording, approve."]

---

## Current state going in

[brief narrative — where things stand at the moment this session closes, as context for reviewing the entries]

---

## Entries to log

### 1. [Bucket] — [headline in plain English]
**Body:** [what will go in the database, in plain English]
**Project:** [project slug]
**Tags:** [comma-separated tags]

[repeat for each item]

---

## Status snapshot

[narrative covering where the project stands post-session. Include handoff observations: patterns noticed, unresolved undercurrents, anything that didn't fit a bucket but matters for the next session.]

---

## Next action line

[one line — what the next session on this project should start with]

---

## Result (fill in after execution)

**What got logged:** [bullet list of entries with returned IDs]
**Where:** [project name in Project State]
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
**Learning to capture:** [if applicable — this often becomes a Learning entry in a future Session Log]
```

### Build Brief

For handing off a scoped task to a sub-chat. Use when the current chat needs to spawn focused work elsewhere — a different codebase, a narrow investigation, a deliverable that belongs in its own context — rather than execute inline.

A Build Brief does not write to the database on creation. It's a handoff payload: a package of context and directive that lets a sub-chat execute without re-negotiating the main chat's decisions.

**Template:**

```
# Build Brief — BB-[YYYY-MM-DD-slug]

**What this is:** [one-line summary of the task being handed off]

**What I'll do:** [what the sub-chat is expected to produce]

**What you'll do:** [approve the Brief, paste it into the sub-chat, any other context-setting]

---

## Current state going in

[brief narrative — what the main chat knows that matters for this work]

---

## Receiving chat

[named sub-chat, or "new chat to be opened"]

---

## Scope

**In scope:** [what the sub-chat will work on]

**Out of scope:** [what it won't touch; common forks pre-answered]

---

## Directive

[do X, return Y in format Z — prescriptive]

---

## Inputs

[files, links, DB entries, prior decisions the sub-chat needs]

---

## Pasteable prompt

[exact first-message text for the sub-chat]
```

**Protocol defaults for Build Briefs:**

- **Brief ID.** Auto-generated as `BB-YYYY-MM-DD-[slug]`. No need to choose.
- **Return contract.** The sub-chat closes with its own Session Log on the same project, referencing the Brief ID. The main chat reads results from the database like any other chat — no special ceremony.

**Build Brief rules:**

- **Authority.** Charles is always decider in whatever chat he's in. When a sub-chat hits an unresolved fork the Brief didn't pre-answer, the sub-chat pauses and surfaces the fork in-chat. Sub-chats do not decide autonomously.
- **Staleness.** A Brief is a creation-time snapshot. When a sub-chat starts, it reconciles against the current Project State DB. If the Brief conflicts with current DB state, the sub-chat escalates rather than following stale guidance. The DB is truth; the Brief is guidance.

**When to use a Build Brief vs. execute inline.** No hard rule yet. When inline feels wrong but no Brief is made, or when a Brief is made but feels like overkill, log the observation as a Learning. A rule will emerge from practice.

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

## Build execution — tool-first

Build and planning chats are MCP- and tool-first. Claude takes over as many build tasks as available tools allow, rather than handing instructions to Charles. Examples:

- **Supabase MCP** — create projects, run migrations, manage tables, fetch keys
- **Custom GitHub MCP** — create repos, commit files, open PRs
- **Vercel MCP** — connect repos, auto-deploy, manage env vars, fetch logs
- **Context7 MCP** — pull current library docs before writing code

This applies during planning as much as during building. Plans, specs, briefs, and Build Briefs are written assuming the eventual executor will hand work to MCPs — name the specific MCP for each step and include the parameters that step needs. Avoid telling Charles to do by hand what an MCP can do.

When a needed MCP is unavailable or fails, Claude flags the gap and falls back to instructions. The default is tool execution; manual handoff is the exception.

---

## Standing rules (database behavior)

- **No logging without a ticket.** Even a single-item log goes on a Session Log artifact.
- **One Session Log per session.** Batch all log entries onto one Session Log. Don't spawn multiple artifacts for the same session.
- **Every Session Log writes a status snapshot.** The snapshot is a mandatory field on every Session Log, not optional. Light sessions still get one.
- **Plain English on tickets.** Technical syntax (SQL, function calls, MCP commands) belongs in the execution step, not on the ticket Charles is reviewing.
- **Chat questions stay in chat.** Clarifying questions, trade-off discussions, and open-ended thinking happen in the chat thread — not on artifacts.
- **Dashboard structure.** Any command center or dashboard that surfaces Project State data must have a section for each of the seven buckets, with urgent next steps broken out as their own sub-section under Next steps.
- **Pre-check parameter syntax on `write_status_snapshot`.** The `project_slug` parameter on this tool is prone to malformed-tag slips that fail with "Project not found: 'undefined'." Verify all parameter tags are well-formed before submitting. If the call fails with that exact error, recognize it as a parameter-syntax slip and retry — do not search for a different cause. (Captured as Learning C-034 on `chat-protocol`.)
- **Before the first database action of any chat, Claude reads this document.**
- **Coordinate with existing projects before creating new ones.** Build chats do not automatically create a new project on start. At the first log moment, Claude calls `list_projects` and identifies which existing project the content belongs to. If the fit is clear, log there. If the content doesn't clearly fall under an existing project, Claude pauses and asks for direction before creating one.

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
