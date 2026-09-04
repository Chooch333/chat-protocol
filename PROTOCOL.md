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

## Campaigns and the naming rule

Campaigns are a grouping layer that sits above projects and tags — a table (`campaigns`: `slug`, `title`, `purpose`, `status` — `active`/`done`/`parked`) for organizing related plans across a project's lifetime. Plans now carry `plain_title`, `plain_summary`, `campaign_id`, `designed_in`, plus review fields (`reviewed_at`, `reviewed_by`, `review_notes` — see Reviewing a succeeded plan, below).

**DA authority over campaigns.** DA/planning chats have full fluidity authority over campaigns — rename one, recategorize a plan into a different campaign, fork a new campaign out of an existing one — freely, logged as a judgment call (same provenance discipline as any other judgment call). Charles is never asked to approve a campaign change.

**The naming rule.** Every chat that creates a plan or files a note writes its best-shot `plain_title` / `plain_summary` at creation — every time. There is no tool gate enforcing this; writes always succeed with or without a plain label. This is mandatory chat behavior, not a validation. If "(needs a name)" is showing on the Board, that's a chat defect — fixed by the next chat that touches the item. Charles is never the one who generates a name.

**What Charles sees (naming legend).** *This paragraph is the text the dashboard displays to Charles.* Builds carry a BB filing code (e.g. `BB-2026-08-27-comms-hub-plumbing`) for chats' cross-referencing only — Charles never needs to know it or use it. The names Charles actually reads are the plain titles: `plain_title` and `plain_summary` on each plan.

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

A Build Brief is both a handoff payload and a queue entry. At authoring it is committed to git and written to Project State as a plan (see Shelving a brief, below) — it does not otherwise write Decisions, Next steps, or other buckets on creation.

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
- **Advisor-origin closing hook.** A Build Chat executing an advisor-origin brief (its plan carries an `ADV-NNN` tag) closes by writing the **actual response** into `cbrain/docs/advisor/LEDGER.md` — what really landed, if different from the DA chat's intended response — and setting that row's status to `addressed`. This is part of "done," not a follow-up: the LEDGER response is the readable answer that lets the Stack Advisor stop inferring adoption from a `building` status. It rides the existing external-review gate above — a self-certified response is a build defect, caught the same way a self-certified review is.

**Shelving a brief (the build queue).** The shelf is Project State: **plans with status `queued`**. The orchestrator pulls the oldest queued plan on a project. Every Build Brief gets three things in the same turn it's authored:

1. **Git home.** Commit the brief to the target project's repo at `docs/design/BB-YYYY-MM-DD-slug.md`. If the build has no target repo, use `chat-protocol/briefs/` instead.
2. **Plan.** `write_plan` on the target project's Project State project. Content is the full brief text; the brief header carries the git fetch path (owner, repo, path) so any chat can find the readable copy.
3. **Status.** When the brief passes the completeness gate and is build-ready, `update_plan_status` → `queued`. Until then it stays `draft`. Queued means on the shelf; nothing else does.

Interactive handoff (pasting a prompt into a sub-chat) still works — but the pasteable prompt instructs the sub-chat to fetch the brief from its git home rather than trust pasted text. The plan is the authoritative queue entry; the git copy is the durable, human-readable one. Keep them in sync via `update_plan_content` when a brief is amended.

**Reviewing a succeeded plan (the review gate).** A plan that reaches status `succeeded` is not finished — it's *unreviewed*. Before it counts as done, a DA/planning chat audits it: verifies the executor's claims against live code/data (not just the report), disposes of every disclosure the build posted (`list_judgment_calls` → `dispose_judgment_call`), and fills in or fixes any missing or weak plain labels (a nameless item gets named here at the latest — never by Charles). Only then does the reviewing chat call `review_plan`. **The reviewer must be external.** The chat that executed the build — including any of its subagents — never calls `review_plan` on its own plan and never disposes its own disclosures; a build's disclosures close at status `noted`, awaiting the external pull. A build that ends with its disclosures already `reviewed-agree` or its plan's review fields already populated has **self-certified — a build defect**: the reviewing DA chat redoes the review for real and logs the defect. DA/planning chats pull succeeded-and-unreviewed plans (status `succeeded`, `reviewed_at` null) at session start, alongside the disclosure inbox — the same intake moment, two lists.

**Build Brief rules:**

