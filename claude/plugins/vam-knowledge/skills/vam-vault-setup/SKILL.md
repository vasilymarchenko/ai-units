---
name: vam-vault-setup
description: Diagnose, repair, and bootstrap the vault side of the knowledge base — the layer the other kb skills depend on and refuse to run without. Use when a kb skill reports a missing or broken vault contract, when `_meta/vault-conventions.md` or `_meta/tag-vocabulary.md` is absent or incomplete, when no vault tools are present in the session, when the vault reads as empty or the MCP server points at the wrong path, when a new project or technology needs registering in the contract, or when the user says "set up my vault", "start a vault from scratch", "my kb isn't working", "check my vault setup", "why won't it save to Obsidian". Read-only until it has shown a plan and the user has approved it.
---

# vam-vault-setup

The other kb skills read the vault contract and refuse to write when it is missing. This skill is what makes the contract exist, and what tells the user why the vault is not working when it isn't.

**The failure mode to avoid is guessing.** A fresh empty vault and a vault whose MCP server points at a path that does not exist are indistinguishable from the inside — and improvising a taxonomy into the second one writes notes nobody will ever find again. Diagnose first, always, and say which of the two you concluded and on what evidence.

## What this owns, and what it does not

| This skill | `vam-kb-organize` |
|---|---|
| Does the contract **exist** and is it **well-formed**? | Do the **notes obey** the contract? |
| Writes and repairs `_meta/*` | Never touches `_meta/*`, except to register a tag |
| Runs when the vault is broken or brand new | Runs when the vault works and has drifted |

If the contract is sound and the complaint is about the notes, stop and hand off to `vam-kb-organize`. Do not audit note frontmatter here.

## Which vault

Resolve the target vault from the plugin's vault contract — `../../references/vaults.md`, relative to this file. Below, `mcp__<vault>__*` means that resolved server.

**This skill is the one exception to the resolution rules there.** "No vault present in the session" is a *finding* here, not the end of the road: the first checks in Phase 1 run with no vault tools at all.

## Phase 1 — Diagnose. Always, and never skipped

Work through every check in [`references/diagnostics.md`](references/diagnostics.md), in order.

Stop descending the moment a check makes the ones below it unanswerable. A missing connector makes every vault-content check moot, and reporting ten failures that all follow from one root cause is noise dressed as thoroughness.

Read-only throughout: no writes, no environment changes, no installs.

## Phase 2 — Report, then propose

State the diagnosis as **what works**, **the root cause**, **what follows from it**. One block, no preamble:

```
VAULT SETUP — obsidian-work (mcp__plugin_vam-vault-work_obsidian-work)

WORKING
  connector installed and connected · 412 notes · write tools present

BROKEN
  _meta/vault-conventions.md   missing
  _meta/tag-vocabulary.md      missing
  -> every capture will refuse. Recall works, but with no tag semantics.

NOT APPLICABLE
  _meta/recon-environment.md   absent; only vam-project-recon reads it

PLAN
  1. Interview — 6 questions, defaults offered — taxonomy, projects, tags
  2. Write both contract notes
  3. Scaffold Home.md and one MOC per top-level folder
```

Then stop for approval. Accept "all", a subset, or "just diagnose".

## Phase 3 — Repair the machine layer

Some fixes are not yours to apply. **Emit the exact command and let the user run it** for anything that changes machine configuration or needs a restart:

- setting `VAM_VAULT_<SLUG>` — a user-scope environment variable
- `claude plugin install` / `claude plugin marketplace add`
- restarting Claude Code so the server relaunches and re-expands `${...}`

The reason is not timidity. `${VAM_VAULT_<SLUG>}` is expanded **at server launch**, so a variable set inside this session changes nothing until a restart you cannot perform — applying it yourself produces a fix that appears to work and does not. Say that when you hand the command over.

What you may do: create folders inside a vault whose path already resolves, and write notes through the vault's own write tools. That is the line.

## Phase 4 — Interview

Only when a contract note has to be written or extended. The questions are in [`references/interview.md`](references/interview.md).

Rules that make it survivable:

- **Ask in one batch, numbered, with a stated default for every question.** "Defaults for all" must be a valid answer that yields a working vault.
- **Never ask what you can read.** List the vault's existing folders and the repos visible in the session *first*, then ask the user to correct a proposal rather than author one from nothing.
- **Six questions is the ceiling.** Anything finer is a later patch, not a blocker.
- **Scale the contract to the vault.** One project needs a far smaller contract than four. Do not extract structure the user has no use for yet — an unused `area/*` taxonomy is decay that ships on day one.

## Phase 5 — Write

The templates in `references/` carry the required structure, including the section numbering the other skills cite: `§1` placement, `§2` note types, `§3` frontmatter, `§4` naming, `§5` linking, `§6` skeletons, `§7` grounding, `§9` recon.

**Never renumber those sections.** `vam-kb-capture` and `vam-kb-organize` reference them by number, so a renumbered contract silently misroutes every write instead of failing.

| Template | Writes to | When |
|---|---|---|
| [`vault-conventions.template.md`](references/vault-conventions.template.md) | `_meta/vault-conventions.md` | required |
| [`tag-vocabulary.template.md`](references/tag-vocabulary.template.md) | `_meta/tag-vocabulary.md` | required |
| [`recon-environment.template.md`](references/recon-environment.template.md) | `_meta/recon-environment.md` | only if `vam-project-recon` will be used |

Procedure:

1. Read the template. Substitute every `<placeholder>` from the interview answers.
2. **Delete the `<!-- TEMPLATE ... -->` block.** A contract note still carrying it was never adapted.
3. Write with `mcp__<vault>__write_note`. Take `created` and `updated` from session context; do not shell out for the date.
4. **Scaffold what the contract now promises** — the folders named in §1, `Home.md`, and one MOC per top-level folder. A contract describing a taxonomy that does not exist on disk is the next person's confusing bug.
5. **Repairing an existing contract is a patch, never a rewrite.** Add the missing section; leave every rule the user already wrote alone, including where the template disagrees with it. Their vault, their conventions — the template is a starting point, not a standard.

## Phase 6 — Verify, out loud

Re-read what you wrote *through the vault tools*, not from memory, and confirm: both notes present, sections numbered as the skills expect, folders exist, `Home.md` links down, each MOC links back.

Close with the one thing that changed for the user — which kb skills work now that did not before.

## Hard rules

- **Diagnose before proposing, propose before writing.** Nothing is written in Phase 1 or 2.
- **Never improvise a contract into a vault that already has one.** Patch it.
- **Never renumber the contract's sections.**
- **A read-only connector cannot be repaired from here.** No write tools means say so and stop: the fix is the connector's `args`, which lives in the plugin, not the vault.
- **Never invent a project, repo, or technology the user did not name.** An empty `repo/*` table is the correct contract for a vault with no projects yet.
- **Machine configuration is emitted as commands, never applied.**
- **A vault that looks empty is a suspected misconfiguration until ruled out.** `get_vault_stats` returning nothing, plus a literal `${...}` in `claude mcp list`, is a wrong path — not a fresh vault.
