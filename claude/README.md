# Claude Code units

Consumed as a plugin marketplace. The manifest that lists these plugins is
[`../.claude-plugin/marketplace.json`](../.claude-plugin/marketplace.json) —
it must stay at the repo root for Claude Code to discover it.

## Plugins

### `vam-knowledge`

A ticket-bound knowledge base over an Obsidian vault.

| Skill | Direction | Role |
|---|---|---|
| `save` | read → write | Write what the session produced — an explanation, a message draft, a finding — into `tickets/<TICKET>/` as a standalone document. A follow-up merges into the existing note. |
| `recall` | read | Pull prior notes back into the session, by ticket or by topic, before re-solving something already solved. |

Everything is filed under a ticket, with a hub note per ticket carrying the problem
in the user's own words. Retrieval is by ticket, by open free-text `topics`, and by
full-text search. There is deliberately **no** long-term-knowledge tree, no closed
tag vocabulary, no MOCs and no gardening skill — all four were tried and cut, and
the reasons are in the skills themselves.

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
- Some way to undo a bad write — the vault in a git repo, or Obsidian's file
  recovery. A save *merges* into the note it already wrote, so it can rewrite a
  file you edited by hand. The skill does not manage that for you: it never runs
  `git`, and versioning the vault is the user's own practice.

There is no vault contract to bootstrap, and nothing to scaffold. The folder
shape, note types and frontmatter schema live in the skills, in this repo — never
in the vault. An earlier design kept a copy in `_meta/vault-conventions.md`, and
the executed rules drifted from the documented ones. A vault holds notes; what
produced them is documented here.

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

The second guard is thin, deliberately. `save` checks the vault root once per
session and stops when it finds folders that are not `tickets/` or `inbox/` — that
catches the dangerous case, a full but *wrong* vault. It cannot catch an empty one,
because an empty vault is also what a correct new vault looks like. So check
`claude mcp list` after setting up a machine, and read the path rather than
trusting the green tick.

The same applies to a path that *changed*. `settings.json` `env` is re-read every
session, but the process environment is not — a desktop app that keeps a
background process alive hands the same stale environment block to a new window,
so a user environment variable can survive what looks like a full restart. Keep
vault paths in `settings.json` for that reason.

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
