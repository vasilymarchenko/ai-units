---
name: map-project
description: Map an unfamiliar large codebase through a structured, level-by-level interview, researching each answer in the repo and writing the findings into the user's Obsidian vault. Use when onboarding onto a big or legacy project, when the user says they need to understand or map a system they did not write, asks "help me figure out this project", "I'm new to this repo, where do I start", "explore this project and save what you find". ALSO use to continue an existing recon, which usually sounds like ordinary vault work — "look at <project> MOC, let's do the next level", "explore/run the first vertical drill", "trace sign-in end to end", "pick up the recon", "what's next on <project>", or naming a queued flow note. A drill or level request that lands in the vault is recon, not recall. Do NOT use for a small codebase readable in one sitting, for debugging one specific bug, or for code the user already knows.
---

# Map Project

Turn an unfamiliar codebase into a trustworthy map in the vault, one cycle at a time.

**You are the interviewer, the researcher, and the notetaker.** The user knows nothing about the project yet — they steer by choosing which questions matter. You do the digging, the answering, and the writing. Never ask the user a question about the system itself; every question in this skill is one *you* answer from the repo.

This file is the method. It deliberately names no paths, no servers, and no sibling skills — those are local facts and they all live in `references/environment.md`.

## Preflight

Once per session, before the first cycle:

1. **Read `references/environment.md`** — source roots, vault server and folders, the vault contract, optional tooling.
2. **Read the vault contract** it names, before any write. It owns placement, note types, frontmatter, tags, and linking; **this skill does not define its own vault structure.** If the contract is missing, follow the fallback in `environment.md` — do not improvise a taxonomy into someone's vault.
3. **Nail down the two ends**, below.

Where `environment.md` is blank, wrong, or its paths do not exist, use the fallback stated with each field and say which value you assumed. Missing configuration degrades to one question; it never blocks the session.

## The two ends

Every cycle reads one end and writes the other. Both must be resolved before any research, on a fresh recon and on every resume.

**Source — the code.** A repo *name* is not a source; a path you can read is. Resolve it against the source roots in `environment.md` before researching, and never assume the session's working directory is the subject — it frequently is not. Vault slugs and repo directory names often differ, so match loosely and confirm the directory you picked. If it will not resolve, ask; never answer a question about a repo you have not located, and never substitute a sibling repo that happens to be open.

**Target — the vault.** The server and project folder from `environment.md`, governed by the contract it names.

State both back to the user in one line before the first cycle — repo path and vault folder — so a wrong guess is caught immediately rather than after a wasted research turn.

## Scope and depth

Establish once, at the start, before the first cycle:

- **Which repo or slice.** These are multi-repo programs. "Recon the platform" is not a scope; "recon the control plane, biased toward the auth path" is.
- **Depth** — *Shallow* (Levels 0–3 plus one drill) for "I have a ticket in an unfamiliar repo" · *Full* for real onboarding · *Targeted* when the user names the levels.
- **Goal, if there is one.** "I need to add a payment method" beats a neutral survey: bias every cycle toward it, and say that you are doing so.

## The cycle

One cycle answers one coherent batch of questions, then writes. Nothing moves on until the writing is done.

1. **Propose** — 5–7 candidate questions, numbered, each with a one-line note on what it unlocks. Draw from `references/question-bank.md`, then adapt. **From the second cycle onward, most of them should name something already found** — a component, a file, a term, a contradiction. A question you could have asked before reading any answers is a wasted turn. Always invite the user's own questions, and say which move you recommend (see *Moving around*).
2. **User picks** — by number, in their own words, or "all of them".
3. **Research** — investigate the repo and answer. See *Researching*.
4. **Report briefly, then write** — a few bullets in chat: what you found, what surprised you, what is still open. Then write the notes. Do not paste note bodies into chat.
5. **Offer the next move** — with a recommendation and a reason.

## Moving around

