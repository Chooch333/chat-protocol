# Build Brief — BB-2026-07-14-shelf-canon

**Git home:** `Chooch333/chat-protocol` → `docs/design/BB-2026-07-14-shelf-canon.md`
**Skills:** none beyond standard build-chat protocol reading
**Status:** queued | **Authored:** 2026-07-14 DA session | **Effort:** S | **Cost:** $0

**What this is:** Make PROTOCOL.md the single, sufficient source of truth for how any chat puts a Build Brief "on the shelf" — and retire the two conflicting decisions that point at the old shelving model.

**What I'll do (receiving chat):** Three edits (two doc edits, one Project State supersession), then verify each by reading back the live file/record.

**What you'll do:** Nothing mid-build. Review the fork log and verification results after.

---

## Current state going in

[verified 2026-07-14] The shelf's consumer side is clear: `orchestrate-build` pulls the oldest Project State plan with status `queued`. The DA producer side is clear: DA SKILL v0.2.2 step 10 commits the brief to the target repo's `docs/design/` and sets the plan to `queued`. But:

1. PROTOCOL.md's Build Brief section predates the shelf. It says a brief "does not write to the database on creation" and describes only paste-into-sub-chat handoff. A non-DA chat reading PROTOCOL.md cannot shelve a brief.
2. Live decisions E-280 and E-285 (email-interface-build, 2026-07-08) say briefs live in `chat-protocol/briefs/` and explicitly rejected Project State plans as a home. Nothing supersedes them.
3. No document states which Project State project a brief's plan is filed under.

Charles resolved both forks 2026-07-14 **[Charles]**: (a) canonical git home is the **target repo's `docs/design/`**, with `chat-protocol/briefs/` as fallback only when the build has no target repo; (b) the plan is filed on the **target project's** Project State project.

## Receiving chat

New build chat. Fetch this brief from the git home above via Custom GitHub MCP `get_file_contents` before doing anything.

## Scope

**In scope:**
1. **PROTOCOL.md edit** (`Chooch333/chat-protocol`, path `PROTOCOL.md`) — replace the stale sentence and insert the Shelving block (exact text in Inputs). Effort S.
2. **Completeness framework edit** (`Chooch333/agent-library`, path `roles/design-assist/references/brief-completeness-framework.md`) — replace the hedged "Durable home" bullet with the canonical text (in Inputs). Effort S.
3. **Supersede E-280 and E-285** on project `email-interface-build` via Project State `supersede_decision`, each replaced by the new-canon decision text (in Inputs). Effort S.

**Out of scope:** editing `orchestrate-build/SKILL.md` or DA SKILL.md (already consistent with the new canon); moving the five existing briefs out of `chat-protocol/briefs/` (they are grandfathered; their pasteable prompts already carry correct fetch paths); any dashboard changes.

