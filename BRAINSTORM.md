# Brainstorm Chat Protocol

**Read this when Charles opens a chat with "this is a brainstorm chat."** This document defines the entire workflow for firing a brainstorming run through the orchestrator. Follow it as written.

**Location:** `https://github.com/Chooch333/chat-protocol/blob/main/BRAINSTORM.md`

---

## What a brainstorm chat is

A brainstorm chat is a single-purpose chat that fires one run of the Brainstorm Orchestrator. The orchestrator runs Charles's brief past seven Claude Opus 4.7 agents in parallel — each interrogating it from a different framing — and produces a compiled output doc with all seven responses preserved verbatim. The chat's job is to take Charles's directive, validate it once, fire the run, and hand back the link.

Brainstorm chats are not for discussion of the topic, advice, or analysis. Charles brings a directive he has already reasoned through; the chat's role is to dispatch it and return the link.

The seven framings (locked in `lib/framings.ts` in the orchestrator repo, and visible in every output doc) are: constraints-first, user-first, alternatives-first, risk-first, simplicity-first, future-first, dependency-first. Do not mention them unless Charles asks — the orchestrator handles them.

---

## Workflow

The chat follows exactly these steps in order. Do not improvise.

### Step 1 — Receive the directive

Charles will provide a brief in his first or second message. Treat whatever he writes as the brief content verbatim. Do not edit it, summarize it, restructure it, or "improve" it. Charles has already reasoned this through with another chat; the brief is final on arrival.

The brief might be a short directive ("Should I sell the Palisade and replace it with a used minivan?") or a multi-paragraph framing. Either is fine. Do not ask clarifying questions about the topic itself — the seven agents will interrogate it.

### Step 2 — Confirm and propose a slug

Paraphrase the brief back in one sentence so Charles can catch a typo or misread before money is spent. Propose a `slug` for the brief — a short kebab-case identifier (e.g. `palisade-vs-minivan`, `ridgeworks-pricing-strategy`) that Charles will see in Project State. Slugs should be distinctive enough that a list of past brainstorms is scannable.

Ask Charles to confirm or amend. **One Needs from you, recommendation included.** Sample:

> Confirming: brainstorm whether to sell the Palisade and replace it with a used minivan, focused on practical day-to-day fit for the family. Slug: `palisade-vs-minivan`.
>
> ---
>
> ## Needs from you
>
> 1. **Confirm and fire?**
>    - **A.** Yes, fire it.
>    - **B.** Edits — specify what to change in the brief or slug.

Once Charles confirms, proceed. Do not fire without confirmation.

### Step 3 — Insert the brief (this auto-fires the run)

Use **Supabase MCP** `execute_sql` against project `drjtbqkizrqrwajofsea` to insert the brief:

```sql
insert into briefs (slug, content)
values ('<slug>', '<content>')
returning id, slug;
```

Capture the returned `id` — this is the brief's UUID.

**That's it for the trigger.** A Postgres trigger on the `briefs` table (`briefs_fire_orchestrator`) auto-fires the orchestrator's `/api/run` endpoint via `pg_net` as soon as the row lands. The chat does NOT need to call any HTTP endpoint, paste a URL, or do anything else to fire the run. The insert IS the fire.

Tell Charles the run is in flight:

> Brief inserted. The seven agents are running in parallel — should take about 60 seconds. I'll watch for completion and hand you the output link as soon as it's ready.

### Step 4 — Wait, then check Project State

The orchestrator takes 30–90 seconds to complete. After inserting, wait, then check Project State for the auto-logged completion note.

**Wait at least 45 seconds before the first check** — the orchestrator typically finishes in 45–60s, and checking earlier just burns tool calls.

Use **Project State MCP** `get_project_state` to check:

```
get_project_state(project_slug='brainstorm-orchestrator', recent_notes_limit=5)
```

Look in `recent_notes` for a note where:
- `topic` equals the slug from Step 3 (e.g. `palisade-vs-minivan`), AND
- `tags` includes `brainstorming-run`, AND
- `created_at` is after the time you inserted the brief

That note's `content` field has the output URL on the line starting with `Output:`.

If the note is not found on the first check, wait 20 more seconds and check again. Allow up to 4 polling attempts (about 2 minutes total wall time) before treating it as a failure. The orchestrator's hard ceiling is 5 minutes — but in practice runs finish well under 90 seconds, so a 2-minute window with multiple checks is generous.

### Step 5 — Hand back the link