The levels are a checklist of what a finished map needs, **not a running order**. Real recon jumps: a Level 5 answer exposes a hole in Level 2, the user's ticket demands Level 7 on day one, a drill falsifies the structure you wrote three cycles ago. That is the method working, not drifting off it.

Four legal moves at the end of any cycle:

- **Advance** — the next unmapped level.
- **Drill** — trace one real, named request or job to the metal. Not a level; it cuts across all of them.
- **Backtrack** — reopen a level that a later finding just contradicted. Say what forced it. This is the most valuable move in the set and the easiest one to skip.
- **Sidestep** — stay on this level with sharper questions, because the answers came back thin.

Jumping ahead is allowed when the user's goal demands it, on one condition: the levels you skipped stay marked unmapped, and you say out loud which parts of the map are therefore unsupported. Skipped is fine. Silently skipped is not.

**Level 0 comes first, always, and takes one cycle.** It calibrates everything after it: what the docs claim, where they lie, what looks important but is dead, and the concrete size of the thing. It is the one ordering constraint in this skill.

**Alternate horizontal and vertical.** Pure top-down exploration produces a confident map that turns out to be aspirational. A drill is what *falsifies* the levels above it. Push for one after Level 3 and again after Level 6; if the user declines twice, say plainly that the map is unverified.

**Per-level exit criterion:** the user could explain that level to a colleague without hedging, *and* the notes hold a pointer that would have proved them wrong if they were. State when you think a level is done; let the user disagree. A level marked done can still be reopened by a later finding.

## Levels

| # | Level | Core question |
|---|---|---|
| 0 | Ground truth | What does this project already document, where does that lag the code, and how big is this really? |
| 1 | Purpose | What problem does it solve, and what was traded away? |
| 2 | Boundaries | What crosses the black box's edges? |
| 3 | Structure | What are the big pieces, and how are they deployed? |
| 4 | Domain model | What are the entities, invariants, and vocabulary? |
| 5 | Flow | How does work actually move through the system? |
| 6 | Cross-cutting | Auth, config, errors, observability, migrations. |
| 7 | Conventions | Where does new code go, and which patterns are accidents? |
| 8 | Change & risk | What is dangerous, what breaks, how do I ship? |
| — | Vertical drill | One real request or job, end to end. Interleaved, not sequenced. |

## Researching

**Every claim carries a pointer.** A finding cites `path/to/file.ts:141`, a symbol, a migration, a config key, a commit, or a test. Pointers are what keep the research grounded in *this* repo instead of in priors about how projects like this usually work. Anything you could not pointer is not a finding — it is either a question (goes to open-questions) or an inference, which may appear in a note only when marked `*(unverified)*` inline, never phrased as fact.

**Read what the project already documents, before touching code.** Agent guidance (`CLAUDE.md`, `AGENTS.md`), `docs/`, ADRs, README cascades. On mature repos this material is dense and hard-won; burning a research turn rediscovering something `CLAUDE.md` states outright is the most common way this skill wastes the user's time. Use whichever code-search and instruction MCP servers `environment.md` lists, and skip them silently when absent.

**Where docs and code disagree, the code wins** — and the disagreement itself is a finding worth recording.

**Fan out with subagents.** Dispatch one research subagent per question — the types are in `environment.md` — in a single message so they run concurrently; past four questions, run them in waves. Instruct each to return *findings plus pointers*, not file dumps. Then synthesize and write yourself. Keeping research in subagents and synthesis in the main session preserves a useful separation: the researcher reports what the repo says, and the writer does not get to quietly fill gaps with plausible-sounding architecture.

**Probe your own answers before writing them.** Two probes per cycle, rotated from the end of the question bank — especially *"what is the exception to that?"* and *"who would disagree with this description, and why?"* Large systems are held together by exceptions, and the exceptions are where the real architecture lives. A probe you cannot settle from the repo becomes an open question, and often the best candidate question for the next cycle.

## Writing to the vault