**Pre-answered forks:** git home = `docs/design/` primary / `chat-protocol/briefs/` fallback [Charles]; plan lives on target project [Charles]; plan content = full brief text with git fetch path in the header, so the queue entry is self-sufficient and the git copy is the human-readable review surface [Claude-per-doctrine — matches DA snapshot rule and framework's fetch-path rule].

## Directive

1. Custom GitHub MCP `get_file_contents` on PROTOCOL.md (repo `chat-protocol`) to confirm current text, then `replace_in_file` twice:
   - Replace the sentence `A Build Brief does not write to the database on creation. It's a handoff payload — a package of context and directive that lets a sub-chat execute without re-negotiating the main chat's decisions.` with the Edit-1a text in Inputs.
   - Insert the Shelving block (Edit-1b in Inputs) immediately after the "Protocol defaults for Build Briefs" bullet list, before "Build Brief rules."
2. `replace_in_file` on the framework file (repo `agent-library`): replace the current Durable-home bullet with Edit-2 text in Inputs.
3. Project State `supersede_decision` on E-280, then E-285 (project `email-interface-build`), new decision per Edit-3 in Inputs, change_reason: "Shelf canon shipped 2026-07-14 (BB-2026-07-14-shelf-canon): docs/design/ + target-project plan replaces chat-protocol/briefs/-only model."
4. **Verify:** re-fetch both edited files and read the changed sections back; `get_decision_chain` (or re-search) on E-280/E-285 to confirm superseded status. Write-before-done applies.
5. Close: flip this brief's plan to `succeeded`, write the Session Log per protocol (include the fork log, even if empty).

## Acceptance criteria

- A fresh chat given ONLY PROTOCOL.md can answer: where does a brief's file go, where does its plan go, what status puts it on the shelf, and what the plan/prompt must carry (the git fetch path). Test by reading the new section cold.
- E-280 and E-285 no longer surface as live decisions; their replacement names the new canon.
- Framework durable-home bullet matches PROTOCOL.md exactly in substance.
- Verification run: the re-fetched live files contain the new text verbatim.

## Inputs

**Edit 1a — replacement sentence for PROTOCOL.md** [draft, Charles-approved direction]:

> A Build Brief is both a handoff payload and a queue entry. At authoring it is committed to git and written to Project State as a plan (see Shelving a brief, below) — it does not otherwise write Decisions, Next steps, or other buckets on creation.

**Edit 1b — new PROTOCOL.md block** [draft]:

> **Shelving a brief (the build queue).** The shelf is Project State: **plans with status `queued`**. The orchestrator pulls the oldest queued plan on a project. Every Build Brief gets three things in the same turn it's authored:
>
> 1. **Git home.** Commit the brief to the target project's repo at `docs/design/BB-YYYY-MM-DD-slug.md`. If the build has no target repo, use `chat-protocol/briefs/` instead.
> 2. **Plan.** `write_plan` on the target project's Project State project. Content is the full brief text; the brief header carries the git fetch path (owner, repo, path) so any chat can find the readable copy.
> 3. **Status.** When the brief passes the completeness gate and is build-ready, `update_plan_status` → `queued`. Until then it stays `draft`. Queued means on the shelf; nothing else does.
>
> Interactive handoff (pasting a prompt into a sub-chat) still works — but the pasteable prompt instructs the sub-chat to fetch the brief from its git home rather than trust pasted text. The plan is the authoritative queue entry; the git copy is the durable, human-readable one. Keep them in sync via `update_plan_content` when a brief is amended.

**Edit 2 — framework durable-home bullet replacement** [draft]:

> - **Durable home:** the brief is committed to the target project's repo at `docs/design/` at authoring (`chat-protocol/briefs/` only when no target repo exists), AND written as a Project State plan on the target project with the git fetch path in its header (E-280 superseded 2026-07-14; E-282 fetch-path rule kept).

**Edit 3 — superseding decision text** [draft]: title "Build Briefs live in the target repo's docs/design/ + a queued Project State plan"; rationale: "Canon per BB-2026-07-14-shelf-canon: git home is the target repo's docs/design/ (chat-protocol/briefs/ fallback for repo-less builds); the brief is also written as a Project State plan on the target project (full text, git path in header) and set to queued when build-ready — the queue, not the repo folder, is the shelf. Replaces the chat-protocol/briefs/-only model and its rejection of plan storage, which predates the canonical plan lifecycle."

**Reference files:** PROTOCOL.md (sha a3d5b32 at authoring), brief-completeness-framework.md (sha c5073c9 at authoring). Staleness rule applies: reconcile against live files at start; if the anchor sentences have changed, adapt the replace targets and log the fork.

## Completeness table

| Domain | State | Where |
|---|---|---|
| 1 Purpose & users | Answered | What this is / Current state (protocol infrastructure; users = every future chat) |
| 2 Acceptance criteria | Answered | §Acceptance |
| 3 Runtime & execution | N/A | Doc + DB edits via MCP in-chat; nothing runs |
| 4 Data — schema | N/A | No schema change |
| 5 Data — storage & recall | Answered | §Inputs (git markdown + Project State decisions/plans) |
| 6 Interconnectivity | Answered | §Directive (Custom GitHub MCP, Project State MCP; halt if Project State unreachable) |
| 7 Access & hard gates | Answered | Repos chat-protocol + agent-library, project email-interface-build only; no credentials, no money, no destructive change (supersession preserves history) |
| 8 Skills & tools | Answered | Header + §Directive |
| 9 Sequence & dependencies | Answered | §Directive order; no upstream builds |
| 10 Assumptions & risks | Answered | Only risk: replace_in_file anchor drift → staleness rule in §Inputs |
| 11 Design intent narrative | Answered | Below |

**Design intent:** One document (PROTOCOL.md) must be sufficient for shelving; everything else defers to it. When trade-offs appear, prefer redundancy (state the rule in full rather than cross-reference) and never leave the two homes ambiguous — ambiguity here is the exact defect being fixed. Write for a non-technical reader first.

## Pasteable prompt

> This is a build chat. Fetch your Build Brief via Custom GitHub MCP get_file_contents (owner=Chooch333, repo=chat-protocol, path=docs/design/BB-2026-07-14-shelf-canon.md). Also read PROTOCOL.md per standing rules. Execute the brief's Directive, verify per its acceptance criteria, and close out per the brief (plan status + Session Log).
