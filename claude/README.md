# Claude Code units

Consumed as a plugin marketplace. The manifest that lists these plugins is
[`../.claude-plugin/marketplace.json`](../.claude-plugin/marketplace.json) —
it must stay at the repo root for Claude Code to discover it.

## Plugins

### `vam-knowledge`

The knowledge-base loop over an Obsidian vault.

| Skill | Direction | Role |
|---|---|---|
| `vault-setup` | diagnose → write on approval | Bootstrap and repair the vault contract the four skills below depend on. Start here on a new vault, or when something is broken. |
| `recall` | read | Pull prior notes into the session before re-solving something already solved. |
| `capture` | write | Turn what was learned into a durable, repo-grounded note. |
| `organize` | read → write on approval | Garden the vault: orphans, duplicates, off-vocabulary tags, stale claims, oversized notes. |
| `map-project` | read → write | Map an unfamiliar large codebase level by level, writing findings to the vault. |

**The vault is not wired into this plugin.** `vam-knowledge` ships skills only.
Which vault they talk to is resolved at runtime from
[`plugins/vam-knowledge/references/vaults.md`](plugins/vam-knowledge/references/vaults.md),
and the vault itself is mounted by a separate connector plugin. Pair them:

```powershell
claude plugin install vam-knowledge@vam-ai-units
claude plugin install vam-vault-kb@vam-ai-units   # or your own vam-vault-<slug>
```

**Still machine prerequisites:**

- `npx` on PATH (Node.js).
- Two contract notes in each vault, which the skills defer to rather than
  inventing their own structure: `_meta/vault-conventions.md` and
  `_meta/tag-vocabulary.md`. Without them the skills refuse to write, by design.
  **`vault-setup` writes them** — it diagnoses what is missing and
  interviews for the rest, so this is not manual setup.

### Vault connectors — `vam-vault-*`

`mcpvault` serves exactly one vault per process, so **one vault = one MCP server
= one connector plugin**. Each connector is a `.mcp.json` and a manifest, nothing
else.

This section and the ones after it use a hypothetical `vam-vault-example`
connector for a vault with slug `example`. Substitute your own slug throughout;
the connector this repo actually ships is `vam-vault-kb`.

```json
{
  "mcpServers": {
    "obsidian-example": {
      "command": "npx",
      "args": ["@bitbonsai/mcpvault@0.16.0", "${VAM_VAULT_EXAMPLE}"],
      "env": {}
    }
  }
}
```

| Connector | Server | Variable | Mode | Notes |
|---|---|---|---|---|
| `vam-vault-kb` | `obsidian-kb` | `VAM_VAULT_KB` | read-write | the shipped connector; copy it per extra vault |

Why one plugin per vault rather than one plugin declaring several servers: a
`.mcp.json` entry cannot be conditionally omitted. `${VAM_VAULT_OTHER:-}` unset
means `mcpvault` gets an empty path and fails, so every machine would carry
failed-connection noise for vaults it does not have. Per-vault plugins make
presence an **install-time** choice, which is where it belongs — and each
installed vault costs roughly 18 always-on tool schemas, so that choice is worth
making per machine.

For a vault you only ever read from, append `"--read-only"` to `args`. The write
tools then do not exist, which is a stronger guarantee than a skill instruction
not to use them.

Adding one: the step-by-step is in
[`vaults.md`](plugins/vam-knowledge/references/vaults.md#adding-a-vault).

### `vam-session`

| Skill | Role |
|---|---|
| `handoff` | Compact the conversation into a document a fresh session can act on. |

No prerequisites.

## Configuring a machine

Plugin `.mcp.json` files support `${VAR}` and `${VAR:-default}` expansion in
`command`, `args`, `env`, `url`, and `headers`, plus `${CLAUDE_PLUGIN_ROOT}` for
paths inside the plugin. That is the seam: the *shape* of a server is versioned
in this repo, the machine-local *values* are not.

For each vault this machine has, set its variable in `~/.claude/settings.json`:

```json
{
  "env": {
    "VAM_VAULT_EXAMPLE": "D:\\Notes\\ExampleVault"
  }
}
```

or as a user environment variable, which also makes it visible to tools outside
Claude Code:

```powershell
[Environment]::SetEnvironmentVariable('VAM_VAULT_EXAMPLE', 'D:\Notes\ExampleVault', 'User')
```

Then install that vault's connector. See [`../.env.example`](../.env.example)
for the current variable list.

Deliberately **no defaults**. A default would be a path that exists on exactly
one machine.

**Verify after setting it — an unset variable does not fail loudly.** Tested:
with `VAM_VAULT_EXAMPLE` absent from the environment, Claude Code passes the
placeholder through unexpanded and `mcpvault` still reports success:

```
$ claude mcp list
plugin:vam-vault-example:obsidian-example: npx @bitbonsai/mcpvault@0.16.0 ${VAM_VAULT_EXAMPLE} - Connected
```

A literal `${...}` in that line is the tell. There is no "missing environment
variable" diagnostic for plugin-provided servers the way there is for ones
declared in `~/.claude.json`.

The real guard is the vault contract, not the connector: the skills read
`_meta/vault-conventions.md` before writing, and a bogus vault does not have it,
so a write is refused. `vaults.md` tells them to suspect an unset variable when
that file is missing. Belt and braces — but check `claude mcp list` after
setting up a machine rather than trusting a green tick.

Correct output looks like:

```
$ claude mcp list
plugin:vam-vault-example:obsidian-example: npx @bitbonsai/mcpvault@0.16.0 D:\Notes\ExampleVault - Connected
```

### Server naming is a contract

A plugin-provided MCP server exposes its tools as
`mcp__plugin_<plugin-name>_<server-name>__<tool>` — **the plugin name is part of
the prefix**, not just the server name. Confirmed by invoking a tool in a live
session, not inferred:

```
mcp__plugin_vam-vault-example_obsidian-example__get_vault_stats
```

`claude plugin details` and `claude mcp list` show the friendlier
`plugin:vam-vault-example:obsidian-example` form; that is display only.

The convention, so the prefix is derivable:

```
vault slug   example
connector    vam-vault-example
server       obsidian-example
tool prefix  mcp__plugin_vam-vault-example_obsidian-example__
variable     VAM_VAULT_EXAMPLE
```

Skills never write that prefix literally — they use `mcp__<vault>__*` and
resolve the real prefix at runtime per
[`vaults.md`](plugins/vam-knowledge/references/vaults.md), by enumerating the
`mcp__plugin_vam-vault-*` tools actually present in the session. That
indirection is load-bearing: renaming either the connector **or** the server
changes the tool prefix, and nothing here has to be edited to follow it. It also
keeps the names of the vaults on a given machine out of this repo.

### Secrets

Never inline a credential in `.mcp.json` — it is committed. Reference it as
`${SOME_TOKEN}` and set the value per machine. Compare the official `github`
plugin: `"Authorization": "Bearer ${GITHUB_PERSONAL_ACCESS_TOKEN}"`.

## Not distributed here

Deliberately out of scope, because they are machine or company scope:

- `~/.claude/settings.json` — model, effort, theme, `enabledPlugins`, auto-mode environment.
- MCP servers not owned by a plugin here — work and third-party servers live
  in `~/.claude.json` alongside machine state and carry credentials. Register
  those per machine.
- `~/.claude/workflows/` — where these are organisation-wide, they come from
  that organisation's own marketplace.
