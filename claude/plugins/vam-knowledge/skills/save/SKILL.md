---
name: save
description: Write something this session produced into the user's Obsidian vault as a ticket-bound document — an explanation, a draft of a message, or a finding. Use when the user says "save this", "save this explanation", "save this in plain Ukrainian for non-specialists", "save a draft of the security comment", "update the draft", "add this to my kb", "note this down", or asks for something to be written up and kept. Also use for follow-ups that revise something already saved — those merge rather than append. Do NOT use for reading prior notes (that is `recall`), for writing files into the working repository, for repository documentation, or for the agent's own memory directory.
---

# Save

Write conversation output into the vault as a document bound to a ticket.

Writing the file is the smallest part of this skill. The work is picking the right
content, finding the right target, and reshaping chat prose into a document that
stands on its own.

## The model

Everything is ticket-bound. There is no long-term-knowledge tree, no closed tag
vocabulary, no promotion pipeline. Retrieval later is by `topics`, by the ticket,
and by full-text search. Do not invent a taxonomy beyond what is written here.

```
tickets/
  <TICKET>/
    <TICKET>.md                                  # hub
    draft--infosec-comment.md
    explain--js-challenge--ua-plain.md
    note--client-ip-behind-proxy.md
inbox/                                           # saves with no ticket context
```

A ticket folder is created **lazily**, on the first save for that ticket. Never
create one ahead of time; empty folders are litter. `write_note` creates missing
parent folders itself, so a first save is one call, not two.

**Two folders, and nothing else.** The vault holds notes, not documentation about
itself: the rules live in this plugin, in git, and a copy inside the vault drifts
from the copy that is executed. An earlier design kept one in `_meta/`, and it was
stale within a day. If the user wants a signpost note, that is their file to write
and their file to maintain — do not offer to generate one.

### Note types

`ticket` · `draft` · `explanation` · `note`. Closed set.

### Filenames

```
<TICKET>.md                                      # ticket hub: the key alone
explain--<subject>--<lang>-<level>.md
draft--<subject>.md
note--<subject>.md
```

`<subject>` is a kebab-case slug of two to four words taken from the content.
`<lang>` is `en` or `ua`. `<level>` is `plain` or `expert`.

The type prefix makes a folder listing readable without opening anything.

**Basenames are unique inside a ticket folder, not across the vault.**
`draft--infosec-comment.md` is a natural name under any number of tickets. So
**every link to an item is path-qualified**, and only the hub — named for the
ticket key — may be linked bare:

```markdown
[[tickets/ABC-1234/draft--infosec-comment|draft--infosec-comment]]   # to an item
[[ABC-1234]]                                                          # to a hub
```

A bare `[[draft--infosec-comment]]` resolves to whichever file the vault finds
first. Never write one.

`inbox/` is the exception that needs care: nothing there is namespaced by a
ticket folder, and nothing links to it from a hub. **An inbox slug must be
meaningful and unique across the whole vault on its own** — say
`note--login-proxy-challenge-cookie`, not `note--the-cookie`. Check it with
`search_notes` before writing, and lengthen it if it collides.

### Frontmatter

All of it is generated. The user never writes frontmatter.

```yaml
type: explanation            # ticket | draft | explanation | note
ticket: ABC-1234             # "" for inbox/
project: [some-repo-name]
topics: [proxy, rate-limiting, bot-protection]
lang: ua                     # explanation only
level: plain                 # explanation only
channel: jira-comment        # draft only: jira-comment | slack | email | pr-description
audience: infosec            # draft only, when known
status: drafting             # draft only: drafting | sent
sent: 2026-09-11             # draft only, when status is sent
created: 2026-09-13
updated: 2026-09-13
source: [session 485c8600]  # a list: a merged note carries every session that shaped it
```

Pass this as the `frontmatter` **object** argument of `write_note`, and let the
server serialise it — lists come back as YAML block sequences, which is fine.
`content` is then the body alone, starting with a blank line so the title is not
glued to the closing `---`. Never hand-write a `---` block into `content`: with
both, the note ends up with two.

- `project` is a **list**. A ticket routinely crosses repositories. Derive it from
  the git repository directory name of the cwd, verbatim — no mapping table, no
  registry note. Add further repositories when the session clearly involved them.
- `topics` is **open free text**, written from the content. There is no vocabulary
  to maintain and nothing to garden. This is the retrieval mechanism, so be
  generous, and use the words the user would actually search with months later,
  including product names, error strings and abbreviations they used in chat.
- `source` is a **list** of session ids, short form — the first segment of
  `CLAUDE_CODE_SESSION_ID`, which Claude Code sets in the environment:

  ```bash
  echo "session ${CLAUDE_CODE_SESSION_ID%%-*}"          # bash
  ```
  ```powershell
  "session " + $env:CLAUDE_CODE_SESSION_ID.Split('-')[0]  # powershell
  ```

  Read it once per session and reuse it. A list, because a merge appends the
  current session and keeps the earlier ones — that is the trail back to the
  conversation a paragraph came from. If the variable is empty, omit the field
  rather than inventing a value.