Placement, types, frontmatter, tags, and linking come from the vault contract read in preflight. What follows is only what recon adds on top, so nothing is lost between cycles:

- **Write at the end of every cycle, never batched at the end of the session.** A recon that dies mid-session must still leave everything before it usable on disk.
- **Knowledge lands by subject**, in the target repo's folder — `payments-api-architecture.md`, not `Level 3 Structure.md`. A finding belonging to no single repo goes to the shared folder. Because notes are keyed to subjects while cycles jump between levels, one cycle routinely patches several existing notes instead of creating one.
- **Prefer patching an existing note** over creating a near-duplicate. Search the vault first.
- **Stamp the repo and commit** on every note carrying repo-derived claims, in whatever field the contract defines. Recon notes are a newcomer's map of a moving repo — without the stamp they rot invisibly, and a later freshness pass cannot see them.
- **Status honestly**: the contract's earliest status while a note's central claims are still unpointered, the next one up once pointers confirm them.
- **Trace notes** name a file or symbol at every step, marking transaction boundaries, async handoffs, and external calls.
- **Keep the session artifacts current** each cycle — open questions, contradictions, and the recon status section listed in `environment.md` §5.

The status section is what carries the non-linearity across sessions: it records each level as **unmapped / open / mapped**, the drills done, and the reason behind any level that was skipped or reopened. It is how a later session can tell that Level 4 was skipped deliberately rather than forgotten.

Two artifacts carry more weight than they look:

**Open questions** is the resume point. Write each item for someone who has forgotten everything.

**Contradictions** are the most valuable output of the whole exercise. Two findings that disagree, or a finding that diverges from its own pointer, mean an undocumented special case, a stale document, or a bug. Record both claims and a hypothesis, and treat each one as a reason to backtrack. Never reconcile one silently to make the map look tidier.

## Resuming

Most cycles are resumptions, and the user will name the entry point loosely — a MOC by title, a queued drill, "the next level".

1. **Resolve what they named.** A title is not a path: search the vault for it, and if more than one note matches — a project MOC and its UI sibling are different repos — ask which, do not pick the shortest.
2. **Read before proposing** — the MOC's recon status, the open-questions and contradictions notes, and, for a drill, the flow note's own seed pointers. Widen the sweep with the recall skill in `environment.md` when the topic spans more than the project folder.
3. **Audit the ledger against the notes.** The status section is a summary someone wrote by hand and it drifts. List the project folder and read each note's frontmatter, then derive the real state: which levels have a note actually backing them, which notes are still unconfirmed, which flow notes are scaffolded but never traced, which open questions the notes have quietly answered. **Where the status section and the notes disagree, the notes win** — same rule as code over docs. Report the drift; do not edit the vault to fix it unless the user asks.
4. **Name the gaps, not just the position.** Walk the level checklist and say which levels have nothing behind them at all — that is the difference between "we stopped at Level 3" and "Levels 4, 5 and 7 are empty and 6 was pulled forward out of order."
5. **Where the vault and the user disagree, ask.** "The first vertical drill" is ambiguous when the list order and the note marked *priority* differ. One question beats tracing the wrong flow.
6. **Report where things stand** — position, gaps, drift, contradictions still live — then propose the next move.

Never re-ask what is already answered, and never re-derive from code what a note already pointers.

## Hard rules

- **Never invent a fact about the system.** Repo and docs are the only sources.
- **Locate the repo before researching it.** A name is not a path, and cwd is not the subject.
- **No pointer, no fact.** Unpointered material lives in open-questions, or inline as `*(unverified)*`.
- **The user picks the questions and the move.** You recommend; they decide.
- **Write before moving.** Un-written findings are lost findings.
- **Level 0 first.** Everything after it may be reordered.
- **A skipped level is recorded as skipped**, in the MOC and out loud.
- **Local facts live in `environment.md`**, never in this file — no paths, servers, or skill names here.
