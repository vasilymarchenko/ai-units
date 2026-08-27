# Vaults

**Which vault a skill talks to is resolved here, not hardcoded in the skill.**

An `mcpvault` process serves exactly one vault, so every vault is its own MCP
server, installed by its own connector plugin (`vam-vault-<slug>`). A machine
carries only the vaults it actually has.

**Tool names.** A plugin-provided MCP server exposes its tools as
`mcp__plugin_<plugin-name>_<server-name>__<tool>` — the plugin name is part of
the prefix. So the vault tools are, in full:

```
mcp__plugin_vam-vault-<slug>_obsidian-<slug>__read_note
└─────────── the prefix, as it appears in session ────┘ └── tool ──┘
```

Throughout the skills, `mcp__<vault>__<tool>` means: take the prefix of the
resolved vault **exactly as the session presents it** and append the tool name.
Copy it verbatim from a tool name you can actually see; never construct it from
the vault name or from this example.

---

## Which vaults are present

**There is no list here.** Enumerate at runtime: every tool whose name starts
with `mcp__plugin_vam-vault-` belongs to a vault connector, and the slug is the
segment after `vam-vault-`. A machine that has one vault shows one prefix; a
machine with none shows none, and that is the answer, not an error.

Per-vault facts — what the vault is for, whether it is the primary, anything
local — belong in that vault's own `_meta/vault-conventions.md`, not in this
file. This plugin ships no knowledge of any particular vault.

**Read-only vaults announce themselves.** A connector passing `--read-only`
simply has no write tools. Absence of `write_note` *is* the read-only marker.

## Resolution — in this order, once per session

1. **The user named a vault** ("my personal vault", "the archive") → use it.
2. **Exactly one vault is present in the session** → use it. Do not ask. This
   is the common case and it must be silent. Vault tools are recognisable by the
   `mcp__plugin_vam-vault-*` prefix.
3. **Several are present** → read each one's `_meta/vault-conventions.md` and
   use the one whose stated purpose matches the topic.
4. **Still ambiguous, and the operation writes** → ask once, then reuse the
   answer for the rest of the session. Never guess on a write.

Reuse the resolved vault for every subsequent call in the session. Re-resolve
only if the user names a different one.

## Rules

- **Never write to two vaults in one operation.** A note belongs to one vault.
- **Never write to a read-only vault.** In practice you cannot: its connector
  passes `--read-only`, so the write tools do not exist. If a write tool is
  missing, that is the answer, not an error to route around.
- **Say which vault you used** when more than one is present in the session.
  Silence is fine when there is only one.
- **The vault contract is per vault.** `_meta/vault-conventions.md` and
  `_meta/tag-vocabulary.md` are read from the resolved vault, not from another
  vault. A vault without them is a vault you do not write to (see
  `vam-kb-capture` preflight).
- **Missing contract notes usually mean a misconfigured connector, not an
  empty vault.** An unset `VAM_VAULT_<SLUG>` is passed through unexpanded and
  the server still reports connected, so it mounts a path that does not exist.
  Before telling the user their vault has no contract, check whether the vault
  looks empty entirely — `get_vault_stats` returning nothing is the signal. Say
  so and point at `claude mcp list`, where the tell is a literal `${...}` in the
  server's command line. Do not improvise a taxonomy into what looks like a
  fresh vault. `vam-vault-setup` owns that diagnosis and the repair — hand off
  to it rather than reasoning it out inline.

## Adding a vault

1. Copy `claude/plugins/vaults/vam-vault-template/` to `vam-vault-<slug>/`.
2. In `.mcp.json`: server name `obsidian-<slug>`, path `${VAM_VAULT_<SLUG>}`.
   Append `"--read-only"` to `args` for a vault you only ever recall from.
   The resulting tool prefix is `mcp__plugin_vam-vault-<slug>_obsidian-<slug>__`
   — confirm it against a live session rather than assuming.
3. In `.claude-plugin/plugin.json`: bump `name`, rewrite `description`.
4. Add the plugin to `.claude-plugin/marketplace.json`.
5. Add the variable to `.env.example`.
6. On each machine that has the vault: set `VAM_VAULT_<SLUG>`, then
   `claude plugin install vam-vault-<slug>@vam-ai-units`.
7. Run `vam-vault-setup` against the new vault. It writes
   `_meta/vault-conventions.md` and `_meta/tag-vocabulary.md`, without which
   every other skill refuses to write, and scaffolds the folders and MOCs the
   contract promises.

A connector for a vault you would rather not name publicly does not need to be
committed at all: the same three files work from `~/.claude/plugins/`.

Cost per installed vault is roughly 18 always-on tool schemas, so install only
the vaults a machine actually uses.
