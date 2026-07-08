# Build Brief — BB-2026-07-01-pass4-reasoner

> **AMENDMENTS 2026-07-08** (from the review + BB-1 execution, approved by Charles):
> 1. Prior-Brief order is now **contracts → storage → archivist → upstream-passes** (archivist and upstream-passes swapped); this Brief still depends on all four.
> 2. **Fallback wording + seed fixed:** on a reasoning fault the pass *authors a degraded brief* containing harvester.json's New claims as bare `create_entity` intents (it never files anything); fallback intents carry `seed`: basis `inferred`, match_basis `none`, corroboration `none`.
> 3. **Scope addition (E-279 carryover):** at cutover, update WATCHER_CONTRACT §4/§8's live-vocabulary wording — the "authored by the downstream Reasoner, not the watcher's current live four" line goes stale the moment Pass 4 ships inside the watcher. Reframe by role, not deployment state.
> 4. **Cron/transition safety:** pause the cron (or feature-flag) during the cutover deploy-and-verify window; re-process paused mail after.
> 5. **Field names in committed BRIEF_SCHEMA v0.5.0 supersede any names in this Brief** where they differ.
> 6. Check E-279's snapshot for any further BB-5 scope additions carried out of BB-1 before starting.

**What this is:** Build the Reasoner — the pipeline's only interpretive pass — replacing the current inline Layer 2 reasoning with a pass that reads frame + both overlays + read-only cbrain and authors the full v0.5.0 brief, including the two PM intents and the mechanically derived confidence seed. Sequence item 5 of 5. Depends on all four prior Briefs.

**What I'll do (receiving chat):** Implement Pass 4 in `Chooch333/email-watcher` per the Part 4 spec, cut the pipeline over from the old inline reasoning pass, and verify on real mail spanning the major judgment paths.

**What you'll do:** Approve this Brief, paste into a fresh chat, forward the verification emails, review the first several drafted briefs closely at the gate — the gate is the calibration surface for this pass.

---

## Current state going in

