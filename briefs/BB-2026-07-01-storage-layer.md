# Build Brief — BB-2026-07-01-storage-layer

> **AMENDMENTS 2026-07-08** (from the review + BB-1 execution, approved by Charles):
> 1. Added scope item 8 — **node-text embeddings on sync** (was a "maybe" mid-BB-upstream-passes; it's known-missing, so it lands here where the sync code is already open).
> 2. Added the **test-fixture sanction** to the verification round-trip: direct commits allowed only under a dedicated test project slug, fully retracted after.
> 3. `gated_by` registers **directional** — the E-247 "symmetric" wording is reconciled on the record in SCHEMA.md's changelog (any-to-any kinds + bidirectional query; stored edge directional per E-248 mechanics). Read that changelog note before registering.
> 4. **Field names in committed BRIEF_SCHEMA v0.5.0 / SCHEMA.md supersede any names in this Brief** where they differ.

**What this is:** The keystone storage change: make the tuples layer bi-temporal (history-preserving), register the five new predicates, add node-level graph extraction, stand up the lexicon tables, and embed workbook node text. Sequence item 2 of 5. Depends on BB-2026-07-01-contracts-first (executed 2026-07-07).

**What I'll do (receiving chat):** Run two Supabase migrations, rewrite the tuple reconciler in the sync code, register predicates, add node/lexicon extraction and node embeddings, deploy, and verify against a real entity round-trip.

**What you'll do:** Approve this Brief, paste into a fresh chat, approve the migration SQL and the reconciler design before they run, spot-check the verification round-trip.

---

## Current state going in

Verified 2026-07-01: `services/lib/extract.ts` reconciles tuples by subject-scoped delete-then-insert — every sync run erases when an edge was learned, which makes the additive/temporal model (E-237) impossible at the graph layer. The predicate registry has no `gated_by` and no RACI predicates. No lexicon storage exists anywhere. The contracts describing all of this landed in BB-contracts-first (committed 2026-07-07); this Brief makes the storage match them.

## Receiving chat

New chat to be opened.

## Scope

**In scope:**

1. **Migration A — bi-temporal tuples** (Supabase MCP `apply_migration`, project ref `lpeswznkxzeeyiqaewma`): add `asserted_at` (timestamp), `source_brief` (text, nullable), `retracted_at` (timestamp, nullable — null means live) to the `tuples` table. Backfill existing rows: `asserted_at` = row creation time if available, else migration time; `retracted_at` = null.
2. **Migration B — lexicon tables**: `lexicon_terms` (`id`, `project_slug`, `term`, `normalized_term`, `status: active|retired`, `source: derived|curated`, `added_at`, `source_brief` nullable) and `lexicon_term_nodes` (`term_id`, `node_ref`). `node_ref` uses the `<project-slug>-workbook#<node_id>` convention.
3. **Reconciler rewrite** (`services/lib/extract.ts` in `Chooch333/cbrain`): replace delete-then-insert with reconcile — declaration present + no live row → insert with `asserted_at`; declaration removed → set `retracted_at`, never delete; unchanged → untouched. Same logic for entity-level and node-level declarations.
4. **Predicate registration** in extract.ts: `gated_by` (**directional** — see amendment note 3 above), `raci_r`, `raci_a`, `raci_c`, `raci_i` (directional). Unknown predicates stay skipped-not-guessed.
5. **Node-level extraction**: teach the sync to walk a `pm-workbook` file's `nodes` list and extract (a) each node's `relationships` into tuples using the node-ref subject convention, and (b) each node's `lexicon_terms` list into the two lexicon tables (same reconcile semantics — a removed term is retired, not deleted).
6. **`graph_query` default**: return only live rows (`retracted_at is null`); add an opt-in flag for history. Verify whether graph_query lives in `services/api/mcp.ts` and adjust there.
7. **Verification round-trip** (decided ≠ wired ≠ verified-working): create a test `pm-workbook` file with two nodes, a `gated_by` edge, one RACI edge, and two lexicon terms; sync; query tuples and lexicon tables; remove one declaration; re-sync; confirm the row is retracted with a timestamp, not gone. Delete the test file after, re-sync, confirm full retraction. **Test-fixture sanction:** this direct commit is allowed only under a dedicated test project slug (e.g. `_test-fixture`) — never a real project — created and fully retracted inside the verification. Real-project workbook content always goes through the brief/gate path.
8. **Node-text embeddings** (new, 2026-07-08): the sync embeds each workbook node's text (title + topic at minimum; receiving chat proposes exact composition with a recommendation) into pgvector alongside node extraction, keyed by node-ref, reconciled on change. The PM pass (BB-upstream-passes) does embedding top-k against these vectors — without this, that pass has no target. Verify in the round-trip: the two test nodes have vectors after sync; retracted node's vector is retired/excluded from query.

**Out of scope:** the derivation job that proposes lexicon terms from node text (deferred to after BB-5, once real workbook content exists to derive from — log as a next step); any pass code; the Archivist; rebuild_index changes. Pre-answered forks: node-ref convention in the one tuples table (no separate edge table); retire-don't-delete everywhere.

## Directive

Read the live extract.ts and mcp.ts in full before proposing anything. Present both migrations as SQL in chat for approval before running. For the reconciler: this is exactly the kind of edit where `replace_in_file` fails on `$`-laden TypeScript — use whole-file overwrite with local tsc verification. Deploy via the existing path (verify how cbrain-sync deploys — Vercel project `prj_cWxtbxXu5JclOyPEzmmQTaY6mAR0`), confirm deploy green via Vercel MCP, then run the verification round-trip. Nothing closes on deploy-green alone; the round-trip is the close condition.

## Inputs

- `Chooch333/cbrain` → `services/lib/extract.ts`, `services/api/mcp.ts`
- Supabase project `lpeswznkxzeeyiqaewma`; Vercel `cbrain-sync` project + team `team_8nMi0Bd6orQHGeTrMZYWCamm`
- Amended SCHEMA.md and BRIEF_SCHEMA.md v0.5.0 as committed 2026-07-07 (the contract text supersedes this Brief on field names)
- Design doc Parts 3 G2/G4/G5

## Pasteable prompt

> Read PROTOCOL.md from Chooch333/chat-protocol first, then fetch this Brief from Chooch333/chat-protocol → briefs/BB-2026-07-01-storage-layer.md and load Project State for `email-interface-build`. This chat executes that Brief. Confirm BB-2026-07-01-contracts-first's commits are live in SCHEMA.md and BRIEF_SCHEMA.md (v0.5.0) before starting — if they aren't, stop and escalate. Note the amendment block at the top of the Brief, including the E-247/directional reconciliation recorded in SCHEMA.md's changelog. Read services/lib/extract.ts and services/api/mcp.ts in full first. Present migration SQL and the reconciler design for my approval before executing. Close only after the verification round-trip in the Brief passes, then write a Session Log referencing the Brief ID.
