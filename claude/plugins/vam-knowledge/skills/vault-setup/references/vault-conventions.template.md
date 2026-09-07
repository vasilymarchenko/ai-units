---
type: reference
project: ""
tags:
  - type/reference
  - area/vault-meta
status: stable
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
source: <conversation YYYY-MM-DD — vault setup>
---
<!-- TEMPLATE — delete this block once adapted.
     Written to _meta/vault-conventions.md by vault-setup.
     Substitute every <placeholder> from the interview answers.
     NEVER renumber the sections: capture and organize cite
     them by number (§1 placement, §3 frontmatter, §4 naming, §5 linking,
     §6 skeletons, §7 grounding, §9 recon).
     Delete a section's *rows* if unused; never delete the section. -->
# Vault Conventions

The contract every note in this vault follows. The `capture`, `organize` and `recall` skills read this file as their source of truth — if a rule changes, change it **here**, not in the skills.

Companion: [[tag-vocabulary]]. Entry point: [[Home]].

## 1. Folder taxonomy

```
<general/>                        # org, process, tools, people — omit if unused
tech-general/                     # technology knowledge not tied to one project
projects/
  <program>/                      # a program with several repos
    <repo-slug>/                  # one folder per repo
    cross-cutting/                # spans repos of this program
other/                            # unfiled / temporary / not yet classified
_meta/                            # this contract
```

**Placement rules, in order:**

1. Knowledge about **one repo** → that repo's folder.
2. Knowledge that **spans repos** of a program → that program's `cross-cutting/`.
3. Knowledge about the **workspace machinery** (manifests, profiles, CI policy, orchestration) → `<program>/<workspace-repo>/`. This is *not* the same as `cross-cutting/`: workspace knowledge is *owned by* the workspace repo; cross-cutting knowledge is owned by *no single* repo.
4. Reusable **technology** knowledge that would still be true elsewhere → `tech-general/`.
5. **Org** context, tools and process → `<general>/`.
6. Cannot classify confidently → `other/` with `status: unfiled`. Never guess a project folder.

## 2. Note types

A closed set. The type determines the section skeleton — see §6.

| Type | Purpose |
|---|---|
| `cheatsheet` | Commands, flags, snippets. Optimized for lookup, not reading. |
| `concept` | How something works: architecture, data flow, model. |
| `flow` | A traced execution path end to end: ordered steps, each naming a file or symbol. |
| `runbook` | Reproducible procedure, ordered steps, verifiable outcome. |
| `troubleshooting` | Symptom → cause → fix. One note may hold many symptoms. |
| `decision` | A choice made, its alternatives, and why. Mirrors an ADR. |
| `reference` | Link hub, external resources, glossary. |
| `moc` | Map of Content — a folder's index note. |

## 3. Frontmatter schema

```yaml
---
type: cheatsheet            # one of §2, required
project: <repo-slug>        # or "" when not repo-scoped
tags: [type/cheatsheet, repo/<slug>, tech/<x>]
status: seed                # seed | working | stable | unfiled
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
source: "conversation <date> | PR #<n> | <ticket>"
repo_ref:                   # only when claims are grounded in code
  - repo: <repository>
    commit: <short-sha>
    date: <YYYY-MM-DD>
---
```

**Status ladder:**

- `seed` — written from reading or research; not yet confirmed by doing it.
- `working` — confirmed at least once in practice; may be incomplete.
- `stable` — trusted; used repeatedly without correction.
- `unfiled` — needs classification. Only valid inside `other/`.

Keys are flat and consistently named so Dataview can query them once installed. Until then, MOCs carry manual link lists — see §5.

## 4. Naming

- Content notes: `kebab-case.md`, descriptive, no dates in the name.
- MOC notes: `Title Case MOC.md`, one per folder, **globally unique basename** so `[[<Program> MOC]]` resolves without a path.
- One topic per note. Notes grow by being patched, not by being duplicated.

## 5. Linking

- Every content note links **up** to its folder's MOC, in a `## Related` section.
- Every MOC lists its notes, grouped by type.
- Notes captured in the **same session** share a `source` and get cross-linked — that provenance edge records *why these things came up together*, often the most valuable link in the graph.
- Links are `[[wiki-links]]` by basename. Because basenames are unique (§4), paths are not needed.
- A `[[link]]` to a note that does not exist yet is **allowed and encouraged** — it marks a gap worth filling. `organize` reports these as intentional stubs, not errors.

## 6. Section skeletons by type

**cheatsheet** — `## Quick reference` (table) · `## Commands` · `## Gotchas` · `## Related`

**concept** — `## Purpose` · `## Components` · `## Flow` (mermaid diagram) · `## Boundaries` · `## Key files` · `## Decisions` · `## Open questions` · `## Related`

**flow** — `## Trace` (ordered steps, each naming a file or symbol) · `## Boundaries` (transactions, async handoffs, external calls) · `## Confirmed / contradicted` (what the trace proved or falsified about the notes above it) · `## Related`

**runbook** — `## Preconditions` · `## Steps` · `## Verification` · `## Failure modes` · `## Related`

**troubleshooting** — `## Symptom → Cause → Fix` (table), then one `###` per symptom with detail · `## Related`

**decision** — `## Context` · `## Decision` · `## Alternatives` · `## Consequences` · `## Related`

**reference** — free-form grouped links · `## Related`

## 7. Grounding rules

Repo-derived claims are the backbone of this vault, so they carry provenance:

- Every claim checkable against code cites `path:line` or a doc path.
- The note records `repo_ref` (repo + commit + date) so staleness is detectable later.
- The **repo is authoritative over conversation** on anything checkable. A conversation can be wrong; the note must not preserve the error.
- A claim that could not be verified is marked `*(unverified)*` inline rather than stated flat.
- A conflict between conversation and repo is **recorded as a conflict**, not silently resolved.

## 8. What does not become a note

Passing trivia, one-off command output, anything already documented in a repo's `CLAUDE.md` or `docs/` (link to it instead of copying), and anything only meaningful inside a single conversation. A note must be **durable** and **reusable**.

## 9. Learning a new project (recon)

The `map-project` skill runs a structured interview to map an unfamiliar codebase. Its output obeys everything above — it does **not** define a second taxonomy. Three additional conventions apply:

**Knowledge lands by subject, not by level.** A recon proceeds through levels (purpose → boundaries → structure → domain → flow → …), but the level is a property of the *session*, not of the vault. What it learns about architecture becomes `<repo-slug>-architecture.md` (`type: concept`), not `Level 3 Structure.md`. A vault organized by the order you happened to learn things ages badly.

**Session state lives in three notes, one set per project being reconned:**

- `<project>-open-questions.md` (`type: reference`) — everything unanswered, unpointered or deferred. This is the resume point, so write each item for a reader who has forgotten everything: the question, why it matters, what would answer it.
- `<project>-contradictions.md` (`type: reference`) — where two findings disagree, or a finding diverges from its own pointer. Each entry: the two claims, their sources, the current hypothesis. **Never resolve one silently.** A contradiction usually means an undocumented special case, a stale document, or a genuine bug — all three are worth knowing.
- `<project>-glossary.md` (`type: reference`) — one glossary per project, not one note per term. For each term: the definition in the code's terms, how the business uses the word, and where the two diverge. The divergences are the reason the note exists.

The folder's MOC carries a `## Recon status` section: per-level state (unmapped / open / mapped), drills done, and the reason behind any level skipped or reopened.

## 10. Related

- [[tag-vocabulary]]
- [[Home]]