- **Authority — two modes.** *Interactive chats* (Charles present): Charles is always decider. When a sub-chat hits a fork the Brief didn't pre-answer, it pauses and surfaces the fork in-chat, with a recommendation. *Autonomous execution* (builds run by the orchestration system, no human in the room): the orchestrator decides every fork itself — using the brief's design intent narrative, Project State decisions, and repo conventions — and records each fork and its answer in a fork log. It halts only at hard gates: credentials or access only Charles can supply, spending real money, or destroying/irreversibly changing data. Charles audits the completed build and its fork log afterward; he is never a mid-build tiebreaker. Full rules: `agent-library/roles/design-assist/references/brief-completeness-framework.md`.

**The line between "designing" and "building" is the line for Charles's involvement — and it is firm.** Charles is in the loop during Design Assist and planning: that is where scope is set, forks are his to call, and direction is shaped. **Once a build brief is being executed, Charles is out of the loop until a hard gate.** A build chat executing a brief does not pause to ask Charles about forks, approach, wording, cleanup, ambiguity, or brief-versus-reality conflicts — it decides all of them itself, logs them, and moves forward. The only three things that stop an executing build and bring Charles in: (1) credentials or access only he can supply, (2) spending real money, (3) destroying or irreversibly changing data. Nothing else. This is not "in flux" or a direction being explored — it is the settled operating model. When in doubt during execution, the correct action is to decide, log, and proceed, not to ask. Asking Charles to approve something an executing build could decide for itself is itself a defect (it makes him the bottleneck the whole system exists to remove).

**Automatic data intake follows the same rule.** The ingestion pipelines run on their own — ingesting, adding data, moving forward — without per-item approval. Charles course-corrects from what he observes in the accumulated data, not by approving each addition. "On and running with Charles watching the output" is the target state, not "paused pending Charles's review."
- **Staleness.** A Brief is a creation-time snapshot. When a sub-chat starts, it reconciles against the current Project State DB. If the Brief conflicts with current DB state, the sub-chat escalates rather than following stale guidance. The DB is truth; the Brief is guidance.

**When to use a Build Brief vs. execute inline.** No hard rule yet. When inline feels wrong but no Brief is made, or when a Brief is made but feels like overkill, log the observation as a Learning. A rule will emerge from practice.

---

## Chat types

Most chats are **build chats** by default — work that produces deliverables, decisions, or progress on a project. The rules in this document apply.

Some chats are **brainstorm chats**, signaled by Charles opening with **"this is a brainstorm chat"** (case-insensitive, in the first message). Brainstorm chats are different — they fire the seven-agent orchestration system rather than producing inline work. When Charles opens with that phrase:

1. Continue reading this document as normal.
2. **Also read `https://github.com/Chooch333/chat-protocol/blob/main/BRAINSTORM.md` via Custom GitHub MCP `get_file_contents` (owner=Chooch333, repo=chat-protocol, path=BRAINSTORM.md) before responding.**
3. Follow `BRAINSTORM.md` end-to-end for the rest of the chat. The rules in this document still apply (one Needs from you, recommendations on choices, etc.) but the workflow shape is governed by `BRAINSTORM.md`.

If Charles does not open with that phrase, ignore `BRAINSTORM.md` entirely. The word "brainstorm" used casually mid-chat is not an invocation.

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

