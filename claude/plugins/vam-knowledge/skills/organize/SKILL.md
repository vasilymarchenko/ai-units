---
name: organize
description: Garden and audit the user's Obsidian vault - find orphans, duplicate topics, off-vocabulary tags, stale repo-derived claims, oversized notes needing a split, unfiled notes in other/, and missing MOC entries. Proposes a plan, waits for approval, then applies it. Use when the user asks to organize/clean up/audit/garden the vault, split a note that got too big, check whether notes are still accurate, or file what is sitting in other/. Read-only until the user approves the plan.
---

# Organize

Periodic gardening for the vault. A knowledge base decays in predictable ways; this skill finds each kind of decay, proposes a fix, and applies it only after approval.

## Which vault

Before the first vault call, resolve the target vault server from the plugin's
vault contract — `../../references/vaults.md`, relative to this file. Below,
`mcp__<vault>__*` means that resolved server: with only one vault installed,
resolution is silent and `<vault>` is simply it.

## Step 0 — load the contract

Read `_meta/vault-conventions.md` and `_meta/tag-vocabulary.md` first (`mcp__<vault>__read_multiple_notes`). Every check below is measured against those two notes, not against this skill's opinion.

Then survey: `mcp__<vault>__get_vault_stats`, `mcp__<vault>__list_directory` (recurse the tree), `mcp__<vault>__list_all_tags`.

## The audit passes

Run all of them unless the user names a specific one.

### 1. Structure

- Notes in the wrong folder per the §1 placement rules — especially the `meta-repo/` vs `cross-cutting/` distinction (owned-by vs spans).
- Notes still sitting at the vault root.
- Folders missing their MOC; MOC basenames that are not globally unique.
- Filenames off convention (§4): content notes `kebab-case`, MOCs `Title Case MOC`.

### 2. Frontmatter

- Missing or invalid `type` / `status`; a `type` outside the closed set.
- `status: unfiled` outside `other/`.
- `updated` older than the file's real modification time.
- `project` inconsistent with the note's folder.

### 3. Tags

- Any tag in the vault absent from `_meta/tag-vocabulary.md`. Either add it to the vocabulary or remap the note — never leave it dangling.
- `status` used as a tag (forbidden — frontmatter only).
- Near-duplicate tags (`tech/k8s` vs `tech/kubernetes`).

### 4. Graph health

- **Orphans** — notes with no inbound links and no MOC entry. These are invisible in practice.
- **Missing MOC entries** — the note links up but the MOC does not list it back.
- **Broken links** — `[[link]]` to a nonexistent note. Distinguish deliberately: a stub link marking a known gap is *healthy* (§5) and should be reported as a gap to fill, not an error. A link pointing at a note that was renamed or deleted is a real break.

### 5. Duplication

Notes covering the same topic under different names. Propose a merge with an explicit winner and a redirect, or merge-and-delete. Never merge silently — the user may have split them on purpose.

### 6. Freshness — the pass that matters most

For every note carrying `repo_ref` or a `path:line` anchor, re-check the claim against the current checkout:

- Cited file or line no longer exists → **stale**, flag it in the note and in the report.
- File changed materially since the recorded commit → **needs re-verification**.
- Claim still holds → refresh `repo_ref` to the current commit.

Also flag `status: seed` notes older than ~60 days: researched but never confirmed in practice.

This is what stops a vault of repo facts from rotting invisibly.

### 7. Size and splitting

A note is a split candidate when it is very long, or carries several `##` sections that are really separate topics, or mixes types (a cheatsheet that grew a runbook inside it).

Splitting is the one operation that can quietly break the link graph, so it is always propose → approve → apply:

1. Propose the parent hub plus the children, with each child's name, type, and which sections move into it.
2. On approval: create the children, reduce the parent to a hub with links down, **rewrite every inbound link** that pointed at the moved content, update the MOC.
3. Report any inbound link that could not be redirected automatically.

### 8. `other/` triage

For each note in `other/`, propose a destination folder, type, and tags. An empty `other/` is the healthy state.

## Output contract

Report findings grouped by pass, each with a concrete proposed action:

```
STRUCTURE     2 findings
  - docker-commands.md sits in tech-general/ but every example is stack-specific
    -> propose: keep, add link to [[local-stack-bootstrap]]  (or move to cross-cutting/)
FRESHNESS     1 finding
  - local-stack-bootstrap.md cites docs/local-oidc-bootstrap.md, no repo_ref
    -> propose: verify against current checkout, stamp repo_ref
```

Then ask which findings to apply. Accept "all", a subset, or none.

## Hard rules

- **Read-only until approved.** No write, move, or delete before the user picks.
- **Never delete a note** without explicit per-note approval. Prefer emptying-and-redirecting over deletion.
- **Never bulk-rewrite frontmatter** across the vault in one sweep without showing the exact diff for at least a representative sample.
- Splits and merges always rewrite inbound links in the same operation. A dangling link created by this skill is a bug.
- If a fix would require inventing a fact, report the gap instead and leave the note alone.