By this point (verify at chat start, escalate on any miss): contracts define pm-workbook, seven intent types, and the seed table (BB-contracts-first, executed 2026-07-07); tuples are bi-temporal, predicates registered, lexicon tables live, node embeddings in pgvector (BB-storage-layer); the Archivist files additively, fires on seal, and has completed a live PM filing run (BB-archivist-rewrite); the three upstream passes emit frame.json + harvester.json + pm.json on real mail (BB-upstream-passes). The existing inline reasoning pass (E-180/E-194 code in `lib/reasoning.ts`) is still the one producing briefs — this Brief retires it. Salvage what transfers: it already reads SCHEMA.md type guidance at runtime (E-186) and carries the §9 co-employment producer (E-194); both behaviors survive into Pass 4 (co-employment likely relocates to consuming the Harvester's tier claims rather than recomputing them — receiving chat confirms against the code).

## Receiving chat

New chat to be opened.

## Scope

**In scope — the pass, implementing the Part 4 spec:**

1. **Inputs:** frame.json, harvester.json, pm.json, read-only cbrain (entities, aliases, node state, predicate registry, type guidance per E-186), capped attachment extracts. The overlays replace the old thin-candidates input. Fallback preserved (E-142): on a reasoning fault, author a degraded brief containing harvester.json's New claims as bare `create_entity` intents (seed: `inferred`/`none`/`none`) so the worklist is never lost.
2. **The seven jobs, in order:** bind anchors across overlays (PM's owner `m-47` ↔ Harvester's claim on `m-47`); settle ambiguous identity via context + cbrain read (true ties escalate, never guess); bind pronouns only when unambiguous, always basis-inferred; consume the steer (sole consumer, reports `steer_consumed`); judge new-vs-revision per PM candidate; derive new entities/relationships/predicate candidates; author the brief.
3. **New-vs-revision decision rule (the hard call):** cross-method agreement (top-k node + lexicon term linked to that node) → `append_pm_assertion` on that node; embedding-only → read the node, judge in prose, lowered seed; no match + task-shaped language → `record_pm_item`, low seed; thread prior weights toward revision but never forces. Each judgment's `effect` classification (reinforce/weaken/contradict/neutral) carries a one-line rationale — the classifier is a model call and reviewable like any basis.
4. **Confidence seed:** derived strictly from the published lookup table — basis × match_basis × corroboration → seed_value. The pass classifies; the table produces the number. No felt confidence anywhere. Group Register 1 intents by seed band and write the Register 2 callout for low-band high-consequence intents per the v0.5.0 legend (E-273).
5. **Escalation rule (both-worlds design):** an unresolvable tie becomes an open flag on the brief AND a proposed open Issue on the workbook (a `record_pm_item`, kind issue, titled as the question). Today the flag stops the seal; gateless-later, the Issue is the surface.
6. **Register 2 authorship:** comprehension prose, flags translated from unresolved overlay claims, convo_suggestion — all authored here, unchanged in shape from v0.4.0.
7. **Boundaries (hard):** no cbrain writes; proposes only; never blends the PM channels into one number; closed intent vocabulary only; reads cbrain (filed knowledge), never its own prior unfiled briefs.
8. **Cutover:** the old inline reasoning path is removed only after Pass 4 produces correct briefs on the verification set — consumer-before-producer ordering; the pipeline never goes dark. Pause the cron during the cutover window; re-process paused mail after. **At cutover, update WATCHER_CONTRACT §4/§8's vocabulary wording** (amendment 3 above): frame it by role — which pass authors which intents — not by count or deployment state.
9. **Verification set — four real emails, each exercising a path:** (a) known person + clear revision to an existing node (expects `append_pm_assertion`, cross-method seed); (b) new person + new task (expects bound create_entity + record_pm_item referencing it intent-to-intent); (c) a bare ambiguous first name (expects flag + proposed workbook Issue, no guess); (d) a steer-directed instruction (expects basis-steer intents, `steer_consumed: yes`). Each drafted brief reviewed at the gate; sealing + filing them is the full-pipeline proof.

**Out of scope:** removing the gate; the true steer command-bypass (E-241, deferred); the lexicon derivation job (unblocks after this — real workbook content now exists; log as next step); the sequencing viewer and cycle detection (E-251/E-252); object/equipment resolution (thin literal captures only, per the standing deferral).

## Directive

Read `lib/reasoning.ts` and the Part 4 spec in full first, and reconcile: list what transfers, what relocates, what retires — present that inventory in chat before writing code. Then build, deploy, run the four-email verification, cut over (cron paused), update the WATCHER_CONTRACT vocabulary wording, and remove the old path. MCP toolset is frozen at conversation start — if any tool built earlier in this chain is needed but not visible, that's a fresh-chat signal, not a workaround.

## Inputs

- `Chooch333/email-watcher` → `lib/reasoning.ts` and surrounding structure (full read)
- BRIEF_SCHEMA v0.5.0 as committed 2026-07-07 incl. the seed table and band legend; SCHEMA.md pm-workbook shape (contract field names win)
- Design doc Part 4 in full (the spec this implements), Part 2 C6/C7
- Live outputs from BB-upstream-passes on a recent real email (the actual input shapes)
- E-279 snapshot (BB-5 scope carryovers from BB-1)

## Pasteable prompt

> Read PROTOCOL.md from Chooch333/chat-protocol first, then fetch this Brief from Chooch333/chat-protocol → briefs/BB-2026-07-01-pass4-reasoner.md and load Project State for `email-interface-build`. This chat executes that Brief — the final item in the intake-pipeline sequence. Note the amendment block at the top, including the E-279 carryovers and the WATCHER_CONTRACT vocabulary update at cutover. Verify all four prior Briefs landed (contracts, storage, archivist, upstream passes) before touching code; escalate on any miss. First deliverable: the transfer/relocate/retire inventory of the existing lib/reasoning.ts against the Part 4 spec, for my approval. Then build → deploy → four-email verification set (I'll forward them) → cutover with cron paused → contract wording update. Session Log per session, referencing the Brief ID.
