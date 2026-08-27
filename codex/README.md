# Codex units

Reserved. Nothing here yet.

Codex and its relatives read `AGENTS.md` plus a skills directory rather than a
plugin marketplace, so this folder gets its own layout when the first unit
lands — not a copy of `../claude/`.

When that happens, the thing worth avoiding is two drifting copies of the same
prose. The likely shape is a harness-neutral `shared/` folder holding the method
text, with thin per-harness wrappers in `claude/` and `codex/` supplying the
frontmatter and local facts each one needs. Decide that when there is a second
consumer to design against, not before.
