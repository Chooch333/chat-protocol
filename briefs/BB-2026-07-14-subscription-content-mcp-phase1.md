> **Relocated 2026-07-14.** This brief's canonical git home moved to
> `Chooch333/subscription-content-mcp/docs/design/BB-2026-07-14-subscription-content-mcp-phase1.md`
> now that the target repo exists (this file was the `chat-protocol/briefs/`
> fallback, used only because no target repo existed at authoring time — see
> PROTOCOL.md's shelving canon). Left here for history; not maintained further.
> The Project State plan (`c58865b5`, project `paywall-mcp`) points at the new location.

---

# Build Brief — BB-2026-07-14-subscription-content-mcp-phase1
*Updated: 2026-07-14 · Turn 2 · BUILD-READY · shipped 2026-07-14 (Charles approved).*
*Git home (at ship): Chooch333/chat-protocol/briefs/BB-2026-07-14-subscription-content-mcp-phase1.md (target repo created by this build; brief moves to its docs/design/ as a build step).*

## Completeness gate

| # | Gate | Status |
|---|------|--------|
| 1 | Goal restated; Charles confirmed | ✅ (confirmed turn 2) |
| 2 | Scope walls populated | ✅ |
| 3 | Zero open forks | ✅ (all 3 answered turn 2) |
| 4 | Inputs verified or gated with named access | ✅ |
| 5 | Needed schemas/configs exist in draft | ✅ |
| 6 | Directive prescriptive, MCP-first | ✅ |
| 7 | Acceptance criteria concrete | ✅ |
| 8 | Assumptions genuinely unverifiable only | ✅ |
| 9 | Return contract | ✅ |
| 10 | Pasteable prompt | ✅ |
| 11 | All 11 domains Answered/Defaulted/N-A | ✅ (table below) |

---

**● What this is:** Build `subscription-content-mcp` — an MCP server that fetches content from sites Charles pays for, using his own logins. This brief ships the shared infrastructure plus the Peter Attia adapter only. Unblocks the Attia→cbrain intake (health-intake HI-001). The architecture is multi-site by design: adding Substack, Rehab Valuator, or Reventure later = one adapter file + one row in the sites registry, no core changes.

**● What I'll do (orchestrator):** Create repo `Chooch333/subscription-content-mcp`, stand up the Vercel-hosted MCP with sites registry and cookie auth, run the Gate 0 paywall-mechanism check, build the PA adapter and article queue, deploy, and prove it by returning one members-only article end-to-end.

**● What you'll do:** Supply the peterattiamd.com session cookie when the build halts at the hard gate. Review the completed build and fork log afterward.

---

## ● Current state going in

Project paywall-mcp planned deep in April 2026, zero code, no repo, no tables **[verified — Project State full dump 2026-07-14]**. 16 architecture decisions locked and durable (P-001–P-016): one MCP server, Vercel-hosted, fetch+cookies (no browser tooling v1), adapter pattern with sites registry, background queue for batch work, standardized error envelope. The April build-plan and deficiencies-tracker artifacts are unreachable from this Project's chat search; Charles waived retrieval (fork 3, 2026-07-14) — the Project State record covers this slice fully.

Charles directed 2026-07-14 that this build gates the Attia intake: BB-2026-07-14-attia-intake (health-intake plan 4a389c93) is blocked on this adapter **[verified — HI-001]**. Theme synthesis is explicitly out: Charles has scoped the content-reasoning pipeline elsewhere; it is gated on this MCP existing, not part of it.

Empirical signal **[verified 2026-07-14]**: unauthenticated fetch of a premium article (`/a-necessary-antidote-to-diet-tribalism/`) returns server-rendered WordPress HTML, no bot-block, no JS wall — teaser paragraph plus "members log in" notice. Site is fetchable with plain HTTP; cookie-authenticated behavior is Gate 0, prior leaning server-gate.

## ● Receiving chat

Orchestrator queue (autonomous execution per PROTOCOL.md). Plan on Project State project `paywall-mcp`, status `queued` at ship. **Queue order matters:** this plan must sit ahead of BB-2026-07-14-attia-intake, which re-queues only after HI-001 resolves.

## ● Scope

**In scope (Phase 1 + PA adapter):**
- New repo `Chooch333/subscription-content-mcp` [Charles, fork 1] — TypeScript, Next.js API routes, Vercel, matching the Custom GitHub / Project State MCP pattern.
- Core MCP: MCP auth (pattern copied from Custom GitHub MCP), `sites` + `site_cookies` tables, adapter dispatch by `site_key` (string, server-validated — P-014), standardized error envelope (P-013). **This core is the multi-site contract** — Substack/RV/Reventure adapters plug in later without touching it.
- Gate 0 (resolves blocker P-025 / deficiency #19): authenticated fetch of one members-only article; record CSS-hide vs server-gate.
- PA adapter: premium articles, show notes, Topic Guides, newsletters → markdown body (P-007).
- Article tools, PA-only initially: `fetch_article` (sync), `fetch_articles` + `ingest_pending` (queued — P-011 recovery rules), `get_queue_status`, `cancel_batch`.
- `articles` + `mcp_article_queue` tables in Supabase project `wejflvxwqpiyfavhcepf` (P-006).
- Politeness: 3s inter-fetch delay, concurrency 1, honest User-Agent (deficiency #18).
- Ops minimums: cookies service-role-only (#23), one saved PA article HTML fixture in `/fixtures` (#25), `mcp_run_log` table (#29).

**Out of scope (pre-answered):**
- Substack / Rehab Valuator / Reventure adapters, tools, schemas — later phases (deficiencies #13, #14, #20, #21, #22, #28; blockers P-023, P-024, P-026, P-027 remain open, untouched).
- Theme synthesis (P-008/P-009/P-010/P-015/P-016) — **out per Charles, fork 2, 2026-07-14**; the reasoning pipeline is scoped elsewhere and gated on this MCP. Those decisions stay valid for whichever build eventually consumes them; not superseded. Deviation logged: `fetch_article` returns article metadata + summary + body pointer, without theme links (departs from P-016's response shape for this phase).
- Automated cookie rotation — manual re-login on `auth_expired` (P-032).
- Email tables — untouched; deficiency #31 closed as "leave emails alone."
- Dev/prod split (#26) — single env for a personal tool.
- cbrain entity creation — downstream, BB-2026-07-14-attia-intake's job.

## ● Directive

1. **Custom GitHub MCP `create_or_update_file`** — create repo `Chooch333/subscription-content-mcp`, scaffold: README, `/adapters/peterattia.ts`, `/lib/{mcp-auth,dispatch,errors,fetcher}.ts`, `/fixtures/`, `vercel.json`. TypeScript + Next.js API routes. Copy this brief into `docs/design/` and update the Project State plan's git-home pointer (`update_plan_content`).
2. **Supabase MCP `apply_migration`** on `wejflvxwqpiyfavhcepf`, name `subscription_content_phase1`: create `sites`, `site_cookies`, `articles`, `mcp_article_queue`, `mcp_run_log` per DDL draft below. Seed `sites` with row `peterattia`.
3. **HALT — hard gate (credentials):** email Charles for the peterattiamd.com session cookie; insert into `site_cookies` on receipt.
4. **Gate 0:** authenticated fetch of one premium article; record whether full body renders (CSS-hide vs server-gate). If cookies-only fails entirely: halt and report — the browser-tooling fallback is a design change, not an orchestrator fork.
5. Build PA adapter against fixture + Gate 0 findings; implement the five article tools with the P-013 error envelope (codes incl. `auth_expired`, `auth_missing`, `unknown_site`, `parser_failure`).
6. **Vercel MCP** — connect repo, deploy, confirm the MCP endpoint responds.
7. Prove end-to-end: `fetch_article` on one members-only URL returns parsed markdown stored in `articles`.
8. **Project State MCP:** resolve P-025 with the Gate 0 answer, resolve HI-001 (then set the attia-intake plan back to `queued`), complete P-022 (naming resolved). Close with a Session Log on `paywall-mcp` referencing this Brief ID, fork log included.

## ● Inputs

- Project State `paywall-mcp` decisions P-001–P-016, blockers P-023–P-027, next moves P-017–P-022 **[verified 2026-07-14]**
- HI-001 gating context on `health-intake`; attia-intake plan 4a389c93 **[verified]**
- MCP auth pattern from Charles's existing custom MCP servers **[pattern verified; file specifics — builder greps the live repo at start]**
- peterattiamd.com: server-rendered WordPress, fetchable unauthenticated **[verified 2026-07-14]**
- Session cookie **[gated: hard gate at Directive step 3 — only Charles can supply]**
- April build-plan + deficiencies-tracker artifacts **[waived by Charles, fork 3 — not required for this slice]**

### DDL draft *(draft — builder refines)*
- `sites` (id, site_key unique, base_url, adapter_key, created_at)
- `site_cookies` (id, site_key fk, cookie_value, updated_at) — service-role read only
- `articles` (id, site_key, url unique, title, published_at, content_type, body_md, summary, ingested_at, content_hash)
- `mcp_article_queue` (id, batch_id, url, status pending/processing/done/failed, attempt_count, error_code, created_at, updated_at) — stuck-processing >5min → pending; attempt_count ≥3 → failed (P-011)
- `mcp_run_log` (id, tool, site_key, ok, error_code, duration_ms, created_at)

## Completeness table (11 domains)

| Domain | State | Where |
|---|---|---|
| 1 Purpose & users | Answered | What this is (users: Charles's agents/chats; downstream: attia-intake) |
| 2 Acceptance criteria | Answered | §Acceptance |
| 3 Runtime & execution | Answered | Vercel serverless + cron worker (P-002, P-011) |
| 4 Data — schema | Answered | DDL draft; P-007 markdown body |
| 5 Data — storage & recall | Answered | Supabase `wejflvxwqpiyfavhcepf` (P-006); recall via tools + direct SQL |
| 6 Interconnectivity | Answered | §Directive MCPs; consumed later by attia-intake |
| 7 Access & hard gates | Answered | One hard gate: session cookie (step 3). Repos: new repo + chat-protocol. No money beyond existing Vercel/Supabase. No destructive data change (additive migrations only) |
| 8 Skills & tools | Answered | orchestrate-build / execute-build-task; Custom GitHub, Supabase, Vercel, Project State MCPs |
| 9 Sequence & dependencies | Answered | Ahead of attia-intake on the shelf; no upstream builds |
| 10 Assumptions & risks | Answered | §Assumptions; Gate 0 is the risk burn-down |
| 11 Design intent narrative | Answered | Below |

**Design intent:** This MCP is the authenticated-session extractor of the intake infrastructure, packaged so any agent can call it. When trade-offs appear, prefer the smallest thing that returns a members-only PA article reliably and politely — this is a personal tool fetching content Charles pays for, not a scraper platform. Keep the core site-agnostic (registry + adapters) even where a PA-only shortcut is tempting; the second site is the point of the pattern. Never store or handle passwords — cookies only, service-role only. If the site pushes back (blocks, captchas), stop and report rather than escalate evasion.

## Acceptance criteria

1. Gate 0 answer recorded (CSS-hide vs server-gate) with fetched evidence quoted in the Session Log; P-025 resolved.
2. `fetch_article` on a members-only URL returns `{ok:true}` and the article row exists in `articles` (ID quoted).
3. `fetch_articles` on 3+ URLs enqueues a batch; cron drains at ≤1 fetch/3s; `get_queue_status` reports accurately.
4. Bad `site_key` → `unknown_site`; missing cookie → `auth_missing` (one live call each).
5. Adding a hypothetical second site requires zero core-file edits — demonstrated by a dry row insert into `sites` with a stub adapter key (extensibility proof).
6. Zero occurrences of "paywall" in repo name, MCP server name, or tool names.
7. HI-001 resolved; attia-intake plan status back to `queued`.

## Assumptions
- PA cookies last weeks; manual rotation acceptable (P-032). Confirmed by living with it.
- Vercel pro tier suffices for the cron worker (P-035). Confirmed at first batch run.

## ● Pasteable prompt

> This is a build chat. Read PROTOCOL.md first (Custom GitHub MCP get_file_contents, owner=Chooch333, repo=chat-protocol, path=PROTOCOL.md). Then fetch and execute Build Brief BB-2026-07-14-subscription-content-mcp-phase1 (owner=Chooch333, repo=chat-protocol, path=briefs/BB-2026-07-14-subscription-content-mcp-phase1.md). Task in one line: build the subscription-content-mcp server (repo Chooch333/subscription-content-mcp, TypeScript/Next.js/Vercel) with sites-registry core and the Peter Attia adapter, per the brief's Directive exactly — Gate 0 paywall check first after the cookie hard gate. Rules: MCP-first; additive migrations only; re-read every file after writing; log every judgment-call fork; halt only at the named hard gate; close with a Session Log on project paywall-mcp referencing the Brief ID with all seven acceptance criteria evidenced.

## Fork log (design-time)
1. Repo name → `subscription-content-mcp` **[Charles, 2026-07-14]**
2. Theme synthesis → out of this brief; reasoning pipeline scoped elsewhere, gated on this MCP **[Charles, 2026-07-14]**
3. April artifacts → proceed without; Project State record sufficient for this slice **[Charles, 2026-07-14]**
4. Stack/parser/auth/env/politeness/ops defaults (deficiencies #1–4, #18, #23–27, #29, #31) → per Decisions in Scope **[Claude-per-doctrine — judgment-call, basis: existing MCP conventions + April decision record]**

## Fork log (build-time — see the live copy at Chooch333/subscription-content-mcp/docs/design/ for entries 5+)
This copy is frozen as of relocation. Entries 5-8 (repo-creation tooling gap, queue schema refinement, RLS judgment call, Next.js-vs-plain-Vercel-functions judgment call) are recorded only in the live copy.
