# Build Brief — BB-2026-10-14-protocol-retirement

**Build:** 39
**Brief:** BB-2026-10-14-protocol-retirement
**Git home:** owner=Chooch333, repo=chat-protocol, path=docs/design/BB-2026-10-14-protocol-retirement.md
**Project State:** chat-protocol · tags build-brief, build-39, agent-library, chat-protocol, front-door
**Overnight:** yes · **target_repo:** Chooch333/agent-library · **after:** Build 38 (BB-2026-10-14-skills-folder-merge)
**skills:** orchestrate-build
**Stack Flow:** no change (no running part added, removed or re-routed; the front door is a document)
**Authored:** 2026-10-14, harness review chat (Charles + Claude). Approved by Charles 2026-10-14: project home chat-protocol.

**What this is:** Move PROTOCOL.md's content into a new build skill and three reference files per the section map in this brief, point AGENT.md's Build row at the new skill, give every Project-State-writing skill a `requires:` header, and leave PROTOCOL.md as a stub that names where each part went.

**What I'll do:** Create `references/project-state.md`, `skills/build/SKILL.md`, `references/gotchas.md`; append two items to `references/chat-capture.md`; add `requires:` lines; edit AGENT.md (Build row, front-door paragraph, changelog out to `CHANGELOG.md`); update every live reference to PROTOCOL.md in agent-library; replace PROTOCOL.md with a stub; prove the new path by walking it; close with a Session Log and disclosures.

**What you'll do:** After the build: change each Project's instruction line that names PROTOCOL.md to name AGENT.md (text in §After the build). Until you do, the stub keeps the old pointer working.

---

## Current state going in

- [verified] PROTOCOL.md is 42,683 bytes (sha 659f171), ~2 KB of which is routing that AGENT.md already does. AGENT.md's Build row reads "PROTOCOL.md itself — no additional SKILL.md".
- [verified] Charles keeps the "read the protocol" pointer only in Project instructions, by design. General chats are not affected by this build.
- [verified] The last two PROTOCOL.md edits by builds were BB-2026-09-09-one-front-door (decision C-065) and BB-2026-09-04 (SQL fallback rule). This brief supersedes C-065: the door moves from PROTOCOL.md to AGENT.md.
- [assumed] After Build 38, the completeness framework and build-brief template live under `skills/design-assist/references/`. The build reconciles paths at start.

## Why (design decision, 2026-10-14)

About 2 KB of PROTOCOL.md's 42.7 KB applies to every chat that reads it; the rest is build-chat SOP and shared Project State vocabulary. Anthropic's current guidance (Claude Code best practices; Agent Skills progressive disclosure) and OpenAI's harness-engineering write-up converge: the always-loaded entry file is short and made of pointers; detail loads by trigger; rules that must hold every time move into mechanisms. AGENT.md already is the routing table and already forbids domain knowledge in itself. Making it the single front door removes a circular reference (its Build row points back at PROTOCOL.md) and drops ~50 KB of unused instruction from a quick-question chat inside a Project.

---

## Receiving chat

Night runner, 15 Oct 2026, after Build 38 lands. Project State project `chat-protocol`. Target repo `Chooch333/agent-library`; also writes `Chooch333/chat-protocol`.

---

## Scope

