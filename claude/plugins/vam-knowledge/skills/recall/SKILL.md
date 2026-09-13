---
name: recall
description: Read the user's own prior notes out of the Obsidian vault into the current session, before re-solving something they already solved. Use when the user asks "what do I have on ABC-1234", "did I ever do something like this", "what did I write about X", "check my kb / vault / notes", "did I already explain this", or when work starts on a ticket they may already have notes on. Strictly read-only — never writes to the vault; writing is `save`.
---

# Recall

Read prior notes back into the session. **Strictly read-only.** This skill has no
write path. If something needs saving, that is `save`.

## Preflight

Resolve the vault per `references/vaults.md`. One vault present in the session →
use it silently. No vault tools at all → say so and stop.

A vault that reads as completely empty, or whose root holds folders other than
`tickets/` and `inbox/`, is usually a misconfigured connector rather than the
truth about the user's notes. `get_vault_stats` and `list_directory('/')`
answer that in one call each. Say so, and point at `claude mcp list` — **read the
path printed there, not the word `Connected`**. A stale process environment can
keep serving the previous vault across what looks like a full restart.

## The shape you are reading

```
tickets/<TICKET>/<TICKET>.md          # hub: In my words · Items · State
tickets/<TICKET>/explain--<subject>--<lang>-<level>.md
tickets/<TICKET>/draft--<subject>.md
tickets/<TICKET>/note--<subject>.md
inbox/                                # saves that had no ticket
```

The hub's `## Items` lists every file in that folder with a one-line description,
as a path-qualified link. Reading the hub is therefore cheaper than listing the
directory and much cheaper than reading everything.

Item basenames repeat across tickets — `draft--infosec-comment` may exist under
several. So **follow the link as written**, path and all: strip the `[[ ]]` and the
`|alias`, and hand the rest to `read_note`. `wiki_link` resolves the same
path-qualified form when you would rather not parse it yourself. Resolve a bare
basename and you get whichever file the vault finds first, with the rest listed as
`alternatives` — which means you are probably reading the wrong ticket.

Frontmatter worth knowing: `topics` (open free text, the retrieval mechanism),
`ticket`, `project`, `lang` / `level` on explanations, `status` on drafts and on
hubs. There is no `tags` field — `list_all_tags` has nothing to say about this
vault, so do not spend a call on it.

## Tools

| Need | Tool |
|---|---|
| the fast answer on a known ticket | `read_note` on the hub |
| the folder when the hub is missing | `list_directory` |
| find notes by topic | `search_notes` — **set `searchFrontmatter: true`** |
| read several items at once | `read_multiple_notes` (max 10) |
| rank candidates without reading them | `get_frontmatter`, `get_notes_info` |
| one section of a long note | `get_note_outline`, then `read_note_lines` |

`search_notes` defaults to `searchFrontmatter: false`, which silently skips
`topics` — the field written at save time specifically to be searched. Always pass
it. `limit` defaults to 5 and goes to 20.

Leave `pathPrefix` unset for a topic search: the whole vault is the point, and
`inbox/` is exactly where an unfiled answer hides. Narrow with `pathPrefix` only
when the user has already named the ticket.

## Two questions, two routes

### "What do I have on ABC-1234" — a named ticket

1. Read `tickets/<TICKET>/<TICKET>.md`.
2. `## In my words` and `## State` are usually the whole answer. Lead with them.
3. Read the items from `## Items` that the user's question actually needs — not
   all of them by reflex. A question about the security comment needs the draft,
   not both explanations. When you do need several, one `read_multiple_notes`
   beats four `read_note` calls.
4. If the folder exists but the hub does not, `list_directory` and read from there.
5. If the folder does not exist at all (`Directory not found:`), say plainly that
   there is nothing on that ticket. Do not go searching for a consolation prize
   unless the user asks.

### "Did I ever do something like X" — a topic

1. `search_notes` with `searchFrontmatter: true`, over content **and** `topics`.
   `topics` is open free text written at save time with retrieval in mind, so a
   term match there is a strong signal.
2. Rank the hits. **Hubs rank above items** — `type: ticket`, or a basename equal
   to the folder name — because `## In my words` and `## State` are the fast answer
   and the items hang off them.
3. Read the best few, not everything. Two or three notes read properly beat ten
   skimmed. `get_frontmatter` is enough to drop a candidate.
4. Do not forget `inbox/`. It holds the saves that had no ticket, and nothing
   links to them from anywhere — an unscoped search is the only way they surface.
5. Summarise what the user already established, and say plainly when the notes do
   **not** cover the question. A confident summary of thin material is worse than
   "nothing on this".

Search several phrasings before concluding there is nothing. The user's words today
may differ from the words they saved with — try the abbreviation, the product name,
and the plain-language description.

## Report

Always name the files you read, as vault-relative paths, so the user can open them:

```
read → tickets/ABC-1234/ABC-1234.md, tickets/ABC-1234/explain--trusted-proxy--ua-plain.md
```

When notes exist but are stale or contradict the current code, say so rather than
presenting them as current. Notes record what was true when they were written;
`updated:` says when that was.

## Out of scope

Writing, merging, moving, tagging, or deleting anything. Gardening the vault.
Repairing a broken vault. If the user's request turns into "and save that", hand
over to `save`.