## Tools

Everything that touches the vault goes through vault tools. Resolve the tool
prefix per `references/vaults.md` and copy it verbatim from a tool you can see in
the session.

**Nothing in this skill acts on the vault as a directory.** No `mkdir`, no `git`,
no file writes behind the server's back — the vault's history is the user's
business, not this skill's, and the skill never needs its filesystem path. The
shell is used only to read the *working repository* for metadata: the branch and
repository name for §2 and `project`, and `CLAUDE_CODE_SESSION_ID` for `source`.

| Need | Tool |
|---|---|
| does this ticket folder exist, and what is in it | `list_directory` |
| identity match without reading bodies | `get_frontmatter`, `get_notes_info` |
| check an `inbox/` slug is not already taken | `search_notes` |
| read the existing note before a merge | `read_note` |
| create or rewrite a note | `write_note` (`frontmatter` object + body `content`) |
| append one line to the hub's `## Items` | `patch_note` |
| bump `updated:` on the hub | `update_frontmatter` (`merge: true`) |

`list_directory` on a ticket with no folder yet answers `Directory not found:`.
That is the "new ticket" signal, not an error to report.

`patch_note` fails when `oldString` matches more than once. Anchor on a whole
line that is unique — the last existing `## Items` line, or the `## Items`
heading itself when the list is empty. Leave `replaceAll` false.

## Preflight

1. **Resolve the vault** per `references/vaults.md`. One vault present in the
   session → use it silently.
2. **No vault tools in the session** → say so and stop. The connector plugin is
   not installed or not enabled, and this skill cannot route around that.
3. **Confirm it is the right vault, once per session, before the first write.**
   `list_directory('/')` costs one call. A root holding `tickets/`, `inbox/`, or
   nothing at all is the expected shape; loose files the user added themselves are
   fine. **Other folders at the root mean you are probably mounted on a different
   vault.** Stop and say so; the fix is `claude mcp list`, reading the *path*
   rather than the word `Connected`, because a stale process environment can
   survive a restart.

   This matters more than it looks. The usual failure is not an empty vault — it
   is a full, wrong one, which no content check can catch after the fact.

## 1. Pick the content

Default to the last substantive thing produced on that topic — usually your own
previous message. The user should not have to explain what to save in the normal
case.

When several things could be meant, **state the pick in the report line instead of
asking**:

```
saved: the trusted-proxy explanation from my last message
```

If the pick was wrong the user says so, and the next save merges over it. Ask a
question only when there is genuinely no reasonable pick.

## 2. Detect the ticket

In order:

1. The user named it in the request.
2. It is already established in the conversation. This is the common case —
   sessions often open with "explain ticket XYZ".
3. The git branch of the cwd: `feature/ABC-1234-something` → `ABC-1234`.
4. Ask. Last resort, only when there is no signal at all.

No ticket → write to `inbox/` with `ticket: ""`. Do not invent one.

## 3. Resolve identity — create or merge

`list_directory` the ticket folder first. Then:

| Type | Identity key |
|---|---|
| `explanation` | (subject, lang, level) |
| `draft` | (subject); when several drafts exist, match the user's words against slug, `channel` and `audience` |
| `note` | (subject) |

Read `channel` and `audience` with `get_frontmatter`, not by opening the drafts.

Same subject but a different lang or a different level is a **different document**.
Do not merge a plain Ukrainian explanation into an English expert one.

If the ticket folder holds exactly one draft, "update the draft" means that one.

A near-miss slug for the same subject is a merge, not a second file —
`explain--js-challenge--ua-plain` and `explain--the-js-challenge--ua-plain` must
never both exist.

Exists → **merge** (§6). Does not exist → create.

## 4. Reshape into a standalone document

This is the real work and must not be skipped.

A chat explanation says "this table", "as I said above", "so, back to your
question". A saved note has none of that. Give it a title, resolve every reference
into the actual content it points at, and drop the conversational scaffolding.
Someone opening the file in six months has no conversation to fall back on.

Shaping modifiers in the request are **not metadata — they change the output**:

| User says | Effect |
|---|---|
| "save this explanation" | language and level as they were in chat |
| "...in plain Ukrainian for non-specialists" | `lang: ua`, `level: plain`, and the text is **rewritten** for that audience |
| "save a draft of the security comment" | `type: draft`; `channel` and `audience` inferred |

Note-body language is driven by the request, independent of the chat language. A
global instruction to answer in English governs the chat, not the note body.

`level: plain` means a non-specialist reads it without help: short sentences, no
unexplained jargon, the conclusion first. `level: expert` may assume the domain.

## 5. Write the note and the hub

Order matters, because the hub is what an item's line lives in:

