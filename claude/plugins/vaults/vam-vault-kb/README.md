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
[Environment]::SetEnvironmentVariable('VAM_VAULT_KB', 'D:\Notes\MyVault', 'User')
claude plugin install vam-vault-kb@vam-ai-units
```

Restart Claude Code so the variable is in the environment it launches `npx` from,
then verify with `claude mcp list`:

```
plugin:vam-vault-kb:obsidian-kb: npx @bitbonsai/mcpvault@0.16.0 D:\Notes\MyVault - Connected
```

A literal `${VAM_VAULT_KB}` in that line means the variable is unset. The
server still reports **Connected** — it mounts a path that does not exist — so
the empty-looking vault that follows is a configuration fault, not an empty
vault. `vam-vault-setup` owns that diagnosis.

## After installing

Run `/vam-knowledge:vam-vault-setup` once against this vault. It writes
`_meta/vault-conventions.md` and `_meta/tag-vocabulary.md`; without them every
other `vam-knowledge` skill refuses to write.

Nothing hardcodes this connector. The skills enumerate `mcp__plugin_vam-vault-*`
at runtime and resolve which vault to use per
[`references/vaults.md`](../../vam-knowledge/references/vaults.md).
