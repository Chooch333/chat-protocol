# Build Brief — BB-2026-11-17-personal-gmail-mcp
**Build:** 86 · **Git home:** Chooch333/chat-protocol · `briefs/BB-2026-11-17-personal-gmail-mcp.md` (the build copies it to the new repo's `docs/design/`)
**Project State plan:** `cf69d9b8-265f-4efb-94ff-5e90dcc19313` on `total-harness`
**Skills:** orchestrate-build, checkpoint-deploy not needed (not cbrain-ui) · **Board tag:** DA-1117-second-gmail · **Desk report:** TH-234 · **Decision:** TH-235

**● What this is:** A new private server on Vercel that lets Claude read Charles's personal Gmail, cecourtney10@gmail.com, and write drafts in it, added to claude.ai as its own connector named so no chat can confuse it with the existing Gmail connector.

**Stack Flow:** adds dot `personal-gmail-mcp` (Vercel, role door, component custom-mcps) and rim dot "Personal Gmail (cecourtney10)" kept apart from gmail-in/gmail-out; two two-way lines Chats↔server↔Personal Gmail; heartbeat check every 15 min on /api/health; pays_with [vercel, supabase].
**Step map:** no change — a connector, not a workflow.

**● What I'll do:** build repo Chooch333/personal-gmail-mcp, deploy it, wire its keys, add the stack-map dots, prove every tool from a fresh chat, record which places can reach it, and leave Charles one setup card.

**● What you'll do:** nothing until the build ends; then one card, about 10 minutes: create the Google sign-in client and put its two values into Vercel, sign in as cecourtney10 and approve, add the connector in claude.ai with the settings the sign-in page shows.

## ● Current state going in

**Asked for — in Charles's words** *(frozen)*
> "I want to create a custom MCP server that will allow claude to access a second gmail email account" — Charles, 2026-11-17

Changes appended:
- 2026-11-17 Charles: "I don't want you to move or mess with the current gmail account set up. I just want a new MCP that will allow the cecourtney10@gmail.com email to be accessed by CLaude. it also needs to be clear to a chat which MCP does which."

- [verified] The existing claude.ai Gmail connector (and Drive, Calendar) is signed in as claude.go.between@gmail.com, the relay inbox. Untouched by this build.
- [verified] The built-in Gmail connector holds one account; a second replaces the first — so a separate server is the only way to have both.
- [verified] email-watcher's Google Cloud project has its consent screen published to Production (E-023), so a refresh token from a client in that project doesn't expire weekly.
- [verified] House servers (project-state-mcp, subscription-content-mcp) are plain TypeScript Vercel functions, JSON-RPC at /api/mcp, and their auth check allows everything when the secret is missing. Do not copy that check.
- [verified] Existing scheduled tasks keep the connectors they were made with (EB-050); new ones get all connectors.
- [assumed] The phone app lists custom connectors added on the web.

## ● Receiving chat
Night build (overnight). Ends at the hard gate with one Next steps card for Charles; a fresh chat finishes the live proof after he adds the connector (that card says so).

## ● Scope
**In:** search, read threads/messages, read attachments as text (PDF included), list labels, drafts (new and replies; update/delete only its own), whoami, health check, own sign-in link locked to cecourtney10, server instructions naming both Gmail connectors, stack-map dots, lane proof, one lesson replacing PP-010's confusion.
**Out:** any change to the existing Gmail/Drive/Calendar connectors; send, forward, trash, archive, relabelling mail; saving personal mail into cbrain (the relay stays the only door); AI calls inside the server; .mcp.json copies; personal Drive/Calendar.

## ● Directive
1. **Repo** — Custom GitHub `create_repo` Chooch333/personal-gmail-mcp (private). Copy this brief to `docs/design/`. Copy subscription-content-mcp's `api/mcp.ts` + `lib/mcp-auth.ts` split with three fixes: (a) fail closed when no secret is stored, compare with `crypto.timingSafeEqual`; (b) `notifications/*` → HTTP 202, no body; (c) `initialize` echoes the client's protocolVersion when supported.
2. **Packages, pinned exact** — `@googleapis/gmail` 22.1.0 (confirm its `auth` export with Context7 first), `@types/node` 24.19.2, `typescript` 7.0.2 (if Vercel's build fails on 7.x, pin 5.9.3 and comment why), `engines.node` "24.x". Scripts `typecheck` (`tsc --noEmit`) and `test` (`tsc -p tsconfig.test.json && node --test dist/test/*.test.js`) from the first commit. PDF text: one small pinned library (`unpdf`), checked on the registry before use.
3. **Files** — `lib/tools.ts`, `lib/handlers.ts` (one function per tool), `lib/gmail.ts` (client from the stored token), `lib/mime.ts` (walk `payload.parts`, base64url, honour charset, prefer text/plain, HTML→text, never an empty body), `lib/draft.ts` (RFC 2822 by hand, RFC 2047 subjects, strip CR/LF from header values, In-Reply-To/References + threadId for replies), `lib/store.ts`, `test/*.test.ts` with redacted fixtures (nested multipart, non-UTF-8, inline vs attached, header injection, reply headers, draft round-trip, JSON-RPC 401/202/tools-list/unknown method).
4. **Telling the two apart (Charles's rule)** —
   - `initialize` returns `serverInfo.name` "personal-gmail" and `instructions`: "Personal Gmail: Charles's own inbox, cecourtney10@gmail.com. Use these tools when Charles means his email or his inbox. The separate 'Gmail' connector is a different inbox, claude.go.between@gmail.com, the relay that feeds cbrain; use that one only for relay and intake checks."
   - Every tool name starts `personal_`; every tool description opens "Charles's personal inbox (cecourtney10@gmail.com):"; every result carries `"mailbox": "cecourtney10@gmail.com"`.
   - Connector name on Charles's card: "Personal Gmail (cecourtney10)".
   - The sign-in callback refuses any Google account other than cecourtney10@gmail.com (checked with `users.getProfile`), so the server can never point at the wrong inbox.
   - Add a Project State lesson on total-harness that updates PP-010: two Gmail connectors now exist; "Gmail" = relay, "Personal Gmail (cecourtney10)" = his inbox.
5. **Tools** (names fixed once the connector is added) — `personal_whoami`; `personal_search_threads` (Gmail query, metadata only: id, from, subject, date, snippet; 25/page; pageToken); `personal_get_thread` (~4k chars/message, ~20k/call, quoted history removed, `truncated`/`chars_total`/`next_offset`); `personal_get_attachment` (text and PDF text, same caps and offset; other types return name and type only; never base64; refuse over ~3 MB with a plain message); `personal_list_labels`; `personal_create_draft` (new, or reply on a thread); `personal_update_draft` and `personal_delete_draft` (only drafts in the write log). Tool descriptions say: "To put an email into cbrain, Charles forwards it to the relay +project address."
6. **Store** — Supabase MCP `apply_migration` on project lpeswznkxzeeyiqaewma (cbrain): `personal_gmail_auth` (single row: refresh token encrypted AES-256-GCM with `TOKEN_KEY`, connector-secret SHA-256 hash, address, granted scopes, updated_at) and `personal_gmail_writes` (append-only: tool, draft_id, at; no body). RLS on, no policies, service-role only. Run `get_advisors` after.
7. **Sign-in** — `/api/auth/start` redirects to Google consent (scopes `gmail.readonly` + `gmail.compose` only, `access_type=offline`, `prompt=consent`, `login_hint=cecourtney10@gmail.com`, signed `state`). `/api/auth/callback` checks the address, stores the token, and on first sign-in mints the connector secret and shows the connector settings once (name, URL, header or `?token=` form); only the hash is kept. Signing in again replaces the token and keeps the secret.
8. **Vercel** — project in team chooch333s-projects, git-connected (Vercel `create_git_project`; if that fails, wire-new-app rehearsal with supabase_ref lpeswznkxzeeyiqaewma). Supabase keys and a generated `TOKEN_KEY` via the wire-new-app courier (A-108), never through chat. `GMAIL_CLIENT_ID` / `GMAIL_CLIENT_SECRET` come from Charles's card. All production-only, Sensitive. `maxDuration` 30.
9. **Health** — `GET /api/health`, no login: fetch an access token, call `users.getProfile`, return `{ok, reason, checked_at}` only. A tool hitting a dead token answers in plain words: "Your personal Gmail sign-in has expired — sign in again at <link>. The stack map already shows this."
10. **Stack map** — edit cbrain `concepts/stack-map.md` per the Stack Flow line; check `type: heartbeat, every: 15m, fail_if: {field: ok, ne: true}`; breaks_if names the sign-in link and notes the Google project is shared with email-watcher. Dry-run the checker against a forced ok:false (red), then real (green). It will be red until Charles signs in; that is expected and the card says so.
11. **Hard gate → card** — one `checkin` (waiting on you) with the three steps and links: Google Cloud console OAuth client (Web app, in email-watcher's project, redirect URI `<prod>/api/auth/callback`) then the Vercel env page for the two values; the sign-in link; claude.ai → Settings → Connectors → Add custom connector using the shown settings (Request headers if the dialog offers them, else the `?token=` URL). Last line: "Then open a new chat and say: Build 86 — finish the live checks."
12. **Live proof (the finishing chat)** — from a fresh chat: `personal_whoami`, a search, a long thread that pages, a PDF attachment, a test draft made and deleted, a refused delete of a draft it didn't make; ask "check my email" and confirm the chat picks the personal tools. Then create one throwaway scheduled task that calls `personal_whoami` and delete it after; record web, new-task and night-build reachability in stack-map `lanes:`; phone stays "unproven until used".
13. **Close** — Session Log, done-receipt against the criteria below, plan → succeeded.

## ● Inputs
- [verified] subscription-content-mcp `api/mcp.ts`, `lib/mcp-auth.ts`; email-watcher `lib/gmail.ts`, README; decisions E-023, E-028, A-108, PP-032, EB-050, PP-010, TH-235.
- [verified] Desk report TH-234; lens-notes EB-251, EB-252, EB-253.
- [gated: Charles] Google Cloud console, Google consent as cecourtney10, claude.ai connector settings — all on the step-11 card.

## Acceptance criteria
1. Typecheck and tests pass; deploy READY on Node 24.
2. /api/mcp refuses a missing or wrong secret, and refuses everything before first sign-in.
3. Signing in as any account other than cecourtney10 is refused.
4. `initialize` returns the instructions text naming both inboxes; every tool name starts `personal_`; every result carries the mailbox.
5. From a fresh chat: search, paged long thread, and PDF attachment all return data; "check my email" uses the personal tools.
6. A test draft is made and deleted; deleting a draft the server didn't make is refused.
7. No send, trash, forward or relabel tool exists; the token's scopes are exactly gmail.readonly + gmail.compose.
8. /api/health is ok:true after sign-in; a forced ok:false turned the dot red, then green.
9. Stack map shows both dots and lines; `lanes:` records the reachability results.
10. The existing Gmail, Drive and Calendar connectors are unchanged (still claude.go.between).

## Domain table
| Domain | State | Where |
|---|---|---|
| 1 Purpose & users | Answered — personal daily-use tool; Charles via any chat | What this is |
| 2 Acceptance | Answered | Acceptance criteria |
| 3 Runtime | Answered — Vercel function, on demand, TypeScript, pinned libraries, $0 | Directive 1–2, 8 |
| 4 Schema | Answered — two new tables | Directive 6 |
| 5 Storage & recall | Answered — Supabase, exact lookup; write log kept, no mail bodies stored | Directive 6 |
| 6 Interconnectivity | Answered — Gmail API in, claude.ai connector out; dead token → health dot → card | Directive 9–11 |
| 7 Access & hard gates | Answered — only gate: Charles's Google client, consent, connector add | Directive 11 |
| 8 Skills & tools | Answered — Custom GitHub, Vercel, Supabase, Project State, Context7 | Directive |
| 9 Sequence | Answered — repo → store → deploy → map → card → live proof; no upstream builds | Directive order |
| 10 Assumptions & risks | Answered — phone unproven; existing tasks won't see it; password change kills the token (health catches it) | Current state |
| 11 Design intent | Answered | below |

**Design intent.** Charles wants Claude to reach his own inbox without anything about the current setup changing, and he never wants a chat to mix up the two inboxes. Protect, in order: (1) the existing connectors and the relay stay exactly as they are; (2) the server can only ever be his personal inbox, and says so everywhere a chat looks; (3) nothing can send or delete mail; (4) no secret is ever pasted into a chat or a repo; (5) Charles's part is one short card. Prefer the house pattern and fewer parts over cleverness. If a choice comes up the brief doesn't cover, pick the option that changes less of the existing setup.

## Return contract
Build chat claims the plan (running), follows skills/build and orchestrate-build, logs judgment calls, writes a Session Log and a done-receipt naming each criterion by number; criteria needing Charles's sign-in are proven by the finishing chat before succeeded. Brief ID BB-2026-11-17-personal-gmail-mcp.

## ● Pasteable prompt
Build 86 — execute Build Brief BB-2026-11-17-personal-gmail-mcp. Pull the plan from Project State (plan_id cf69d9b8-265f-4efb-94ff-5e90dcc19313, project total-harness, status queued — set it to running when you claim it). Git copy at Chooch333/chat-protocol/briefs/BB-2026-11-17-personal-gmail-mcp.md. Follow the brief's fork-handling rules: answer forks autonomously with judgment-call tags; escalate only at hard gates (Charles's Google sign-in client, his consent as cecourtney10, and adding the connector in claude.ai — all on one card). Never change the existing Gmail, Drive or Calendar connectors.

---
## Decisions made
- **Charles:** new server for cecourtney10 only; existing setup untouched; chats must tell them apart (TH-235).
- **Build, not adopt** [Claude-per-doctrine] — paid services hold mail keys and cost money; open-source ones need a new host.
- **Drafts only** [Claude-per-doctrine] — new scheduled tasks inherit the connector; code is the send guard.
- **Own sign-in link** [Claude-per-doctrine] — no key pasted by hand (desk 2–1).
- **Separate Google client, same project** [Claude-per-doctrine] — resetting one never breaks email-watcher.
- **Store in cbrain's Supabase** [Claude-per-doctrine] — the callback must write somewhere; existing key courier reaches it.

## Out of scope
- Governance flag, parked for the security session: a connector secret now opens a personal inbox that may hold work mail.