```
hub missing?  → write_note(hub)            # with an empty ## Items
write_note(item)                            # frontmatter object + body
patch_note(hub)                             # one line onto ## Items
update_frontmatter(hub, {updated: today})   # plus topics, if they grew
```

The item's body ends with:

```markdown
## Related
- [[ABC-1234]]
```

An **inbox** note has no ticket and no hub, so it has no `## Related` section and
nothing to patch. Its slug carries the whole burden of being findable — see
*Filenames*. Moving an inbox note under a ticket later is a separate skill's job,
not this one's.

### The hub

Created with the first item of a ticket, never before it:

```markdown
# ABC-1234 — <short title>

## In my words
<the problem as the user would describe it, in English>

## Items
- [[tickets/ABC-1234/explain--js-challenge--ua-plain|explain--js-challenge--ua-plain]] — how the two filter rules differ
- [[tickets/ABC-1234/draft--infosec-comment|draft--infosec-comment]] — *sent 2026-09-11*

## State
<where the ticket actually stands>
```

Hub frontmatter is `type: ticket`, the ticket key, `project`, `topics`,
`status: in-progress` (`in-progress | blocked | closed`), `created`, `updated`.

**`## In my words` is the most important section in the whole model.** It is not
the Jira title and not a restatement of it. It is the problem stated the way the
user would describe it months later, because that is the string they will search
with. Write it on hub creation, from the conversation, in English.

**Links go both ways, deliberately.** Down, via `## Items`, so the folder is
readable in a terminal, and so an agent sees the contents without running a
query. Up, via `[[<TICKET>]]` under `## Related`, which gives Obsidian's
backlinks pane for free. The redundancy is intentional and costs the user nothing.

### Keeping the hub true

The hub is a live index, not a creation-time artefact.

- **New item** → `patch_note` one line onto `## Items`, with a short description of
  what the file is for. Never rewrite the hub wholesale.
- **The item's line is now wrong** → `patch_note` that one line. The usual case is
  a draft that was sent: the note gets `status: sent` and `sent: <date>`, and its
  hub line gains *sent \<date\>*. A merge that changed what a file is for gets a
  corrected description the same way.
- **Every save**, item or hub, new or merged → `update_frontmatter` on the hub with
  `updated: <today>`, plus any `topics` the ticket has genuinely gained. `merge:
  true` replaces the fields you pass and leaves the rest alone, so pass the whole
  new `topics` list, not the additions.

A hub whose `updated:` has not moved in a month is a signal to the user. Do not
make it lie.

## 6. Merge rules

Merge is **not** append and **not** overwrite. It is read → integrate → write the
whole file back.

```
read_note(target)
  → hold the old text and the new material together
  → compose one combined version
write_note(target)             # full frontmatter object + full body
update_frontmatter(hub)        # updated:, and topics if they grew
patch_note(hub)                # only for a new item, or a line that is now wrong
```

- **Deeper** — new detail on a point that already exists → expand that section in
  place. Never add a second section about the same thing.
- **Wider** — a genuinely new aspect → a new section, placed in a sensible order,
  not merely appended at the end.
- **Contradiction** — the new material shows the old text was wrong → correct it,
  **and say so in the chat**: "corrected: rule 2 does not need the cookie from
  rule 1". Never silently drop a claim that was written earlier.
- `updated:` is bumped, `source:` gains the current session id (append, never
  replace), `topics:` gains any new terms. `created:` never changes.

Drafts merge the same way, but the new input is usually an edit instruction —
"add a paragraph on the proxy header", "make it shorter" — rather than new
material. Apply the instruction to the saved text, not to the chat version.

**The risk this design accepts:** merge rewrites a whole file, and the user may
have edited it by hand in Obsidian. Silent loss is the failure mode. Two rules
contain it, and neither is optional:

- **Read before you write, every time, and account for everything you found.**
  Content you did not expect is the user's, not noise — carry it into the new
  version. Drop something only to fix a contradiction, and say so in chat.
- **Report what changed** (§7), so a bad merge is visible in the same breath.

Recovering a bad merge is the vault's own version history — Obsidian's file
recovery, or git if the user keeps the vault in a repo. Neither is this skill's
job: it does not stage, commit, or mention committing.

## 7. Report

One line back to the user, no preamble. Vault-relative path in both cases:

```
created → tickets/ABC-1234/draft--infosec-comment.md
merged  → tickets/ABC-1234/explain--trusted-proxy--ua-plain.md (expanded §2, +1 section)
```

Say what changed, not only that something changed. Add the correction line from §6
when the merge overruled something, and the `saved:` line from §1 when the pick was
not obvious.

## Out of scope

Reading notes back — that is `recall`. Moving anything out of `inbox/`, renaming,
deleting, gardening, migrating or repairing a vault. Committing the vault, or any
other git operation on it. Dataview or Bases query blocks.
