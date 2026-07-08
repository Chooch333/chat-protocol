# Build Brief — BB-2026-07-01-archivist-rewrite

> **AMENDMENTS 2026-07-08** (from the review, approved by Charles):
> 1. **Resequenced from item 4 to item 3** — now runs BEFORE BB-upstream-passes. Its stated dependencies were always only contracts + storage, and the reorder breaks a circular dependency: the upstream-passes PM increment needs a stub workbook filed through this rewritten Archivist.
> 2. Added a **live PM-filing verification** via the gate path (`seal_brief` `added_intents`) — this both verifies the new PM filing rules against a real producer AND creates the stub workbook nodes BB-upstream-passes needs. Two problems, one run.
> 3. Filing failures get a concrete surface: a **`FILING-ERROR-<brief_id>.md` sidecar in `_briefs/`**.
> 4. The rewritten skill **accepts deprecated v0.4.0 briefs** (`high|medium|low`, `implied-pm-item`) unchanged until producers migrate.
> 5. Write-back clarifier: the fingerprint write-back **never touches `frame.json`** (E-228 immutability) — it stamps the thread-fingerprint record only.
> 6. **Field names in committed BRIEF_SCHEMA v0.5.0 supersede any names in this Brief** where they differ (e.g. `node_ref`/`assertion`/`decision` are the contract's names).

**What this is:** One rewrite of the Archivist skill covering three decided changes — additive filing, the two new PM filing rules, and the fingerprint write-back — plus closing the load-bearing gap: an automated filing trigger after seal (blocker E-239). Sequence item 3 of 5. Depends on BB-contracts-first (executed 2026-07-07) and BB-storage-layer.

**What I'll do (receiving chat):** Rewrite `roles/archivist/SKILL.md` in `Chooch333/agent-library`, build the seal-triggered filing function, and verify with a live end-to-end filing run including PM intents.

**What you'll do:** Approve this Brief, paste into a fresh chat, review the rewritten skill before commit, approve one real brief at the gate for the live verification run — adding the PM test intents via `added_intents` at the seal.

---

## Current state going in

Verified from Project State and the live protocol docs (2026-07-01): the Archivist skill still enforces the supersede model E-237 reversed — a build chat reading the live file today gets instructions that contradict the decided data model. `record_pm_item` is "surface and stop" in the skill. The automated path has no filing trigger (E-239): in chats Charles says "file this brief"; the pipeline has no equivalent, so sealed briefs would never become cbrain data. The Archivist has also never completed a real filing run on a live approved brief — that milestone lands inside this Brief's verification.

## Receiving chat

New chat to be opened.

## Scope

**In scope:**

1. **Skill rewrite — additive filing.** Replace supersede-never-edit with: every filed fact is an addition. New assertions append to entity bodies / node assertion streams with timestamp + `brief_id` + basis; a correction files alongside the fact it corrects, carrying `effect: contradict`; nothing is ever edited in place except the two sanctioned in-place moves — flipping a node's `status` (E-250) and clearing a resolved `org_unverified_domain` (existing contract behavior). The append-only, structurally-forbidden-from-overwriting character of the Archivist is preserved and strengthened.
2. **Skill rewrite — PM filing rules.** Filing SOP for the two new intents:
   - `record_pm_item` → append a node to the project's `pm-workbook` file (create the file from the pm-family pattern if it's the project's first item), assign the next `node_id`, write RACI edges and `gates` edges as node-level relationship declarations (the sync extracts them to tuples), stamp `brief_id` and the seed as the first assertion. Intent-to-intent `node_ref`s resolve in filing order within the brief.
   - `append_pm_assertion` → append the dated assertion to the named node; apply `status_change` by flipping status in place; record the decision text on an issue close. Missing node ref = stop and surface, never guess.
   - **Legacy acceptance:** the skill files deprecated v0.4.0-format briefs (`high|medium|low` confidence, `implied-pm-item` flags) unchanged until producers migrate — the inline reasoning pass keeps emitting them until BB-pass4-reasoner cuts over.
3. **Skill rewrite — write-back step (E-234).** After filing, stamp each filed outcome's cbrain id + thin label back onto the source email's thread fingerprint. Best-effort; a failed write-back is logged, never blocks the filing. The write-back never touches `frame.json` (E-228 immutability) — the thread-fingerprint record only.
4. **Collision and pre-flight rules carry forward unchanged:** files only `status: approved` with a `brief_id`; stops cold on drafts; create-on-existing-slug surfaces, never overwrites; files verbatim, never re-judges truth.
5. **Auto-trigger.** On `seal_brief` success, enqueue/invoke a filing function (Vercel, in the cbrain-sync or email-watcher project — receiving chat proposes which with a recommendation) that runs the Archivist skill against the sealed file. Seal is the trigger; nothing new for Charles. **Failure surface:** on filing failure the trigger writes a `FILING-ERROR-<brief_id>.md` sidecar into `_briefs/` carrying the error — the sealed brief stays intact, and any folder listing shows the failure. Never silent.
6. **Verification — the real close-the-loop milestone, now covering PM filing too.** One live run end to end: a real forwarded email → draft brief → at the gate, Charles seals **adding two test PM intents via `added_intents`** (one `record_pm_item` creating a node on a real project's workbook — IMI or C92 per Charles's pick — and one `append_pm_assertion` against it, or a second `record_pm_item` with a `gates` edge between the two) → auto-trigger fires → Archivist files → verify in cbrain (entity written, workbook file created with nodes, `brief_id` stamped, seed as first assertion, tuples live including the `gates` edge and RACI, assertion stream correct) → fingerprint write-back confirmed. This is the pipeline's first real filing run AND the first live exercise of the PM filing rules with a real producer (the gate is a legitimate producer — `basis: gate`). The filed nodes double as the stub workbook BB-upstream-passes' PM increment verifies against. Nothing closes without this run.

**Out of scope:** removing the gate (it stays in the automated path, E-236 — the trigger fires *after* seal); Pass 4 (the current inline reasoning pass's briefs are the test input); any change to seal_brief/reject_brief beyond emitting the trigger. Pre-answered fork: one skill rewrite covering all three changes, not three sequential edits.

## Directive

Read the live SKILL.md and the seal code (`services/lib/seal.ts`) in full first. Draft the rewritten skill and present the changed sections in chat before committing. The skill is prose instructions an agent follows — write it in the skill's existing voice and structure. Build the trigger second, verify third. If the live skill contains rules this Brief doesn't account for, escalate — don't silently drop them.

## Inputs

- `Chooch333/agent-library` → `roles/archivist/SKILL.md` (live read required)
- `Chooch333/cbrain` → `services/lib/seal.ts`, `services/api/mcp.ts`
- BRIEF_SCHEMA v0.5.0 as committed 2026-07-07 (the intent shapes being filed — contract field names win)
- Design doc Part 3 G3/G7/G8, Part 2 C3
- Blocker E-239 (resolve it in the closing Session Log)

## Pasteable prompt

> Read PROTOCOL.md from Chooch333/chat-protocol first, then fetch this Brief from Chooch333/chat-protocol → briefs/BB-2026-07-01-archivist-rewrite.md and load Project State for `email-interface-build`. This chat executes that Brief — note the amendment block at the top, especially the resequencing (this now runs BEFORE upstream-passes) and the PM-intent verification via added_intents. Confirm BB-contracts-first and BB-storage-layer are landed first; escalate if not. Read the live archivist SKILL.md and seal.ts in full before drafting. Show me the rewritten skill sections before committing. The Brief closes only on the live end-to-end filing run — I'll forward the test email, pick the target project for the PM test nodes, and seal at the gate when you're ready. Session Log resolves blocker E-239.
