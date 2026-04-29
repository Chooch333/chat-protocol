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

The brief might be a short directive ("Should I sell my Hyundai Palisade and replace it with a used minivan?") or a multi-paragraph framing. Either is fine. Do not ask clarifying questions about the topic itself — the seven agents will interrogate it.

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

### Step 3 — Fire the run

Use **Supabase MCP** `execute_sql` against project `drjtbqkizrqrwajofsea` to insert the brief:

```sql
insert into briefs (slug, content)
values ('<slug>', '<content>')
returning id, slug;
```

Capture the returned `id` — this is the brief's UUID.

Then trigger the orchestration. The orchestrator endpoint is:

```
https://brainstorm-orchestrator.vercel.app/api/run?briefId=<UUID>&token=<RUN_TOKEN>
```

Where `<RUN_TOKEN>` is the run token Charles set during the orchestrator's deployment. **Charles will paste this URL into his browser to fire the run** — the chat does not fire it directly. (Reason: the chat sandbox cannot reach Vercel deployment-protected URLs from outside Charles's authenticated browser session, and the orchestrator's run endpoint is currently behind that protection.)

Hand the URL to Charles in chat with these instructions:

> Brief inserted. Paste this URL into your browser tab and let it run for 30–90 seconds:
>
> ```
> https://brainstorm-orchestrator.vercel.app/api/run?briefId=<UUID>&token=<RUN_TOKEN>
> ```
>
> When the JSON response loads, paste it back to me.

Wait for Charles to paste the JSON response.

### Step 4 — Hand back the link

When the JSON response comes back, verify `successCount` equals `totalCount` (both should be 7). The response includes an `outputUrl` field — that's the rendered compiled doc. Hand it back as a single clickable line. Do not summarize the seven framings, do not characterize the outputs, do not preview anything.

Sample response from the chat:

> Done. Read the compiled brainstorm here: https://brainstorm-orchestrator.vercel.app/output/<UUID>
>
> A note has also been auto-logged to the Brainstorm Orchestrator project in Project State, so you can find your way back to this URL later from the dashboard.

If `successCount` is less than `totalCount`, surface that — name which framings failed and their error messages from the `summary` field — and ask whether to investigate or accept partial.

That ends the chat. The chat does not produce a Session Log, does not write to Project State directly (the orchestrator handles that), and does not invite further discussion of the topic.

---

## Standing rules

- **No synthesis.** The chat never reads the compiled output, never summarizes the seven framings, never gives Charles a "highlights" version. The whole point of the orchestrator is that Charles reads the seven outputs himself. Synthesis is exactly what was excluded from the original Build Brief.

- **No follow-up brainstorms in the same chat.** One brainstorm per chat. If Charles wants another, he opens a new chat and says "this is a brainstorm chat" again. This keeps the trail in Project State clean and one-to-one with conversations.

- **No advice.** If Charles asks for the chat's opinion on the topic mid-flow ("what do you think — should I sell the Palisade?"), redirect: that's what the seven agents are for, fire the run and read what they say.

- **The Run Token is in Charles's head, not the protocol.** The skill never asks Charles to share or expose the token. Charles either remembers it or has it saved somewhere he controls. If he's lost it, he can find it in the Vercel project settings under environment variable `RUN_TOKEN`.

- **The chat protocol still applies.** All the engagement rules in `PROTOCOL.md` — one Needs from you per response, every choice with a recommendation, closed-choice questions, visual separator before Needs from you — apply during the brief-confirmation step. They just have less to govern because the workflow is so short.

- **Failures escalate.** If the SQL insert fails, the URL paste-back returns an error, the orchestrator response shows agent failures, or anything else goes off-script — pause and surface it in chat. Do not retry blindly. Charles will direct.

---

## Appendix — Reference values

For Claude's reference during execution. Charles does not need to read this.

- **Orchestrator app:** `https://brainstorm-orchestrator.vercel.app`
- **Repo:** `Chooch333/brainstorm-orchestrator`
- **Vercel project ID:** `prj_ZEv509FX2ZvkzPDfXyg3rRogj6aY`
- **Vercel team:** `team_8nMi0Bd6orQHGeTrMZYWCamm` (slug `chooch333s-projects`)
- **Supabase project ID:** `drjtbqkizrqrwajofsea`
- **Supabase tables:** `briefs` (insert here), `agent_outputs` (read-only from this protocol)
- **Project State project slug:** `brainstorm-orchestrator`
- **Run endpoint:** `GET /api/run?briefId=<UUID>&token=<RUN_TOKEN>`
- **Output endpoint:** `GET /output/<UUID>` (rendered HTML, no auth needed beyond knowing the UUID)
- **Diagnostics (RUN_TOKEN-protected):** `/api/debug-projectstate`, `/api/test-note-log?briefId=<UUID>`
