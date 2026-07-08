# Build Brief — BB-2026-07-01-upstream-passes

> **AMENDMENTS 2026-07-08** (from the review, approved by Charles):
> 1. **Resequenced from item 3 to item 4** — now runs AFTER BB-archivist-rewrite. The PM increment's stub workbook is created by the Archivist Brief's verification run (gate-path `added_intents`), so the "hand-author the stub" circular dependency is gone.
> 2. **Node embeddings are no longer a maybe** — they land in BB-storage-layer scope item 8. If they're missing at increment 3, that's an escalation, not something to build here.
> 3. Increment 2 addition: the **Harvester may use the steer as a matching context clue** (WATCHER_CONTRACT v0.9 §8.5) — wire it, don't interpret it.
> 4. Increment 3 addition: the lexicon check reads **`status: active` terms only** (retire-don't-delete means retired terms are live rows).
> 5. **Cron/transition safety:** pause the watcher cron (or feature-flag the new path) during each increment's deploy-and-verify window; re-process any mail that arrived during the pause. Deploy-green on a half-cut-over pipeline with live mail incoming is not acceptable.

**What this is:** Build the three mechanical/matching passes into the deployed watcher: mention anchors in the Organizer, the Harvester as its own overlay-writing pass, and the two-channel PM pass. Sequence item 4 of 5. Depends on BB-contracts-first (executed 2026-07-07), BB-storage-layer (pgvector targets + lexicon tables + node embeddings), and BB-archivist-rewrite (the filed stub workbook).

**What I'll do (receiving chat):** Ship three increments to `Chooch333/email-watcher`, each independently deployable and verified on real forwarded mail before the next starts.

**What you'll do:** Approve this Brief, paste into a fresh chat, forward one real test email per increment for live verification, review each increment's design before code.

---

## Current state going in

Verified from Project State (2026-07-01): the Organizer is deployed (WATCHER_CONTRACT v0.8; v0.9 as of BB-contracts-first) but emits no per-mention anchor IDs — the join hook the whole architecture rests on doesn't exist yet. The Harvester's logic largely exists inside today's monolithic watcher (§8–§9 harvest, dedup, domain→org) but not as a separate pass writing its own overlay. The PM pass exists nowhere. The frame/overlay folder model (E-228: immutable frame.json + harvester.json + pm.json per email) is decided but the current watcher doesn't produce it.

## Receiving chat

New chat to be opened. This Brief is large; the receiving chat may propose splitting increments 2 and 3 into their own sub-chats — that's an acceptable fork to surface, not decide.

## Scope — three increments, in order

**Increment 1 — Organizer anchors + frame.json.**
- Restructure the Organizer's output into the E-228 folder model: one folder per email; `frame.json` as the immutable structured record (segments with `s-` ids, sentences, participants, steer, carried-forward thread priors); the markdown email-record stays as the human/searchable view generated from it.
- Add mention anchors: every person/org occurrence in the cleaned text gets a stable `m-` id. People and orgs only (E-229) — no object/equipment anchors. Anchors mark occurrences, never identities.
- Sentence segmentation with pre-clean (deterministic splitter; signatures/headers/quotes already stripped by existing framing — verify how much is covered and close the gap).
- Verify: forward a real multi-reply email; inspect frame.json for correct message decomposition, anchor ids on every named person/org, stable sentence ids.

**Increment 2 — the Harvester as its own pass.**
- Extract the existing §8–§9 logic (entity harvest, dedup against live cbrain, alias resolution, domain→org co-employment tiers) out of the inline flow into a pass that reads frame.json and writes `harvester.json`.
- Output per anchor: one claim — Resolved (slug + match band: exact-id | mechanical | semantic), New (candidate create with thin fields), or Ambiguous (flagged, never guessed). Semantic-band matches are always Ambiguous-flagged, never auto-resolved.
- The steer is available as a **matching context clue only** (WATCHER_CONTRACT v0.9 §8.5) — the same way carried-forward issue ids are used. No directive interpretation here.
- Candidate relationships and enrich/flag_clue candidates carry over from the existing logic unchanged in substance — they just land in the overlay instead of directly in a brief.
- No pronoun resolution, ever, including one-candidate cases (E-231).
- Verify: run against the increment-1 test email; confirm every anchor has exactly one claim, bands are correct on a known person (Kris/Tyler/Benny), and an intentionally bare first name comes back Ambiguous.

**Increment 3 — the PM pass, two channels.**
- Reads frame.json (never harvester.json — parallel, E-227). Per sentence:
  - Channel 1: embed, pgvector top-k against the project's workbook node vectors (built by BB-storage-layer scope 8 — if vectors are missing, escalate, don't build), emit candidate node edges with similarity scores. Top-k only — no Cartesian product.
  - Channel 2: check against `lexicon_terms` for the project, **`status: active` only**, emit the hit list. Lexicon never filters the candidate pool (channel independence — corroboration only).
- Output `pm.json`: per sentence, graph result + vocab list; task-owner mentions carried as `m-` anchor ids only. Zero interpretation, zero task/issue judgment.
- Single round — no 3× multi-pass (both channels deterministic; document the caveat that 3× returns only if a non-deterministic step re-enters PM).
- **Stub workbook:** the nodes filed by BB-archivist-rewrite's verification run (real project, gate-path provenance) are the verification target. If more terms/nodes are needed, add them via another gate-path brief — never a direct commit.
- Verify: forward a real email mentioning a known node's subject; confirm the expected node appears in top-k and the lexicon hit corroborates.

**Out of scope:** Pass 4 / brief assembly (the current inline reasoning pass keeps producing briefs as-is until BB-pass4-reasoner replaces it — the pipeline must never go dark mid-build); the Archivist; the lexicon derivation job; steer interpretation in any of these passes.

## Directive

One increment at a time: design in chat → approve → build → deploy → verify on real mail → then the next. Pause the cron or feature-flag the new path during each deploy-and-verify window; re-process paused mail after. Consumer-before-producer commit ordering where applicable. Contract drift found mid-build goes back to the contract first (self-amendment rule), then code. Each increment's verification uses a real forwarded email, not synthetic input — deploy-green is not done.

## Inputs

- `Chooch333/email-watcher` (Vercel `prj_b2qdE8CWYcRVYNP0nrkOYHNfbBAP`), full read of the current parser/reasoning/lib structure before designing
- WATCHER_CONTRACT v0.9 + SCHEMA.md + BRIEF_SCHEMA v0.5.0 as committed 2026-07-07
- Lexicon tables + node embeddings from BB-storage-layer; stub workbook nodes from BB-archivist-rewrite
- Design doc Part 1 steps 1–2, Part 3 G5/G6; PM Redesign synopsis (channel independence, segmentation, single-round caveat)

## Pasteable prompt

> Read PROTOCOL.md from Chooch333/chat-protocol first, then fetch this Brief from Chooch333/chat-protocol → briefs/BB-2026-07-01-upstream-passes.md and load Project State for `email-interface-build`. This chat executes that Brief, increment 1 only to start — note the amendment block at the top (resequenced to item 4; embeddings and stub workbook are prerequisites, not work). Confirm BB-contracts-first, BB-storage-layer, and BB-archivist-rewrite are landed before touching code (check the contracts, the tuples table, and the stub workbook nodes; escalate on any miss). Read the full email-watcher code structure before proposing the increment-1 design. Each increment: design → my approval → build → pause cron → deploy → verify on a real email I forward → resume. Session Log per session, referencing the Brief ID.
