# Environment

**Every location this skill needs is resolved here.** `SKILL.md` describes the method and names no paths, no servers, and no sibling skills; it reads this file to find out where things are.

This file holds the *contract* — what must be known, and what to do when it is not. The **machine-local values** that satisfy it (repo roots, project-specific tooling) live in the resolved vault as `_meta/recon-environment.md`, so nothing about one person's disk or employer is committed to this plugin.

Read it before the first cycle of a session. If a field below is blank or wrong, use the fallback stated with it and tell the user which value you assumed — a wrong assumption caught in one line beats one caught after a wasted research turn.

---

## 1. Source — where the code is

**Machine-local repo roots are not listed here.** They are a fact about one
person's disk, so they live in the resolved vault as `_meta/recon-environment.md`
(see §3) — read it before the first cycle and use its root table as this
section's content.

Searched in order, always:

| Order | Root | Notes |
|---|---|---|
| 1 | the session's working directory | Only when it *is* the subject repo. Never assume it is. |
| 2+ | each root listed in `_meta/recon-environment.md` §1, in its stated order | Checkout roots, monorepo parents, workspace files. |

**Slug mismatch is normal.** A vault may name a project `foo-service` while the
directory is `acme-foo-service`. Match on substring, then confirm the directory
you picked before researching it.

*Fallback if `_meta/recon-environment.md` is absent or lists no usable root:*
resolve from the session's git root, then search upward and outward from cwd for
a directory matching the repo name, then ask. Never substitute a sibling repo
that happens to be open.

## 2. Target — where the knowledge goes

| Field | Value | Fallback if wrong or missing |
|---|---|---|
| Vault MCP server | resolved per `../../../references/vaults.md` (the plugin's vault contract) | Any `mcp__plugin_vam-vault-*` server in the session; if several, ask which. |
| Project folder | `projects/<program>/<project-slug>/` | Ask where notes for this project should live, once, and reuse the answer. |
| Shared findings | `projects/<program>/cross-cutting/` | The project folder, tagged as cross-cutting. |

## 3. The vault contract

These notes own placement, note types, frontmatter, tags, and linking. This skill does **not** define its own structure — it defers to them.

| Note | Covers |
|---|---|
| `_meta/vault-conventions.md` | Folder taxonomy, note types, frontmatter schema, naming, linking, grounding rules. §9 covers recon specifically. |
| `_meta/tag-vocabulary.md` | The closed tag set. |
| `_meta/recon-environment.md` | **This machine's local facts**: §1 repo roots, §4 project-specific tooling. Optional — absent means use the fallbacks in §1 and §4. |

*Fallback if the contract notes are missing:* stop and tell the user before writing anything. Do not improvise a taxonomy into someone's vault. If they confirm there is no contract, the minimum this skill needs is: one note per subject named for the subject, frontmatter carrying `type` / `status` / `updated` / a repo+commit stamp, and the three session artifacts in §5 below.

## 4. Optional tooling

Use when present in the session; skip silently when not. Never block a cycle on
an absent one.

| Tool | Use |
|---|---|
| `recall` skill | The wider vault sweep, when a topic spans more than the project folder. |
| `organize` skill | Owns the freshness pass that reads the `repo_ref` stamps this skill writes. |
| `Explore` / `general-purpose` subagents | Per-question research fan-out. |

Project- or company-specific tools (code-search MCPs, instruction servers,
in-house CLIs) are listed in `_meta/recon-environment.md` §4. Read that table
too and treat its rows the same way — present means usable, absent means skip.

## 5. Session artifacts

Three per project, plus the status section. Names follow the vault contract's `<project>-<subject>` convention.

| Artifact | Path | Role |
|---|---|---|
| Open questions | `<project-folder>/<project>-open-questions.md` | The resume point. |
| Contradictions | `<project-folder>/<project>-contradictions.md` | Findings that disagree. |
| Glossary | `<project-folder>/<project>-glossary.md` | One note per project, not per term. |
| Recon status | `## Recon status` in the folder's MOC | Per-level state, drills, skip/reopen reasons. |

---

## Adapting this file

This file is deliberately free of machine- and company-local facts: those live in
the resolved vault as `_meta/recon-environment.md`, next to the notes describing the
same systems, so there is one trust boundary rather than two. To adapt the skill to
another machine, write that note — not this file. Edit §2/§3/§5 here only if your
vault contract names folders or artifacts differently.

Blank template for `_meta/recon-environment.md`:

```markdown
## 1. Repo roots
| Order | Root | Notes |
|---|---|---|
| 1 | <absolute path> | <what lives here> |

## 4. Project-specific tooling
| Tool | Use |
|---|---|
| <mcp server or skill> | <when to reach for it> |
```

### Porting the skill wholesale

`SKILL.md` and `question-bank.md` are portable as they stand — neither names a
company, a path, or a server, so neither needs editing. This file is the only
one with anything to fill in. Blank template:

```markdown
## 1. Source — where the code is
| Order | Root | Notes |
|---|---|---|
| 1 | <repo root> | |

## 2. Target — where the knowledge goes
| Field | Value |
|---|---|
| Vault MCP server | <server prefix, or "none — use plain file writes to <path>"> |
| Project folder | <pattern> |
| Shared findings | <pattern> |

## 3. The vault contract
| Note | Covers |
|---|---|
| <path> | <what it governs> |

## 4. Optional tooling
| Tool | Use |
|---|---|
| <tool> | <when to reach for it> |
```

With every section blank, the skill still runs: it asks for the repo path once, asks where notes go once, and refuses to write until it knows the conventions. That degraded path is the floor, not a failure.
