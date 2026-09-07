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
     Written to _meta/recon-environment.md by vault-setup.
     OPTIONAL: only map-project reads it, and that skill degrades to
     asking for a repo path once per session when it is absent.
     Section numbers match map-project/references/environment.md —
     §1 repo roots, §4 project-specific tooling. Do not renumber. -->
# Recon Environment

**Machine-local facts for `map-project`.** The `vam-knowledge` plugin is public and project-agnostic, so its `references/environment.md` carries only the *contract* — what must be known and what to do when it is not. The values that satisfy it live here, next to the notes describing the same systems.

Read this before the first recon cycle of a session. Sections are numbered to match `environment.md`. If a section is missing, use that section's fallback and say which value was assumed.

## 1. Repo roots

Searched after the session's working directory, in this order:

| Order | Root | Notes |
|---|---|---|
| 1 | `<absolute path>` | `<what lives here — a metarepo checkout, a workspace parent, loose clones>` |

**Slug mismatch is normal.** This vault may name a project `<slug>` while the directory is `<repository-name>`. Match on substring, then confirm the directory you picked before researching it. The `repo/*` table in [[tag-vocabulary]] holds the full slug-to-directory mapping.

## 4. Project-specific tooling

Use when present in the session; skip silently when not. Never block a cycle on an absent one.

| Tool | Use |
|---|---|
| `<mcp server or cli>` | `<when to reach for it — and what it beats>` |

The portable tooling — `recall`, `organize`, and the `Explore` and `general-purpose` subagents — is listed in the plugin's own `environment.md` §4 and needs no entry here.

## Why this note exists

The plugin is personal, project-agnostic tooling intended to be public. Repo roots on one person's disk, and an employer's internal service names, are neither portable nor publishable — they are configuration, not source. Keeping them in the vault puts them behind the same trust boundary as the notes that already describe those systems: one boundary instead of two. A private companion *repo* could not have served this purpose anyway, since a plugin cannot override another plugin's internal reference files.

## Related

- [[vault-conventions]]
- [[tag-vocabulary]]
