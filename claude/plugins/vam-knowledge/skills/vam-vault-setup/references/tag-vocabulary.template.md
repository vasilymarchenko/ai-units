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
     Written to _meta/tag-vocabulary.md by vam-vault-setup.
     type/* is fixed: it mirrors §2 of vault-conventions and must match it exactly.
     repo/*, tech/*, area/* come from the interview. An empty section is correct
     for a vault that has no projects yet — keep the heading, drop the rows. -->
# Tag Vocabulary

A **closed, namespaced** tag set. A closed vocabulary is what keeps the vault queryable; free-form tags are how vaults rot.

**Adding a tag is allowed** — but the new tag must be added to this file in the same write that first uses it. `vam-kb-organize` flags any tag in the vault that is absent here.

## `type/*` — mirrors the `type` frontmatter field

`type/cheatsheet` · `type/concept` · `type/flow` · `type/runbook` · `type/troubleshooting` · `type/decision` · `type/reference` · `type/moc`

Fixed by §2 of [[vault-conventions]]. Changing this list means changing §2 and §6 in the same write.

## `repo/*` — which repository the knowledge belongs to

The slug is the vault's name for the project; the repository is its directory or remote name. They frequently differ, and this table is what lets a skill match one to the other.

| Tag | Repository |
|---|---|
| `repo/<slug>` | `<repository-name>` |

## `tech/*` — technology

`tech/<x>` · `tech/<y>`

What has actually been written about, not everything in use. Additions are cheap.

## `area/*` — problem domain

`area/<domain>` · `area/vault-meta`

`tech/*` is *what it is built with*; `area/*` is *what problem it solves*. A note about debugging auth in containers carries both. `area/vault-meta` is reserved for these contract notes.

## Rules

- Lowercase, singular, one `/` level of nesting.
- No tag duplicates information already held in a frontmatter field, except `type/*` — kept because tag search is the fastest path in the Obsidian UI.
- `status` is **frontmatter only** — never a tag.

## Related

- [[vault-conventions]]
- [[Home]]
