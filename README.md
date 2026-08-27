# ai-units

Personal AI units — skills, and later agents and commands, kept in one repo and installed on any machine from a single source.

> **Personal project. Not affiliated with, endorsed by, or containing material
> belonging to any employer, past or present.**

Everything here is personal and project-agnostic. Only the *method* is committed:
every local fact — vault paths, repo roots, taxonomies, in-house tooling — is
resolved at runtime from environment variables and from notes that live inside
the vault, never from this tree. Units distributed by an organisation belong in
that organisation's own marketplace, not here.

## Layout

```
ai-units/
├── LICENSE                   # MIT
├── .claude-plugin/
│   └── marketplace.json      # Claude Code plugin marketplace (must sit at repo root)
├── claude/                   # everything Claude Code specific
│   └── plugins/
│       ├── vam-knowledge/    # skills, vault-agnostic
│       ├── vam-session/      # skills
│       └── vaults/           # one connector plugin per vault
│           └── vam-vault-template/  # copy per vault; real ones need not be committed
└── codex/                    # reserved for Codex / other harnesses
```

One folder per AI harness. `.claude-plugin/marketplace.json` is the single
exception to that rule: Claude Code discovers a marketplace only at the repo
root, so the manifest stays there while every unit it points at lives under
`claude/`.

## Install on a fresh machine

Seven steps. Order matters in exactly one place — step 1 before step 4 — and the
reason is explained there.

**Prerequisites**

| Need | Why | Check |
|---|---|---|
| Node.js | the vault connector runs `npx` | `node -v` |
| a vault folder | any directory of markdown files | it exists |
| Claude Code | the harness | `claude --version` |

Obsidian itself is **not** a prerequisite and does not need to be running. The
connector reads the vault as plain files on disk; Obsidian is just the editor you
happen to point at the same folder. Nothing is installed inside Obsidian.

### 1. Set the vault path

```powershell
[Environment]::SetEnvironmentVariable('VAM_VAULT_EXAMPLE', 'D:\Notes\MyVault', 'User')
```

or, equivalently, in `~/.claude/settings.json`:

```json
{ "env": { "VAM_VAULT_EXAMPLE": "D:\\Notes\\MyVault" } }
```

`VAM_VAULT_EXAMPLE` is the variable the shipped connector reads — that exact
name, unless you copy the connector under a new slug (see *Units* below).

**Do this first.** `${VAM_VAULT_EXAMPLE}` is expanded when the MCP server
*launches*, not when the plugin installs, so a value set after Claude Code is
already running has no effect until the restart in step 4. Setting it now means
one restart instead of two.

### 2. Register the marketplace

```powershell
claude plugin marketplace add vasilymarchenko/ai-units
```

This adds the *listing* only; nothing is installed yet. Equivalent sources, when
the shorthand will not do — a local clone you are iterating on, or a private repo
`gh auth login` has to reach first:

```powershell
claude plugin marketplace add C:\Work\Personal\ai-units
claude plugin marketplace add https://github.com/vasilymarchenko/ai-units.git
```

### 3. Install the plugins

```powershell
claude plugin install vam-knowledge@vam-ai-units        # the kb skills
claude plugin install vam-vault-template@vam-ai-units   # mounts the vault
claude plugin install vam-session@vam-ai-units          # optional: handoffs
```

`vam-knowledge` carries no vault of its own, so the first two are a pair — skills
alone have nothing to read. `vam-session` is independent and needs no vault.

