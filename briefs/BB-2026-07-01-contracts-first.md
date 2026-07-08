# Build Brief — BB-2026-07-01-contracts-first

> **STATUS: EXECUTED — closed 2026-07-07.** E-267 closed; Session Log logged (E-277 gated_by reconciliation, E-278 eval-discipline learning, E-279 snapshot carrying two BB-5 scope additions). Committed here as the canonical record. As executed: `gated_by` documented as **directional**, with E-247's "symmetric" reconciled on the record as any-to-any kind combinations + bidirectional query (SCHEMA.md changelog note); WATCHER_CONTRACT §4/§8 "reserved" wording reconciled against BRIEF_SCHEMA v0.5.0 (PM intents specified, authored by the Reasoner, not the watcher's live four) — ratified by Charles.

**What this is:** Amend the three contracts so every decided-but-undocumented piece of the email-intake design exists on paper before any code moves. Sequence item 1 of 5.

**What I'll do (receiving chat):** Commit amended versions of SCHEMA.md, BRIEF_SCHEMA.md (v0.5.0), and WATCHER_CONTRACT.md to `Chooch333/cbrain` / `Chooch333/email-watcher` respectively, verify each commit by re-reading, and close with a Session Log on `email-interface-build`.

**What you'll do:** Approve this Brief, paste the prompt into a fresh chat, review each contract diff before commit.

---

## Current state going in

The workbook model (E-246–E-250), the PM intent types, the confidence seed mechanism, and mention anchors are all decided but absent from every live contract (verified 2026-07-01 against live SCHEMA.md and BRIEF_SCHEMA v0.4.0). Contract-first is the house rule: contracts change first, code follows. Nothing in this Brief touches code.

## Receiving chat

New chat to be opened.

## Scope

**In scope:**

1. **SCHEMA.md** —
   - Add `pm-workbook` to the type enum. One per project at `projects/<slug>/pm/workbook.md`.
   - Document the node shape: `node_id` (stable, workbook-scoped, never reused), `kind: task|issue`, `title`, `status: open|closed` (flips in place, nothing relocates — E-250), `duration_days`/`buffer_days` (tasks), `topic` (free-text label), `assertions` (append-only dated list, each with `at`, `basis {origin, source_brief}`, `text`, `effect: reinforce|weaken|contradict|neutral`), `decision` (issues only, set at close).
   - State explicitly: due dates are never stored — always derived by walking `gated_by` backward from `pm-milestones` (E-248). No confidence field is ever stored — current confidence is computed from the assertion stream.
   - Document node addressing in tuples: subject/object may be `<project-slug>-workbook#<node_id>`.
   - Document five new predicates in the registry section: `gated_by` (directional, any-node → any-node, all four kind combinations legal), `raci_r`, `raci_a`, `raci_c`, `raci_i` (each directional, node → person).
   - Document a `lexicon_terms` list field on workbook nodes (declared in git, extracted to Supabase on sync — same pattern as `relationships` → tuples).
   - Add the machine-readable guidance stanza for `pm-workbook` (meaning / propose_when / key_fields), keeping the enum and stanza block in sync.
2. **BRIEF_SCHEMA.md → v0.5.0** —
   - Specify `record_pm_item` (no longer reserved): target = `project_slug`, `kind`, `title`, `topic?`, `duration_days?`, `buffer_days?`, `raci {r?, a?, c?, i?}` (person slugs), `gates` (list of `{direction: gated_by|gates, node_ref}` where node_ref is an existing node_id or another intent_id in the same brief), `milestone_ref?`, `seed`.
   - Add new intent type `append_pm_assertion`: target = `project_slug`, `node_id`, `text`, `effect`, `status_change?: close|reopen`, `decision_text?`. Vocabulary goes from five types to seven.
   - Add the `seed` block spec: `basis` (stated|inferred|steer|gate), `match_basis` (exact-id|mechanical|semantic|none), `corroboration` (cross-method|single-method|none), `seed_value` derived from a fixed lookup table published inside the schema. Draft the table (start: stated+exact-id+cross-method=0.9 down to inferred+semantic+none=0.3; receiving chat proposes the full grid for gate review). Mark the old `high|medium|low` enum as deprecated-but-accepted until producers migrate — do not break the deployed watcher's output.
   - Reword the correction rule for the additive model (E-237): a correction is a new brief whose filed assertions are added alongside the old, timestamped; sealed briefs stay immutable, but filed facts never overwrite.
   - Retire the `implied-pm-item` flag type with a note: replaced by the real intents as of v0.5.0 (producers may still emit it until migrated).
   - Add one paragraph to Purpose: the gate is the current truth anchor; the strategic direction is gateless with three relocated anchors (E-236), so contracts and strategy stop silently disagreeing.
3. **WATCHER_CONTRACT.md** (repo `email-watcher` or wherever it lives in cbrain — verify path first) —
   - Add per-mention anchor IDs to the email-record/frame schema: people/orgs only (E-229), stable `m-` IDs marking occurrences, never identities.
   - Amend the steer section per the reconciled rule: steer present in the frame for all passes; Harvester may use it as a matching context clue; PM ignores it; the Reasoner is sole consumer and sole `steer_consumed` reporter.

**Out of scope:** any code change; predicate registration in extract.ts (BB-storage-layer); the Archivist skill (BB-archivist-rewrite); creating any workbook file. Pre-answered forks: RACI = four predicates (no attribute column exists in tuples); two PM intents, not one; node-ref convention, not a separate edge table.

## Directive

For each of the three contracts: fetch the live file via Custom GitHub MCP (`get_file_contents`, owner=Chooch333), draft the amendment, show Charles the changed sections in chat for approval, commit with a fresh SHA fetched immediately before the write, then re-read the committed file to verify integrity. Update each contract's changelog. Close with a Session Log referencing this Brief ID.

## Inputs

- Live contracts: `Chooch333/cbrain` → `contracts/SCHEMA.md`, `contracts/BRIEF_SCHEMA.md`, `contracts/WATCHER_CONTRACT.md` (verify actual location of the watcher contract before editing)
- Design source: the consolidated design doc (email-intake-consolidated-design.md, 2026-07-01), Parts 3–4
- Decisions: E-229, E-236, E-237, E-246–E-250 in Project State `email-interface-build`

## Pasteable prompt

> Read PROTOCOL.md from Chooch333/chat-protocol first, then load Project State for `email-interface-build`. This chat executes Build Brief BB-2026-07-01-contracts-first (pasted below / attached). It is a contracts-only chat: amend SCHEMA.md, BRIEF_SCHEMA.md (v0.5.0), and WATCHER_CONTRACT.md per the Brief's In-scope list. No code changes. Fetch each live file first, show me the changed sections before committing, re-fetch SHA immediately before each write, and re-read after each commit to verify. If anything in the Brief conflicts with current Project State, escalate before following the Brief. Close with a Session Log on `email-interface-build` referencing the Brief ID.
