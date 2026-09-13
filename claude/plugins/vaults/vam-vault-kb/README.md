# vam-vault-kb

A vault connector is three files and no content: a `.mcp.json` declaring one
`mcpvault` server, a plugin manifest, and this README. The vault lives on disk;
this only mounts it.

This is the connector the repo ships. It also serves as the template for a
second vault — see *Adding a vault* in
[`references/vaults.md`](../../vam-knowledge/references/vaults.md).

It names no particular vault. The slug `kb` just means "the primary knowledge
vault on this machine"; which directory that is depends entirely on the value of
`VAM_VAULT_KB`, which is machine-local and never committed.

| | |
|---|---|
| Server | `obsidian-kb` |
| Variable | `VAM_VAULT_KB` |
| Mode | read-write |
| Tool prefix | `mcp__plugin_vam-vault-kb_obsidian-kb__` |

## Install

```powershell
claude plugin install vam-vault-kb@vam-ai-units
```

Set `VAM_VAULT_KB` in `~/.claude/settings.json` under `env` — a user environment
variable works too, but only `settings.json` is re-read at every session start:

```json
{ "env": { "VAM_VAULT_KB": "D:\\Notes\\MyVault" } }
```

Restart Claude Code, then verify with `claude mcp list`:

```
plugin:vam-vault-kb:obsidian-kb: npx @bitbonsai/mcpvault@0.16.0 D:\Notes\MyVault - Connected
```

**Read the path, not the word `Connected`.** Two ways it lies:

- a literal `${VAM_VAULT_KB}` means the variable is unset, or spelled differently
  from the name this connector expands. The server still reports **Connected** —
  it mounts a path that does not exist — so the empty-looking vault that follows
  is a configuration fault, not an empty vault;
- the **previous** path, after you changed it, means the value came from a stale
  process environment. A desktop app that keeps a background process alive can
  carry the old environment across what looks like a full restart. Putting the
  value in `settings.json` makes that impossible.

## After installing

Nothing to set up. There is no vault contract, and `vam-knowledge:save` creates
the folder tree lazily on the first write.

Consider putting the vault in a git repo anyway. A save *merges* into the note it
already wrote, so it can rewrite a file edited by hand in Obsidian, and a version
history is the undo. The skills never run `git` themselves — that stays yours.

Nothing hardcodes this connector. The skills enumerate `mcp__plugin_vam-vault-*`
at runtime and resolve which vault to use per
[`references/vaults.md`](../../vam-knowledge/references/vaults.md).