For a second vault, copy the connector rather than installing the template
twice: [vaults.md](claude/plugins/vam-knowledge/references/vaults.md#adding-a-vault).

### 4. Restart Claude Code

The MCP server starts with the session, so this is what makes step 1 take
effect. Skipping it is the single most common reason a correct setup looks broken.

### 5. Verify the path actually resolved

```powershell
claude mcp list
```

```
plugin:vam-vault-template:obsidian-example: npx @bitbonsai/mcpvault@0.16.0 D:\Notes\MyVault - Connected
```

**Read the path, not the word `Connected`.** An unset variable is passed through
unexpanded and the server still reports success — a literal `${VAM_VAULT_EXAMPLE}`
in that line is the tell, and it means the vault is mounted at a path that does
not exist. There is no "missing environment variable" diagnostic for
plugin-provided servers.

### 6. Bootstrap the vault contract

In Claude Code:

```
/vam-vault-setup
```

The skills defer to two notes inside the vault — `_meta/vault-conventions.md`
(folder taxonomy, note types, frontmatter schema) and `_meta/tag-vocabulary.md`
(the closed tag set) — rather than imposing a structure on someone's vault.
**Until those exist, every write refuses**, by design.

`vam-vault-setup` diagnoses what is missing, asks about six questions with a
default for each, writes both notes, and scaffolds the folders and MOCs they
describe. It is also the thing to run when something breaks later: it checks the
connector, the path expansion, read-only mode, and whether the contract is
complete. An existing vault is patched, never overwritten.

### 7. Use it

| Say | Skill |
|---|---|
| "what do I know about X" | `vam-kb-recall` |
| "save this to my kb" | `vam-kb-capture` |
| "organize / audit my vault" | `vam-kb-organize` |
| "help me map this codebase" | `vam-project-recon` |
| "hand this off" | `vam-handoff` |

Skills trigger from intent, so invoking them by name is optional.

### If something is wrong

| Symptom | Likely cause |
|---|---|
| no vault tools in session | connector not installed or not enabled — `claude plugin list` |
| `${...}` in `claude mcp list` | variable unset, or set after launch without a restart |
| "the vault contract is missing" | step 6 not done |
| writes refused, reads fine | connector has `--read-only` in its `args` |
| a skill appears twice | same skill in `~/.claude/skills/` *and* a plugin — delete the personal copy |

`/vam-vault-setup` diagnoses all of these; the table is for when you would rather
not ask.

### A note on the pinned connector

The connector pins `@bitbonsai/mcpvault@0.16.0`. It is a pre-1.0, single-maintainer
package, so a minor bump may break compatibility, and `npx` would pick that up
silently at the next session start.

`@latest` works and is a legitimate choice — you get fixes without touching the
repo, at the cost of an unannounced upgrade landing between two sessions. To take
it, change the version in your connector's `.mcp.json`:

```json
"args": ["@bitbonsai/mcpvault@latest", "${VAM_VAULT_EXAMPLE}"]
```

Either way the value lives in the connector plugin, so the choice is per vault.
## Units

| Plugin | Kind | Contents | Needs |
|---|---|---|---|
| `vam-knowledge` | skills | `vam-vault-setup`, `vam-kb-recall`, `vam-kb-capture`, `vam-kb-organize`, `vam-project-recon` | a vault connector below |
| `vam-session` | skills | `vam-handoff` | nothing |
| `vam-vault-template` | connector | template for an `obsidian-<slug>` MCP server | `npx`, `VAM_VAULT_<SLUG>` |

### Skills and vaults are separate units

`mcpvault` serves exactly one vault per process, so **one vault = one MCP server
= one connector plugin** under `claude/plugins/vaults/`. `vam-knowledge` carries
no vault of its own: its skills resolve the target vault at runtime from
[`references/vaults.md`](claude/plugins/vam-knowledge/references/vaults.md) and
address it as `mcp__<vault>__*`.

That split is what makes multiple vaults work. A machine installs `vam-knowledge`
once plus the connectors for the vaults it actually has; with a single vault
installed, resolution is silent and nothing ever asks which one.

Each connector reads its path from its own variable — `VAM_VAULT_<SLUG>`, set per
machine as in step 1 above and never committed.

Adding a vault is a new connector plugin and one table row — no skill changes.
Rationale, the naming contract, the read-only trick, and the secrets rule are in
[claude/README.md](claude/README.md); the step-by-step is in
[vaults.md](claude/plugins/vam-knowledge/references/vaults.md#adding-a-vault);
the variable list is in [.env.example](.env.example).

## How installation actually works

A plugin is **not** merged into `~/.claude/skills/`. It is unpacked whole,
versioned, into the plugin cache, and Claude Code discovers skills from several
roots at once — `~/.claude/skills/`, a project's `.claude/skills/`, and every
enabled plugin's `skills/`:

```
~/.claude/plugins/cache/vam-ai-units/
├── vam-knowledge/0.4.0/          # .claude-plugin/, references/, skills/
├── vam-session/0.1.1/
└── vam-vault-<slug>/0.1.1/      # .mcp.json
```

Consequences worth knowing:

**Keep skills in exactly one root.** A skill present both in `~/.claude/skills/`
and in a plugin appears **twice** — the plugin's namespaced as
`vam-knowledge:vam-kb-recall`, the personal one bare as `vam-kb-recall`. They do
not shadow each other; both are live and the model picks between two identical
descriptions. Once a skill lives in a plugin here, delete the personal copy.

**Which version wins: whichever `installed_plugins.json` points at.** Several
versions can sit in the cache side by side; superseded ones are tagged
`.orphaned_at` and ignored. Check with `claude plugin list`.

**`marketplace update` and `plugin update` are different things.** The first
refreshes the marketplace *listing*; the second upgrades the *installed* copy.
`claude plugin install` on an already-installed plugin is a no-op. So the
runtime can sit on an old version while `claude plugin details` — which reads the
marketplace source — cheerfully reports the new one. On a local-directory
marketplace those diverge silently. `claude plugin list` and the cache directory
are the truth.

**`${VAR}` is resolved at server launch, not at install.** The cached
`.mcp.json` keeps the placeholder verbatim:

```json
"args": ["@bitbonsai/mcpvault@0.16.0", "${VAM_VAULT_EXAMPLE}"]
```

Claude Code expands it each time it starts the server, per session, from the
process environment plus `env` in `~/.claude/settings.json`. So the repo stays
machine-independent and a changed path needs no reinstall — just the new value
and a restart. Caveat: an **unset** variable is passed through unexpanded and
the server may still report connected. Verify with `claude mcp list`; a literal
`${...}` in the command line is the tell.

## Developing a unit

Skills are plain markdown; there is no build step.

1. Edit the `SKILL.md` under `claude/plugins/<plugin>/skills/<skill>/`.
2. Bump the plugin `version` in **both** `claude/plugins/<plugin>/.claude-plugin/plugin.json`
   and the matching entry in `.claude-plugin/marketplace.json` — they must agree.
3. Commit and push.
4. On each machine: `claude plugin update <plugin>@vam-ai-units`.

While iterating, work against the local clone as the marketplace source so
there is no push/pull round-trip.

## Adding a new unit

A plugin can carry `skills/`, `agents/`, `commands/`, hooks, and its own
`.mcp.json`. To add one:

1. `claude/plugins/<name>/.claude-plugin/plugin.json` with `name`, `description`, `version`.
2. Content beside it in `skills/` / `agents/` / `commands/`.
3. A new entry in `.claude-plugin/marketplace.json` pointing at `./claude/plugins/<name>`.

Group by dependency, not by theme: units that need the same MCP server or
external tool belong in one plugin, so installing it either works fully or
not at all.

## Non-goals

- No secrets, credentials, or tokens. `.gitignore` blocks the usual suspects;
  anything machine-specific belongs in `~/.claude/settings.local.json` or a
  user environment variable.
- No `settings.json` or workflows — machine and company scope respectively.
- No MCP servers other than those a plugin here owns. A unit's own server
  belongs in that plugin's `.mcp.json`, parameterised with `${VAR}`; company
  servers stay in `~/.claude.json`.

## License

[MIT](LICENSE). The units are method text, not company material — reuse, fork, and
adapt them freely.
