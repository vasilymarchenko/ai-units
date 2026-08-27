---
name: vam-kb-recall
description: Read the user's own prior notes out of the Obsidian vault into the current session, before re-solving a problem they already solved. Use when starting work on a topic the user may have notes on, when they ask "what do I know about X", "check my kb/vault/notes", "did I write anything about this", or when a task touches a project, service, or repository the user works on regularly and prior context would help. Strictly read-only - never writes to the vault.
---

# vam-kb-recall

The read path. A knowledge base you only write to is a diary; this skill is what makes it a knowledge base.

## Which vault

Before the first vault call, resolve the target vault server from the plugin's
vault contract — `../../references/vaults.md`, relative to this file. Below,
`mcp__<vault>__*` means that resolved server: with only one vault installed,
resolution is silent and `<vault>` is simply it.

## Procedure

1. **Load the contract** — `_meta/vault-conventions.md` and `_meta/tag-vocabulary.md`, so tag and folder semantics are known. (`mcp__<vault>__read_multiple_notes`)

2. **Search along several axes**, not one. Any single axis misses notes:
   - `mcp__<vault>__search_notes` for the topic terms and their synonyms
   - the relevant folders via `mcp__<vault>__list_directory` — the topic's repo folder, plus `cross-cutting/`, plus `tech-general/`
   - tags: `mcp__<vault>__list_all_tags`, then the matching `repo/*`, `area/*`, `tech/*`
   - the folder's MOC, and the `## Related` links of anything that hits — the graph is the index

3. **Read what looks relevant** — `read_multiple_notes` for full notes, `get_note_outline` when only structure is needed to decide.

4. **Report with the trust level attached.** This is the part that matters: a `status: seed` claim and a `status: stable` one must not be presented as equally reliable.

```
FROM YOUR VAULT — "local stack bootstrap"

[stable]  junctions-vs-symlinks-cheatsheet
          Junctions need no admin rights and are directories-only.
[working] local-stack-bootstrap
          AUDIT_FULL_ENCRYPTION_KEY is required though undocumented — the
          api-server will not boot without it.
[seed]    (none)

STALE / WORTH RE-CHECKING
  - local-stack-bootstrap has no repo_ref: claims date from 2026-08-21 and are
    unverified against the current checkout.

GAPS
  - nothing in the vault on the OIDC token flow itself, only the bootstrap.
```

5. **Flag staleness rather than trusting blindly.** If a note cites a `path:line`, spot-check that the path still exists before relying on it in the current task. A confidently-quoted stale note is worse than no note.

## Hard rules

- **Read-only.** Never write, patch, move, or delete. If the session produces something new worth keeping, say so and point at `vam-kb-capture` — do not write it here.
- **Never present a vault claim as current fact without its status and age.**
- **Report gaps explicitly.** "Nothing in the vault about X" is a useful answer and a prompt for a future capture.
- Quote the user's own notes; do not paraphrase them into something they did not say.
