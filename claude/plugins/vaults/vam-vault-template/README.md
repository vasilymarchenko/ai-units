# vam-vault-template

A vault connector is three files and no content: a `.mcp.json` declaring one
`mcpvault` server, a plugin manifest, and this README.

## Installed as-is

It works. The shipped server is `obsidian-example` reading `${VAM_VAULT_EXAMPLE}`,
so on a single-vault machine the whole setup is one variable:

```powershell
[Environment]::SetEnvironmentVariable('VAM_VAULT_EXAMPLE', 'D:\Notes\MyVault', 'User')
claude plugin install vam-vault-template@vam-ai-units
```

The vault is then reachable as `mcp__plugin_vam-vault-template_obsidian-example__*`,
which the skills resolve at runtime — nothing hardcodes it. Verify with
`claude mcp list`; a literal `${...}` in the command line means the variable is unset.

The cost of taking the default is that `example` is a meaningless slug. Fine for
one vault, worth renaming for two.

## Copied per vault

Copy this directory to `vam-vault-<slug>/`, then:

1. `.mcp.json` — server name `obsidian-<slug>`, path `${VAM_VAULT_<SLUG>}`.
   Add `"--read-only"` to `args` for a vault you only ever recall from; the
   write tools then do not exist, which beats instructing a skill not to use them.
2. `.claude-plugin/plugin.json` — set `name` and `description`.
3. Register the plugin in a marketplace and install it.

The resulting tool prefix is `mcp__plugin_vam-vault-<slug>_obsidian-<slug>__`.
Skills never hardcode it; they enumerate `mcp__plugin_vam-vault-*` at runtime.

**A connector for a vault you would rather not name publicly does not have to
live here.** The same three files work from any directory registered as a
marketplace, including one outside version control.
