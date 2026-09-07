# Diagnostics

Checks in dependency order. Each one assumes the ones above it passed.

**Stop descending when a check makes the rest unanswerable.** D1 failing means D3–D9 cannot be evaluated at all — report D1 and the plan to fix it, not nine unknowns.

Every probe here is read-only.

---

## Layer 1 — is there a vault at all

### D1 · Connector present

**Probe:** enumerate the tools in this session for the prefix `mcp__plugin_vam-vault-`.

| Result | Verdict |
|---|---|
| one match | Pass. That slug is the vault. |
| several | Pass. Resolve per `../../../references/vaults.md`; if the operation writes and it is still ambiguous, ask once. |
| none | **Fail — no connector installed or enabled.** |

**Fix (user runs it):**

```powershell
claude plugin list                                    # is it installed but disabled?
claude plugin install vam-vault-kb@vam-ai-units # or the user's own vam-vault-<slug>
```

Then restart Claude Code. Note the naming constraint while you are here: **discovery matches on the plugin name starting with `vam-vault-`.** A connector named anything else is invisible to every kb skill, however well its server works.

### D2 · Path expanded

A connector reports `Connected` whether or not its `${VAM_VAULT_<SLUG>}` expanded, so connection is not evidence of a working vault.

**Probe:** `mcp__<vault>__get_vault_stats`.

| Result | Verdict |
|---|---|
| plausible note count | Pass. |
| zero notes / empty / path error | **Suspect an unset variable.** Do not conclude "fresh vault" yet — go to the tell below. |

**The tell** — ask the user to run:

```powershell
claude mcp list
```

A literal `${VAM_VAULT_...}` in the printed command line means the variable is unset and the server is pointing at nothing.

**Fix (user runs it, then restarts):**

```powershell
[Environment]::SetEnvironmentVariable('VAM_VAULT_<SLUG>', '<absolute vault path>', 'User')
```

or as `env` in `~/.claude/settings.json`. Expansion happens at server launch, so the restart is not optional.

**Only after this check comes back clean may a zero-note vault be treated as genuinely new.** Say explicitly which of the two you concluded.

### D3 · Write tools present

**Probe:** does `mcp__<vault>__write_note` exist among the vault's tools?

| Result | Verdict |
|---|---|
| present | Pass. |
| absent | **Read-only connector** — its `args` carry `--read-only`. |

A read-only vault cannot be repaired from inside a session. Report it and stop: the fix is removing `--read-only` from that connector's `.mcp.json`, which lives in the plugin. Recall still works; capture, organize, recon and this skill's write phases do not.

---

## Layer 2 — does the contract exist

### D4 · Required contract notes

**Probe:** `mcp__<vault>__read_multiple_notes` for `_meta/vault-conventions.md` and `_meta/tag-vocabulary.md` in one call.

| Result | Verdict |
|---|---|
| both present | Pass — go to D5. |
| either missing | **Fail. This is the blocking gap:** every capture, organize and recon run refuses to write. Recall degrades to searching without tag or folder semantics. |

**Fix:** Phase 4 interview, then Phase 5 write. This is the common case on a vault that predates the plugin.

### D5 · Contract well-formed

A present-but-incomplete contract is worse than a missing one: the skills load it, find no `§6`, and fall back to improvising exactly the structure the contract was meant to fix.

**Probe:** read `_meta/vault-conventions.md` and check that each section exists **at its expected number**:

| § | Must define | Cited by |
|---|---|---|
| 1 | folder taxonomy + ordered placement rules | `capture` A3, `organize` pass 1 |
| 2 | the closed set of note types | `organize` pass 2 |
| 3 | frontmatter schema + status ladder | `capture` A5, `organize` pass 2 |
| 4 | naming rules for notes and MOCs | `organize` pass 1 |
| 5 | linking rules, incl. stub links being healthy | `organize` pass 4 |
| 6 | section skeleton per note type | `capture` A5 |
| 7 | grounding rules — pointers, `repo_ref`, repo-over-conversation | `capture` A2 |
| 9 | recon conventions | `map-project` |

`§8` (what does not become a note) and `§10` (related) are conventional, not load-bearing.

**Fix:** patch in the missing sections from the template, at their numbers. Never renumber what is there, and never rewrite a section the user already wrote.

### D6 · Tag vocabulary usable

**Probe:** read `_meta/tag-vocabulary.md`; confirm it declares the namespaces the conventions' §3 examples use — at minimum `type/*`, and whichever of `repo/*`, `tech/*`, `area/*` the vault actually needs.

| Result | Verdict |
|---|---|
| namespaces declared | Pass. |
| present but empty or namespace-less | **Fail** — a closed vocabulary with nothing in it blocks every tagged write. Patch it. |

Do **not** audit the vault's actual tags against this file here. That is `organize` pass 3.

### D7 · Recon environment — optional

**Probe:** does `_meta/recon-environment.md` exist?

Absent is **not a failure**. Only `map-project` reads it, and that skill degrades to asking for a repo path once. Report it as *not applicable* unless the user intends to use recon, then offer the template.

---

## Layer 3 — does the vault match its own contract

### D8 · Promised folders exist

**Probe:** `mcp__<vault>__list_directory` at the root; compare against the taxonomy in §1.

Report both directions — a folder in §1 that does not exist, and a top-level folder absent from §1. Neither is fatal; both mislead the next capture into a wrong placement decision.

### D9 · Entry point and MOCs

**Probe:** does `Home.md` exist? Does each top-level folder have exactly one MOC, with a globally unique basename per §4?

§5 requires every note to link up to its folder's MOC. Where the MOC does not exist, that rule cannot hold and captures will either skip the link or invent a target. Offer to scaffold.

---

## Out of scope here

Hand these to `organize` — this skill validates the contract, not the corpus:

- tags in the vault that are absent from the vocabulary
- notes in the wrong folder, orphans, broken links, duplicates
- stale `repo_ref` stamps and unverified `path:line` anchors
- oversized notes, `status: unfiled` notes sitting in `other/`