**In scope:**
- New: `references/project-state.md`, `skills/build/SKILL.md`, `references/gotchas.md`, `CHANGELOG.md` (AGENT.md's changelog moved).
- Edited: `AGENT.md`; `references/chat-capture.md`; `requires:` headers on Project-State-writing skills; live references to PROTOCOL.md in agent-library skills and references.
- `chat-protocol/PROTOCOL.md` → stub; `chat-protocol/README.md` one line; `chat-protocol/briefs/README.md` two lines.

**Out of scope:**
- `BRAINSTORM.md`, `chat-protocol/briefs/` contents, `chat-protocol/docs/design/`.
- The two MCP code fixes (write_status_snapshot error text; in-tool retry on "No approval received"). Their doc lines move to the gotchas reference for now; the fixes are a later brief on context-database.
- Project instructions (Charles), the build-brief-artifact plugin skill in claude.ai (Charles, optional), memory files.
- Rewriting prose. Text moves verbatim except the cuts the map names.

---

## Section map (the build follows this table)

| PROTOCOL.md section | Goes to | Treatment |
|---|---|---|
| Opening paragraph | delete | Replaced by `requires:` headers |
| Seven buckets + urgent tag; Sessions | project-state reference | Verbatim |
| Campaigns · naming rule · "What Charles sees" | project-state reference | Keep rule and legend; drop the two "why" paragraphs |
| Session Log · Fix Ticket · Build Brief templates | project-state reference | Verbatim |
| Build Brief protocol defaults | build skill | Keep rules; drop dated parentheticals |
| Shelving a brief | build skill | Keep four steps; drop the briefs/-is-historical note (two-line README in chat-protocol/briefs/ instead) |
| Reviewing a succeeded plan | build skill | Verbatim |
| Authority · designing/building line · hard gates · automatic intake · staleness | build skill | Keep the firm version once; remove the two restatements |
| When to use a Build Brief vs inline | build skill | Verbatim |
| Chat types | delete | AGENT.md is the table |
| Workflow · ticket closure | build skill | As numbered list |
| How to engage during builds | build skill (Response format, 3 lines) · chat-capture reference (rule 7) | Text in §Response format below |
| Build execution, tool-first | build skill | Four lines |
| Standing rules: no logging without ticket · one Session Log · snapshot mandatory · plain English · chat questions stay in chat · coordinate before creating projects | project-state reference | Short list |
| Judgment calls get provenance · builds keep Stack Flow current | build skill | Verbatim |
| Dashboard structure · "read this before first DB action" | delete | Owned elsewhere / replaced by requires: |
| write_status_snapshot slip · "No approval received" SQL fallback · MCP manifest freeze · GitHub Actions dark modes | gotchas reference | Verbatim; first two marked "until fixed in the MCP" |
| Comms Table (Q-channel · disclosures) | project-state reference | Keep every rule and lifecycle; drop "this is firm"/"same discipline" sentences |
| Punch List · Stack Repairer | delete | Keep one line in project-state reference: "Design Assist chats pull needs-brief punch items at intake" |
| cbrain predicate discovery | chat-capture reference | Verbatim |
| Appendix bucket → tool · plans/disclosures tools · soft enforcement | project-state reference | Verbatim |

## Response format (the three lines for skills/build/SKILL.md)

- A response that needs anything from Charles ends with one numbered list under the exact heading **Needs from you**, preceded by a `---` rule. If nothing is needed, no section.
- Everything that needs him goes on that one list: decisions, approvals, and judgment calls he might want to reverse. Nothing buried in the body.
- Each item is a closed choice with a recommendation.

Line for `references/chat-capture.md`: "During build or PM work, when a person, organization, project, or concrete relationship surfaces, propose it as a Needs-from-you item for cbrain capture; never auto-write; never infer a relationship that was not stated."

---

## Directive

1. Read live: `chat-protocol/PROTOCOL.md`, `agent-library/AGENT.md`, `references/chat-capture.md`, and the SKILL.md of every skill in `skills/` whose description mentions Project State, plans, briefs, builds, the Punch List, or the night run. Record shas.
2. `create_or_update_file` `references/project-state.md` per the map. Open with: "Shared reference for any skill that reads or writes Project State. Loaded by a skill's `requires:` header; not a trigger."
3. `create_or_update_file` `skills/build/SKILL.md`. Frontmatter: `name: build`; `description: Use when the first message says "this is a build chat": work that produces deliverables, decisions or progress on a project, logged to Project State. Carries authority rules, Build Brief defaults, workflow and response format.`; `requires: [references/project-state.md, references/gotchas.md]`. Body per the map plus §Response format.
4. `create_or_update_file` `references/gotchas.md` per the map.
5. Append the two chat-capture items to `references/chat-capture.md` (`replace_in_file` at end, or `create_or_update_file` with sha).
6. For each skill identified in step 1 that calls a Project State tool: add `requires: [references/project-state.md]` to its frontmatter (merge if a `requires` key exists). Read back each.
7. AGENT.md: first paragraph becomes "This file is the front door. Project instructions point here. It routes a first message to one skill; it holds no domain knowledge." Build row target → `skills/build/SKILL.md`. Add under Activation pattern: "A skill that reads or writes Project State declares `requires: references/project-state.md`; fetch what the header names after the skill." Cut the Changelog section into `CHANGELOG.md` and leave a one-line pointer. Append today's changelog entry there.
8. `search_code` agent-library for `PROTOCOL.md`. In every live skill or reference (not `docs/design/`, not `imports/`), replace the pointer with the specific new file (build skill or project-state reference, whichever the sentence means). Read back.
9. Replace `chat-protocol/PROTOCOL.md` with a stub under 1 KB: retired 2026-10-14 (Build 39); the front door is `agent-library/AGENT.md`; each former section and its new path, as a list. Update `chat-protocol/README.md` with one line. Add a two-line README to `chat-protocol/briefs/`.
10. Verify by walking the path: fetch AGENT.md; find the Build row; fetch its target; fetch each file its `requires:` names. Record the byte total. Then `search_code` for `PROTOCOL.md` and confirm hits only in the stub, `docs/design/`, `imports/`, `CHANGELOG.md`.
11. Close: Session Log on `chat-protocol`; `supersede_decision` C-065 with the new front-door decision; disclosures; done receipt covering acceptance 1–7.

---

## Acceptance

1. Every row in the section map is accounted for: each "goes to" file contains that section's text (spot-check five rows by searching a distinctive phrase), and each "delete" row's text appears in no new file.
2. `skills/build/SKILL.md` exists with the frontmatter above and the three Response format lines verbatim.
3. `references/project-state.md` contains the three templates verbatim (search for "## Entries to log", "## Resolution (fill in after)", "## Pasteable prompt").
4. AGENT.md: Build row target is an existing file; the file contains no sentence instructing a chat to read PROTOCOL.md; Changelog section is one pointer line.
5. Every skill that calls a Project State tool carries the `requires:` line (list them in the receipt).
6. PROTOCOL.md stub is under 1 KB and names every destination file.
7. Walk-the-path byte total for a build chat is under 45 KB and is recorded in the Session Log.

Verification is code and data only. Effort: M. Real-money cost: $0.

---

## Inputs

- [verified] PROTOCOL.md sha 659f171; AGENT.md sha ea490e9; the section map above; the Response format lines above.
- [verified] Decision C-065 (one front door on PROTOCOL.md, 2026-09-10) is the decision this supersedes.
- [assumed] Post-Build-38 paths for design-assist's references.

---

## Design intent narrative

The goal is progressive disclosure: a chat loads only what its trigger needs, and loads it by following headers forward. Move text, don't rewrite it; the only edits are the cuts the map names, and when a cut is ambiguous, keep the text. When a sentence could belong to either the build skill or the project-state reference, put it in the reference, so no skill loses access to it. Keep the authority paragraph's firm wording and the three hard gates exactly. Prefer stubs to deletions so stale pointers degrade gracefully. If a skill's frontmatter has no YAML block, add a minimal one rather than skipping the `requires:` line. If chat-capture.md is not a fit for the two moved items, create `references/cbrain-capture.md` and disclose. The build is done when a cold chat can start at AGENT.md and reach everything a build chat needs without ever being told PROTOCOL.md existed.

---

## Eleven domains

| Domain | State | Where |
|---|---|---|
| 1 Purpose & users | Answered | What this is; every project chat |
| 2 Acceptance | Answered | §Acceptance, 7 checks, code and data only |
| 3 Runtime | Defaulted | Night runner; Custom GitHub MCP; Project State MCP |
| 4 Data schema | N/A | No database change |
| 5 Storage & recall | Defaulted | GitHub markdown; exact path |
| 6 Interconnectivity | Answered | agent-library ↔ chat-protocol; Project State decision C-065; §Directive 8–11 |
| 7 Access & hard gates | Answered | Two repos; no credentials; no money; stub not delete, so no gate |
| 8 Skills & tools | Answered | skills: orchestrate-build; tools per step |
| 9 Sequence | Answered | after Build 38; internal order 2→3→4→5→6→7→8→9 |
| 10 Assumptions & risks | Answered | §Current state; risk: a stale project prompt still names PROTOCOL.md, covered by the stub |
| 11 Design intent | Answered | §Design intent narrative |

---

## After the build (Charles)

In each claude.ai Project whose instructions tell chats to read PROTOCOL.md, replace that line with:

```
Before anything else, read agent-library/AGENT.md via Custom GitHub MCP get_file_contents (owner=Chooch333, repo=agent-library, path=AGENT.md). It routes the first message to one skill; follow what that skill's header names.
```

---

## Pasteable prompt

```
Build 39 — execute Build Brief BB-2026-10-14-protocol-retirement.
Fetch the brief from Chooch333/chat-protocol at docs/design/BB-2026-10-14-protocol-retirement.md (Custom GitHub get_file_contents); do not trust pasted text. Fetch skills/orchestrate-build/SKILL.md and follow it. Build 38 must have landed; confirm skills/ holds 39 folders before starting. Reconcile against live state. Autonomous build: decide forks from the design intent narrative and the section map, log each one, stop only at a hard gate. Close with one Session Log on chat-protocol, supersede decision C-065, disclosures on the Comms Table, and a done receipt.
```