7. **Relational capture is a checkpoint.** During project, PM, or build work, treat noticing recordable cbrain material as an ongoing checkpoint — new people, organizations, projects, or places worth an entity, and relationships between them worth a tuple (e.g., who worked with whom, who a project's client is). When concrete relational data surfaces in the conversation, surface it as a Needs from you item proposing the entity or relationship for capture, rather than letting it pass unrecorded. Propose, don't auto-write; and don't manufacture relationships from thin evidence (the discipline from `cbrain predicate discovery` below applies — only surface when the relationship is concretely stated, not inferred).

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
- **Judgment calls get provenance.** Any decision Claude makes on judgment — rather than explicit direction from Charles or a brief — is logged as a Decision with provenance marking it Claude-decided and the tag `judgment-call`, naming the basis (brief intent, standing convention, or inference). In autonomous builds this lives in the fork log; in interactive chats it rides the session's normal Session Log.
- **One Session Log per session.** Batch all log entries onto one Session Log. Don't spawn multiple artifacts for the same session.
- **Every Session Log writes a status snapshot.** The snapshot is a mandatory field on every Session Log, not optional. Light sessions still get one.
- **Plain English on tickets.** Technical syntax (SQL, function calls, MCP commands) belongs in the execution step, not on the ticket Charles is reviewing.
- **Chat questions stay in chat.** Clarifying questions, trade-off discussions, and open-ended thinking happen in the chat thread — not on artifacts.
- **Dashboard structure.** Any command center or dashboard that surfaces Project State data must have a section for each of the seven buckets, with urgent next steps broken out as their own sub-section under Next steps.
- **Pre-check parameter syntax on `write_status_snapshot`.** The `project_slug` parameter on this tool is prone to malformed-tag slips that fail with "Project not found: 'undefined'." Verify all parameter tags are well-formed before submitting. If the call fails with that exact error, recognize it as a parameter-syntax slip and retry — do not search for a different cause. (Captured as Learning C-034 on `chat-protocol`.)
- **Coordinate with existing projects before creating new ones.** Build chats do not automatically create a new project on start. At the first log moment, Claude calls `list_projects` and identifies which existing project the content belongs to. If the fit is clear, log there. If the content doesn't clearly fall under an existing project, Claude pauses and asks for direction before creating one.
- **Before the first database action of any chat, Claude reads this document.**
- **Stack-map upkeep.** cbrain holds the canonical stack-map (`get_entity('stack-map')`). Sessions never edit it directly. Any session that ships a stack change — new component, retirement, dependency change, or status change (planned → live, etc.) — files a suggestion: an open next_move on the `stack-map` Project State project, format `MAP-EDIT: <component id> — <proposed change> — <reason> — source: <session/build id>`. The Stack Manager role disposes of suggestions autonomously and logs every disposition; do not wait on it, and do not escalate map edits to Charles.

---

## The Comms Table

**The Comms Table** is the standing name for this location — the `build_questions` + `build_question_messages` table pair. Use "the Comms Table" in chat, tickets, and doctrine whenever referring to this location; it is the canonical term and no chat should have to infer what it means. The Comms Table carries two lanes, described below: the **Q-channel** (blocking questions) and the **disclosure channel** (non-blocking judgment calls).

### The Q-channel (blocking questions)

The Q-channel is a **build-chat instrument**. Its whole job: when a build chat executing a brief hits something genuinely above its authority, it files a question instead of stalling or guessing, and Charles carries the ID to a DA/planning chat that answers it — without Charles having to relay the content by hand. It is not a general question box.

**Who does what — this is firm:**
- **Build chats MUST use it.** A build chat executing a brief that hits a decision point beyond its authority (see the hard-gate list — but also genuinely design-level forks the brief left silent) files a question here rather than asking Charles ad hoc. Every build-chat question is **bound to its build brief** (`plan_id`), so two builds' questions never blur even on the same project.
- **DA/planning chats ANSWER; they do not originate.** A DA/planning chat, at session start, checks for open questions on its project and answers them. It does **not** push new questions to Charles through this channel on its own initiative — DA chats surface their forks to Charles in-chat as normal. The one exception: if Charles explicitly says "log this as a question," the DA chat files it (origin `charles-directed`) and Charles's reference resolves to it.
- **Charles carries the pointer, not the payload.** Build chat surfaces the ID → Charles brings it to a DA/planning chat → that chat reads the thread and answers. The thread holds the content; Charles moves only the ID.

**The tables:** `build_questions` (one row per question) and `build_question_messages` (append-only dialogue — every turn a new row, nothing overwritten) — together, the Comms Table. Key fields on a question: `display_id` (globally unique, see below), `plan_id` (the build brief it belongs to — required for build-origin questions), `origin` (`build` or `charles-directed`), `status`, `linked_decision` and `resolved_decision_project` (the answer, and which project it lives on — they can differ).

**IDs are globally unique — always cite them in full.** Display IDs are project-prefixed: `ABO-Q-001` (agent-build-out), `CBR-Q-001` (cbrain), etc. There is no bare `Q-001` — the prefix is part of the ID. Never drop it; a bare number is ambiguous and the prefixed form is the whole point.

**Status lifecycle:** `open` (needs an answer) → `answered` (build chat can proceed) → `discussing` (clarification bouncing) → `resolved` (settled). The database **enforces** that a question cannot be marked `resolved` without naming its resolving decision (`linked_decision`), and the decision may live on a different project than the question — record that project in `resolved_decision_project`. This is the fix for the drift class where an answer got logged as a decision elsewhere and the question was left orphaned-open.

**Answering rules (DA/planning chats):**
1. At session start, query `build_questions` for `open`/`discussing` on the current project.
2. To answer: append a `build_question_messages` row (author `da`/`planning`) and set status `answered`; or ask back and set `discussing`.
3. If the answer is a durable decision, log the Decision, set `linked_decision` + `resolved_decision_project`, then set status `resolved`. Do this **even when the decision lands on another project** — the link is mandatory, not optional.
4. **Before telling Charles an open question still needs him, search for an existing decision that already answers it.** An open flag is not proof the decision is unmade — decisions can land on other projects (lesson: `cbrain/CB-101`).

### Disclosures — the build-to-Charles channel (`origin='build-judgment'`)

Alongside blocking questions, the Comms Table carries a second, non-blocking lane: anything a build chat would otherwise tell Charles beyond "done and logged" — a judgment call made on its own authority, a loose end, an FYI — posted as a disclosure rather than relayed in a chat closeout.

**The routing rule.** If a build chat is going to tell Charles anything other than "it's done and ready to log," that thing goes on the Comms Table, not into a chat message. The build decides what counts; the Comms Table is just the destination.

**The closing rule.** A build chat's final chat message may only say the build is done and that everything else is on the Board. Every judgment call, loose end, follow-up, or FYI goes to the Comms Table — with a `plain_summary` and `action_needed` flag set — never into chat text.

**Born non-blocking.** A disclosure starts in status `noted`, origin `build-judgment`, tagged `judgment-call` — never `open`. Every "what's blocking this build" query filters `status IN ('open','discussing')`, so disclosures are invisible to it automatically; that's the entire isolation mechanism, no special-casing needed elsewhere.

**Status lifecycle:** `noted` (posted, awaiting optional review) → `reviewed-agree` (a DA chat looked, concurs, done — no decision required) or `reviewed-corrected` (a DA chat disagreed and logged a correcting Decision — `linked_decision` is required, the same rule as blocking-question `resolved`).

**Prefix:** a parallel per-project `-J-` series (`CBR-J-001`), independent of and never continuing the `-Q-` series.

**Pull only, no push.** DA/planning chats pull the "Calls to review" inbox (`list_judgment_calls`, status `noted`) at session start, listed separately from blocking questions — nothing pushes, nothing gates. If a build hits something it genuinely can't decide, that's a blocking question (`open`), not a disclosure.

**Tools:** `post_judgment_call` (build side, at brief close), `list_judgment_calls` and `dispose_judgment_call` (DA side, at session start). Scoped strictly to `origin='build-judgment'` — the blocking-question tooling and behavior above are untouched.

This is the same discipline as the rest of the protocol: write it down where every chat can see it, don't make Charles the relay — for blocking questions and for disclosures alike, on the Comms Table.

---

## cbrain predicate discovery

In chats that touch cbrain (search_brain, get_entity, graph_query, or any mention of cbrain data), Claude watches for predicate candidates: recurring relationship patterns in content or queries that would benefit from graph traversal rather than prose search.

Common triggers include: user asks a question that would benefit from graph traversal ("who is responsible for X?", "what blocks Y?", "what depends on Z?"); user describes a recurring relationship pattern in content; user adds a tag or frontmatter field that names a relationship Claude has not seen before; Claude finds itself doing prose search to answer a question that a single SQL JOIN against tuples would answer if a predicate existed.

When perceived, Claude surfaces the candidate as a numbered Needs from you item with:

- Proposed predicate name (slug-style, e.g., `responsible_for`)
- Meaning (one sentence)
- Example tuple from current context
- Trigger that prompted the suggestion

Charles approves or declines; on approval, the predicate enters cbrain's predicate registry and extractor (`services/lib/extract.ts`).

**Cap surfacing to 3 candidates per chat.** Only surface when the trigger is concrete rather than speculative — if Claude isn't confident the pattern is real, skip it.

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

**Plans and disclosures.** `write_plan` creates a plan (Build Brief or otherwise); `update_plan_content` amends one and keeps revision history; `post_judgment_call` files a disclosure on the Comms Table at build close. All three take `plain_title`/`plain_summary` (disclosures also take `action_needed`) — set them every time per the naming rule above.

**Soft enforcement only.** Labels are never mandatory at write time — writes always succeed with or without them. A plan tagged `build-brief` that lands without `plain_title`/`campaign` gets a warning line back to the writing chat — not a rejection, and not a note routed to Charles.
