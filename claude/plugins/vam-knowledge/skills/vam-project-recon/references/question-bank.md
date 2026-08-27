# Question Bank

Candidate questions per level. Raw material, not a script — adapt to the project and to what earlier cycles revealed.

**Every question is answered by you, from the repo, not by the user.** Cite the files, symbols, migrations, config keys, or commits each finding rests on. If you are inferring rather than reading, say so explicitly and file it as unverified.

Levels are a checklist, not an order — cycles jump between them. See *Moving around* in `SKILL.md`.

Contents: [L0 Ground truth](#level-0--ground-truth) · [L1 Purpose](#level-1--purpose) · [L2 Boundaries](#level-2--boundaries) · [L3 Structure](#level-3--structure) · [Vertical drill](#vertical-drill) · [L4 Domain model](#level-4--domain-model) · [L5 Flow](#level-5--flow) · [L6 Cross-cutting](#level-6--cross-cutting) · [L7 Conventions](#level-7--conventions) · [L8 Change and risk](#level-8--change-and-risk) · [Repeatable probes](#repeatable-probes) · [Backtrack questions](#backtrack-questions)

---

## Level 0 — Ground truth

Cheap, fast, and it recalibrates everything after it. Read the project's own guidance before any code.

- What does this project already document about itself — `CLAUDE.md`, `AGENTS.md`, `docs/`, ADRs, READMEs? Summarize what it claims. — never spend a research turn rediscovering a documented fact.
- Where does the documentation contradict or lag the code? — tells you which sources to distrust for the rest of the session. The code wins; the divergence is a finding.
- How large is this, concretely — files, services, languages, age of the oldest and newest code, commit cadence? — sets expectations for how much recon is even possible.
- What is here that looks important but is dead, deprecated, or vestigial? — prevents hours spent mapping a graveyard.
- What do newcomers get wrong about this system, judging by the warnings the repo bothers to write down? — a `docs/` page full of "rules that fail silently" is a map of the traps.
- If only three things could be said before touching anything, what are they? — highest-yield question in the bank. Worth re-asking at every level.

## Level 1 — Purpose

- What problem does this exist to solve, and for whom? — orients every later "why is it like this".
- What happens to the business if it is down for an hour, a day, a week? — reveals the true criticality ranking of components.
- Which two or three qualities is it optimized for — latency, correctness, cost, auditability, flexibility? What was deliberately sacrificed? — explains design choices that otherwise look like mistakes.
- What is explicitly out of scope, even though people keep asking for it? — marks the boundaries not to cross.
- What is the value path — the flow that matters most if it breaks? — picks the first vertical drill for you.
- What did this replace, and what constraints did the predecessor leave behind? — finds the fossils in the schema and the API.

## Level 2 — Boundaries

Draw the black box. No internals yet.

- Who and what talks to this system — humans, services, scheduled jobs, webhooks? — the complete inbound surface.
- What are all the entry points: HTTP routes, queue consumers, cron, CLI, admin tooling? — you will find entry points nobody mentions.
- What does it depend on but not control, and which of those are unreliable in practice? — the failure modes you inherit.
- What contracts cannot be broken — public APIs, event schemas, tables another team reads directly? — the tripwires.
- Where does data come from, and where does it durably land? — the persistence map at black-box level.
- What runs on a schedule, and what does each scheduled thing do? — cron jobs are where undocumented business logic hides.

## Level 3 — Structure

- What are the 5–10 top-level components, one sentence each? — the skeleton of the whole map.
- Which are load-bearing and which are peripheral? — where to spend remaining attention.
- What is the deployment topology, and what is the unit of scaling? — distinguishes a monolith with folders from a real distributed system.
- Which boundaries are enforced (network, package, module) versus merely conventional? — conventional boundaries are the ones already violated.
- By commit history, which components change weekly and which have not been touched in two years? — churn predicts where bugs and knowledge live.
- Which two components have the most painful coupling, and why does it exist? — names the architectural debt directly.

## Vertical drill

Interleaved with the levels, not one of them — typically after Level 3 and again after Level 6. Pick one real, important, named thing. Produces a `type: flow` note.

- Trace `<specific request or feature>` end to end: entry point, every layer it touches, external calls, what it writes, what it returns. Name each file in order. — the single highest-value question in the framework.
- Where does it become asynchronous, and what guarantees hold across that boundary — ordering, at-least-once, idempotency? — async boundaries are where correctness quietly dies.
- What is the transaction boundary, and what state is left behind if it fails halfway? — reveals real consistency versus hopeful consistency.
- Which parts of the map built so far did this trace just falsify? — ask it every drill. This is the entire point of drilling, and it usually triggers a backtrack.
- What was surprising while tracing this? — anomalies rarely surface any other way.

## Level 4 — Domain model

Most-skipped level, and the one that makes code readable.

- What are the core entities and their lifecycles or state machines? — the nouns everything else is built from.
- What invariants must always hold, and where is each enforced — DB constraint, application code, or nowhere? — "or nowhere" is where the incidents come from.
- What is the vocabulary, and where does the same word mean different things in different modules? — prevents weeks of subtle misreading. Feeds `<project>-glossary.md`.
- What is the identity model — user, account, tenant, org: how do they nest, and which is the real primary key? — identity confusion is a top source of security bugs.
- Where does the code's model diverge from how the business talks about it? — the translation layer needed in every meeting.
- Which tables or entities are written by more than one component? — shared-write entities are the real coupling.

## Level 5 — Flow

- What is the read path versus the write path for the core entity? — usually more different than anyone admits.
- Where are the caches, what is in them, and how are they invalidated? — invalidation strategy predicts a whole class of bugs.
- What is queued, what is synchronous, and what drove each choice? — reveals latency budget and failure isolation.
- What happens on retry? Which operations are idempotent, and which merely hope not to be retried? — critical before touching anything async.
- Where is state actually mutated — every writer to the central tables. — often shockingly long.
- What is the slowest thing in normal operation, and the slowest under load? — different answers, both useful.

## Level 6 — Cross-cutting

The rules no single feature owns but every feature obeys.

- Where is an authorization decision actually made? Follow one concrete permission check end to end. — auth is almost never where the docs say.
- What changes behavior at runtime without a deploy — flags, config, env, DB-driven settings? — explains "it works in staging".
- What is the error-handling convention, and where is it violated? — violations mark the oldest and riskiest code.
- If this breaks at 3am, what do you look at first — which dashboard, log, metric, or trace? — instant operational grounding.
- How do schema migrations work, and what is the rollback story? — the most common way to cause an outage.
- Which of these are handled well and which are landmines: multi-tenancy, time zones, i18n, money and rounding, PII? — working through them as a list surfaces the embarrassing one.

## Level 7 — Conventions

Now code becomes readable rather than merely visible.

- Where does a new endpoint, model, or job go? Find the canonical example of each. — the most actionable answer for a newcomer.
- Which patterns here are intentional, and which are historical accident that got copied? — stops you cargo-culting a mistake.
- What is the layering rule, and which parts of the codebase predate it? — the codebase's geology.
- What is the testing strategy — what is genuinely covered, what is theater, what runs before a merge? — tells you how much safety net exists.
- What would fail code review here that would not elsewhere? — the unwritten rules.
- Which abstractions should be used, and which look reusable but are traps? — saves a wasted first PR.

## Level 8 — Change and risk

- How do you get this running locally, and what usually breaks first? — the actual onboarding blocker. Check for a documented pitfalls list before investigating.
- What is the path from commit to production, and how long does it take? — sets iteration speed.
- Which files should be feared? Use git history — churn plus complexity plus age is a reliable danger signal.
- What refactor does everyone want and nobody has done, and why not? — the "why not" is usually the deepest constraint in the system.
- What was the most recent production incident, and what did it reveal about the architecture? — incidents are compressed architectural truth.
- If a mistake is made here, what is the worst realistic outcome? — calibrates how carefully to move.

## Repeatable probes

Apply to your own answers, two per cycle, before writing them. Rotate rather than using all of them.

- What is the exception to what was just concluded?
- Who would disagree with this description, and why — which file would they point at?
- What was there not enough context to ask about at this level?
- Which part of that answer is least certain?
- Which single file would teach the most about what was just covered?

## Backtrack questions

For reopening a level a later finding contradicted. Name the trigger in the note.

- Which earlier claim does this finding contradict, and which of the two has the stronger pointer?
- What else was written on the strength of the claim now in doubt?
- Was the earlier answer wrong, or right for a case narrower than it was stated for? — the second is far more common, and the narrowing is the finding.
- What would have caught this in the earlier cycle — a file not read, a question not asked?
