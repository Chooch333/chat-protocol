# Build Brief — BB-2026-09-09-one-front-door

**What this is:** Make every chat start the same simple way — read one file, get pointed to the right role — instead of today's split where some chats are triggered by instructions hidden in the project prompt and others are routed through a table.

**What I'll do (the build):** Edit two repo files so the single front door works, then hand Charles the short pointer text to paste into his project instructions.

**What you'll do:** Approve this brief. After the build ships, paste the small pointer text (Part B below) into your Claude project instructions yourself — that's the one piece a build can't do for you.

---

## Current state going in

Today chats activate two different ways:

- **Design Assist** and the review roles are already routed properly through the table in `agent-library/AGENT.md`. (Verified — DA is in that table with the exact trigger "Design Assist.")
- **Brainstorm** is different: it's switched on by a block of instructions sitting inside Charles's project prompt (and echoed in PROTOCOL.md) that says "if the chat opens with 'this is a brainstorm chat,' go fetch BRAINSTORM.md."
- **A plain build chat** is just the default — no trigger, it's what a chat is when nothing else fires.

So the same basic act — a chat reads a file and takes on a role — happens two different ways. The goal is one way.

**The model (settled this session):** every chat reads `PROTOCOL.md` first. PROTOCOL.md is the rulebook, and it points onward to the role table in `AGENT.md` for which role to load. Rules stay in PROTOCOL.md; the list of roles and their triggers stays in AGENT.md; the project prompt shrinks to a single pointer at PROTOCOL.md. Brainstorm becomes a normal row in the AGENT.md table like every other role. Build stays the unnamed default.

---

## Receiving chat

New build chat to be opened.

---

## Scope

**In scope (the two file edits):**
1. Add a **Brainstorm** row to the AGENT.md routing table so it's activated the same way every other role is.
2. Update PROTOCOL.md so it clearly names AGENT.md as the place a chat goes to find its role, and so its "Chat types" section reflects the one-front-door model instead of the old hardcoded-trigger wording.

**Out of scope (do NOT do these):**
- Do **not** touch any of the 19 existing role files.
- Do **not** edit Charles's project instructions — that's Charles's to do by hand (Part B). The build produces the text; Charles pastes it.
- Do **not** remove the old brainstorm block from anywhere until the reliability check below passes.
- Do **not** merge, rename, or move any repo.

**Forks pre-answered:**
- *Should build get its own trigger row?* No — build stays the default (a chat with no matching trigger is a build chat). A trigger nobody types is clutter.
- *Should PROTOCOL.md absorb the routing table?* No. Rules and routing stay in separate files (PROTOCOL.md → points at → AGENT.md). One front door, two files behind it.

---

## Directive

**The one thing to verify before anything is removed:** the old brainstorm trigger is hardcoded in the project prompt because that fires reliably on the very first message. Before deleting it, confirm the routed path fires just as reliably — i.e. a chat that reads PROTOCOL.md, follows it to AGENT.md, and matches "this is a brainstorm chat" actually loads BRAINSTORM.md dependably on message one. This is the first task, not an afterthought. If routed activation proves flaky, keep the hardcoded brainstorm trigger and stop — report that finding rather than shipping a less-reliable setup.

If the check passes:
1. **AGENT.md** — add a "Brainstorm" row (new sub-section or under an existing one) with trigger phrase `this is a brainstorm chat` → `chat-protocol/BRAINSTORM.md`. Note in the row that BRAINSTORM.md lives in the chat-protocol repo, not agent-library. Add a changelog line.
2. **PROTOCOL.md** — in the "Chat types" section, replace the brainstorm-specific hardcoded instruction with the general rule: every chat reads PROTOCOL.md, then consults AGENT.md for its role; brainstorm is one such role; a chat with no role match is a build chat. Keep the rulebook content itself unchanged.
3. Produce the final pointer text for Charles's project instructions (Part B) and post it as the build's closing deliverable.

Log every judgment call to the Comms Table per protocol. Close with a Session Log on the `chat-protocol` project.

---

## Inputs

- `agent-library/AGENT.md` (current SHA a01846eb) — the routing table.
- `chat-protocol/PROTOCOL.md` (current SHA b1da7a55) — the rulebook; see its "Chat types" and "Standing rules" sections.
- `chat-protocol/BRAINSTORM.md` — the brainstorm workflow the row points at.
- This session's decision that the model is Option B (one front door, rules and routing in separate files).

---

## Pasteable prompt

> This is a Design Assist chat. Fetch and follow `roles/design-assist/SKILL.md` via Custom GitHub MCP (owner=Chooch333, repo=agent-library, path=roles/design-assist/SKILL.md), and read PROTOCOL.md per standing rules.
>
> Then execute Build Brief `BB-2026-09-09-one-front-door`, fetched from its git home: owner=Chooch333, repo=chat-protocol, path=docs/design/BB-2026-09-09-one-front-door.md. Reconcile it against current Project State before acting. Start with the reliability check in the Directive — do not remove any existing trigger until it passes.

---

# ===========================================================
# THE TWO HALVES — clearly separated
# ===========================================================

## PART A — What the build does (in the repos, no action from you)

1. Adds a **Brainstorm** row to the role table in `AGENT.md`.
2. Rewrites the **"Chat types"** wording in `PROTOCOL.md` to say plainly: read PROTOCOL.md, then check AGENT.md for your role; brainstorm is a role; no match means build chat.
3. Verifies the new routed path fires reliably **before** removing the old brainstorm block.
4. Hands you the pointer text below as its closing deliverable.

You do nothing here. It happens in the repos.

## PART B — What only you can do (paste into project instructions)

Once the build ships, replace the brainstorm/DA activation block currently in your Claude **project instructions** with this short pointer. This is the whole thing every chat needs:

```
Before doing anything else, read PROTOCOL.md from GitHub:
owner=Chooch333, repo=chat-protocol, path=PROTOCOL.md

That file is the rulebook. Follow it. It will tell you how to
pick up the right role for this chat (for example, a chat that
opens with "this is a brainstorm chat" becomes a brainstorm
chat). If nothing special is named, it's an ordinary build chat.
```

That's it. No trigger lists to maintain in the project prompt anymore — when you add or change a role later, you edit the table in AGENT.md, and the pointer above never has to change.

---

## Result (fill in after execution)

**What got changed:** [files + commits]
**Reliability check outcome:** [passed / failed — with what was observed]
**Pointer text delivered to Charles:** [yes/no]
