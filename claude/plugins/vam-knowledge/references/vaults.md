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

This plugin ships no knowledge of any particular vault, and vaults carry no
self-description either. What a vault is for is known from its slug and from the
user, not from a note inside it.

**Read-only vaults announce themselves.** A connector passing `--read-only`
simply has no write tools. Absence of `write_note` *is* the read-only marker.

## Resolution — in this order, once per session

1. **The user named a vault** ("my personal vault", "the archive") → use it.
2. **Exactly one vault is present in the session** → use it. Do not ask. This
   is the common case and it must be silent. Vault tools are recognisable by the
   `mcp__plugin_vam-vault-*` prefix.
3. **Several are present** → go by the slug, which is the only label a vault
   has. If the slug does not settle it, treat this as ambiguous and fall to 4.
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
- **There is no vault contract to read.** The folder shape, note types and
  frontmatter schema live in the skills, in this repo — nowhere inside the vault.
  There is nothing to bootstrap, nothing to validate, and no note to keep in step
  with the skills.
- **A vault that reads as empty usually means a misconfigured connector.** An
  unset or misspelled `VAM_VAULT_<SLUG>` is passed through unexpanded and the
  server still reports connected, so it mounts a path that does not exist.
  `get_vault_stats` returning nothing is the signal. Say so and point at
  `claude mcp list` — the tell is a literal `${...}` in the server's command line,
  or simply the wrong path. **Read the path, not the word `Connected`.**
- **A stale path can survive a restart.** `settings.json` `env` is re-read every
  session; the process environment is not. A desktop app that keeps a background
  process alive hands the same old environment to a new window, so a user
  environment variable set between sessions may never arrive. If the printed path
  is the *previous* vault, that is the cause — the fix is to put the variable in
  `~/.claude/settings.json` under the exact name the connector expands.

## Adding a vault

1. Copy `claude/plugins/vaults/vam-vault-kb/` to `vam-vault-<slug>/`. A connector
   is three files and no content, so the shipped one doubles as the template.
2. In `.mcp.json`: server name `obsidian-<slug>`, path `${VAM_VAULT_<SLUG>}`.
   Append `"--read-only"` to `args` for a vault you only ever recall from.
   The resulting tool prefix is `mcp__plugin_vam-vault-<slug>_obsidian-<slug>__`
   — confirm it against a live session rather than assuming.
3. In `.claude-plugin/plugin.json`: bump `name`, rewrite `description`.
4. Add the plugin to `.claude-plugin/marketplace.json`.
5. Add the variable to `.env.example`.
6. On each machine that has the vault: set `VAM_VAULT_<SLUG>` in
   `~/.claude/settings.json`, then
   `claude plugin install vam-vault-<slug>@vam-ai-units`, then restart and verify
   the path with `claude mcp list`.
7. Nothing scaffolds the vault — `save` creates `tickets/<TICKET>/` lazily on the
   first write, and an empty vault is the expected starting state. A vault kept in
   a git repo has an undo for a bad merge, but no skill runs `git`; that is the
   user's own practice.

**A connector for a vault you would rather not name publicly does not have to
live in this repo.** The same three files work from any directory registered as
a marketplace, including one outside version control — steps 4 and 5 then point
at that marketplace instead.

Cost per installed vault is roughly 18 always-on tool schemas, so install only
the vaults a machine actually uses.
