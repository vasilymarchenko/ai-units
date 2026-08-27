---
name: vam-kb-capture
description: Write durable knowledge into the user's Obsidian vault, grounded in the repository. Use when the user says "save this to my kb", "add this to Obsidian", "note this down", "remember this in my vault", or asks for something to be described/documented AND saved (e.g. "describe the architecture of this feature and save it in my kb"). Two modes - retrospective (capture what this conversation established) and generative (research the repo, then write). Do NOT use for the agent's own memory directory, for repo documentation, or for reading the vault (use vam-kb-recall for reading).
---

# vam-kb-capture

Turn what was learned into a durable note in the Obsidian vault, correctly placed, correctly linked, and grounded in the repo.

## Which vault

Before the first vault call, resolve the target vault server from the plugin's
vault contract — `../../references/vaults.md`, relative to this file. Below,
`mcp__<vault>__*` means that resolved server: with only one vault installed,
resolution is silent and `<vault>` is simply it.

## Preflight — five steps, in this order, every time

No write happens before all five are done. Skipping step 2 is what produces a vault full of near-duplicate notes.

1. **Read the contract** — `_meta/vault-conventions.md` + `_meta/tag-vocabulary.md`.
2. **Search the vault** — `mcp__<vault>__search_notes` for every candidate topic. Mandatory, even when you are confident the note does not exist.
3. **Ground in the repo** — verify each claim against `CLAUDE.md`, `docs/`, and the source.
4. **Show the plan** — the target/action table (mode A) or the outline (large mode B).
5. **Write**, then update the MOC.

The sections below are the detail for each step.

## Step 0 — always load the contract first

Read these two notes before anything else. They are the source of truth; this skill only describes *procedure*:

- `_meta/vault-conventions.md` — folder taxonomy, note types, frontmatter schema, section skeletons, grounding rules
- `_meta/tag-vocabulary.md` — the closed tag set

Use `mcp__<vault>__read_multiple_notes` with both paths in one call.

If either note is missing, stop and tell the user the vault contract is gone — do not improvise a structure.

## Choosing the mode

| Signal | Mode |
|---|---|
| The knowledge is already in this conversation ("save what we just did") | **A — retrospective** |
| The user asks for something to be described, explained, or documented and saved, with no prior work in this session | **B — generative** |

When both apply (a conversation happened *and* the user asks for a broader write-up), run A for what the session established and B for the gaps, then present one combined plan.

---

## Mode A — retrospective

### A1. Extract candidate items

Re-read the conversation and pull out the durable facts. Each candidate must pass **both** tests — if either fails, drop it:

1. **Would the user look this up again in a month?** (No → it is passing trivia or one-off command output.)
2. **Is it absent from the repo's own `CLAUDE.md` / `docs/`?** (No → link to that doc instead of copying it into the vault.)

**A conversation usually yields several items belonging to different notes in different folders.** That is the normal case, not the exception. Never force multiple topics into one note.

### A2. Ground each item in the repo

For every item, search the repository before writing. Use `Grep`/`Glob`/`Read` over `CLAUDE.md`, `docs/`, and the actual source. Each item resolves to one of:

- **confirmed** — write it with a `path:line` anchor
- **enriched** — the repo knows more than the conversation did (real flag name, adjacent gotcha, canonical doc). Write the fuller version.
- **contradicted** — record what the repo says *and* flag the conflict. The repo wins on anything checkable; do not preserve a conversational error.
- **unverifiable** — write it marked `*(unverified)*` inline.

Capture `repo_ref` (repo name + current commit SHA + date) for any note carrying repo-derived claims. Get the SHA with `git -C <repo> rev-parse --short HEAD`.

### A3. Route each item

For each item, in order:

1. `mcp__<vault>__search_notes` for the topic. Also check `mcp__<vault>__list_all_tags` when unsure which tag applies.
2. Decide the target folder using the placement rules in §1 of the conventions.
3. Decide **patch vs new** by rule, not by preference: **if the search in step 1 returned any note whose topic overlaps this item, patch that note. Create a new note only when the search returned zero overlapping notes.** Creating is always the easier option and is almost always the wrong one — a vault grows by notes getting deeper, not by getting more numerous.
4. If classification is genuinely uncertain, target `other/` with `status: unfiled`. Never guess a project folder.

### A4. Present the plan, then write

Show a table and stop for approval:

| Fact | Target note | Action | Grounding |
|---|---|---|---|
| … | `projects/billing/payments-api/local-stack.md` | patch | confirmed `compose.py:141` |
| … | `projects/billing/workspace/workspace-bootstrap.md` | new | enriched from `CLAUDE.md` |

**Do not re-print the note prose in mode A** — the facts are already in the conversation and repeating them is pure waste. The table is the decision point: right target, patch or new.

### A5. Write

- Patch with `mcp__<vault>__patch_note` (surgical) or `write_note` with `mode: append` (new section).
- Create with `mcp__<vault>__write_note`, using the type's section skeleton from §6 of the conventions.
- Set/refresh frontmatter per §3. Always update `updated`. On a patch, never downgrade `status`.
- Add the `## Related` link up to the folder's MOC.
- **Cross-link siblings**: items captured from this same session share a `source` and link to each other.
- Update each affected MOC's note list.

---

## Mode B — generative

### B1. Clarify, but only what changes the outcome

Ask at most two questions, and only when the answer changes what gets written: the feature boundary, the depth (component-level vs. call-flow), whether it spans repos. If the request is already unambiguous, skip this and say nothing.

### B2. For a large or cross-repo subject, show the outline first

Before a deep investigation, present a 5-line outline and confirm. This is cheap and prevents researching the wrong thing — which is the failure that actually costs tokens. For a small, single-repo subject, skip straight to B3.

### B3. Investigate the repo

This is real work, not a formality. Read the docs cascade (`CLAUDE.md`, `docs/`) and then the code itself. Same grounding rules as A2: `path:line` anchors, `repo_ref`, `*(unverified)*` markers, conflicts recorded.

### B4. Write first, brief second

Write the note, then report **3–6 bullets** in chat: what the note claims, what is uncertain, what deserves a second look — plus the vault path.

Do not render the note body in chat. The user reviews it in Obsidian; corrections land as patches, not rewrites.

### B5. Status discipline

A mode-B note describes something the user has read about but not necessarily done. Set `status: seed`. It graduates to `working` when a later mode-A capture confirms it in practice. The vault must reflect what the user actually knows versus what was merely researched.

An architecture note is `type: concept` and gets the full skeleton — including a **mermaid diagram** in `## Flow`, which Obsidian renders natively. A wall of prose is not a usable architecture note.

---

## Hard rules

- **Never invent a fact.** Conversation and repository are the only sources.
- **The repo outranks the conversation** on anything checkable.
- **Search the vault before writing.** Skipping this is how a vault ends up with forty near-duplicate notes.
- **Patch over create.**
- **One topic per note**; many notes per session is expected.
- **Tag from the closed vocabulary.** A genuinely new tag must be added to `_meta/tag-vocabulary.md` in the same write.
- **Every note links up to its MOC**, and the MOC gets the reciprocal entry.
- **Never touch** `_meta/` contract notes as part of a capture, except to register a new tag.