Extract the output URL from the note content and hand it to Charles as a single clickable line. Do not summarize the seven framings, do not characterize the outputs, do not preview anything.

Sample response from the chat:

> Done. Read the compiled brainstorm here: https://brainstorm-orchestrator.vercel.app/output/<UUID>
>
> Auto-logged to Project State as note `<display_id>` for the Brainstorm Orchestrator project.

That ends the chat. The chat does not produce a Session Log, does not write to Project State directly (the orchestrator handles that), and does not invite further discussion of the topic.

### Step 6 — Failure handling

If the polling window expires without finding a note, something has gone wrong. Diagnose by querying the brief's status:

```sql
select status, completed_at,
  (select count(*) from agent_outputs where brief_id = b.id) as output_count,
  (select count(*) from agent_outputs where brief_id = b.id and status = 'complete') as complete_count,
  (select count(*) from agent_outputs where brief_id = b.id and status = 'failed') as failed_count
from briefs b where id = '<UUID>';
```

Possible states:

- **status='complete' but no Project State note.** Orchestrator finished but couldn't reach the Project State MCP. Hand Charles the output URL directly: `https://brainstorm-orchestrator.vercel.app/output/<UUID>`. Surface the missing-note as something to investigate later.
- **status='running'.** The run is still in flight; wait another 60 seconds and check again before declaring failure.
- **status='pending'.** The trigger didn't fire. Check whether `pg_net` is healthy in Supabase and whether `briefs_fire_orchestrator` is still attached to the table.
- **failed_count > 0.** Some agents errored. Hand Charles the output URL anyway (the doc shows error messages in place of failed framings) and surface which framings failed.

In all failure cases: surface clearly in chat, do not retry blindly. Charles will direct.

---

## Standing rules

- **No synthesis.** The chat never reads the compiled output, never summarizes the seven framings, never gives Charles a "highlights" version. The whole point of the orchestrator is that Charles reads the seven outputs himself. Synthesis is exactly what was excluded from the original Build Brief.

- **No follow-up brainstorms in the same chat.** One brainstorm per chat. If Charles wants another, he opens a new chat and says "this is a brainstorm chat" again. This keeps the trail in Project State clean and one-to-one with conversations.

- **No advice.** If Charles asks for the chat's opinion on the topic mid-flow ("what do you think — should I sell the Palisade?"), redirect: that's what the seven agents are for, fire the run and read what they say.

- **The chat protocol still applies.** All the engagement rules in `PROTOCOL.md` — one Needs from you per response, every choice with a recommendation, closed-choice questions, visual separator before Needs from you — apply during the brief-confirmation step. They just have less to govern because the workflow is so short.

- **No tokens or URLs in chat.** The trigger is fully automated via the Postgres trigger; Charles never needs to see, paste, or remember the run token or the run URL. The skill never asks Charles to share these.

- **Failures escalate.** If the SQL insert fails, polling expires, the orchestrator response shows agent failures, or anything else goes off-script — pause and surface it in chat per Step 6. Do not retry blindly. Charles will direct.

---

## Appendix — Reference values

For Claude's reference during execution. Charles does not need to read this.

- **Orchestrator app:** `https://brainstorm-orchestrator.vercel.app`
- **Repo:** `Chooch333/brainstorm-orchestrator`
- **Vercel project ID:** `prj_ZEv509FX2ZvkzPDfXyg3rRogj6aY`
- **Vercel team:** `team_8nMi0Bd6orQHGeTrMZYWCamm` (slug `chooch333s-projects`)
- **Supabase project ID:** `drjtbqkizrqrwajofsea`
- **Supabase tables:** `briefs` (insert here — trigger auto-fires the run), `agent_outputs` (read-only from this protocol), `orchestrator_config` (holds the run endpoint URL and token; do not query unless debugging)
- **Postgres trigger:** `briefs_fire_orchestrator` on `public.briefs`, calls `public.fire_orchestrator_run()`, uses `pg_net` for async HTTP
- **Project State project slug:** `brainstorm-orchestrator`
- **Run endpoint (auto-fired by trigger; do not call manually):** `GET /api/run?briefId=<UUID>&token=<RUN_TOKEN>`
- **Output endpoint:** `GET /output/<UUID>` (rendered HTML, no auth needed beyond knowing the UUID)
- **Diagnostics (RUN_TOKEN-protected):** `/api/debug-projectstate`, `/api/test-note-log?briefId=<UUID>`
