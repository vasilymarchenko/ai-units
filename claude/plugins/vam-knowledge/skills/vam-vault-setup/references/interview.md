# Interview

Six questions, asked in **one numbered batch**, each with a stated default. "Defaults for all" must be a valid answer that produces a working vault.

**Read before you ask.** List the vault's existing top-level folders, and the repos visible in the session or under the working directory, and fold both into the questions as a *proposal to correct*. A user correcting a wrong guess spends ten seconds; a user authoring a taxonomy from nothing stalls.

**Scale to the vault.** One project needs Q2 and little else. An unused `area/*` taxonomy is decay shipped on day one — if the user does not have four problem domains, do not invent four.

---

## Q1 — What is this vault for?

*What kinds of knowledge land here: work projects, general technology, company or org process, personal?*

**Default:** work projects + general technology.

**Drives:** the top-level folders in §1. Each answer earns one folder and no more:

| Answer | Folder |
|---|---|
| work projects | `projects/` |
| general technology | `tech-general/` — knowledge still true at another employer |
| company / org / process | `general/` |
| always | `other/` (unfiled) and `_meta/` (this contract) |

## Q2 — Which projects, and which repositories?

*Name each project or program, and the repos under it. A program with several repos becomes a folder with children.*

**Default:** none — `projects/` stays empty until there is something to put in it.

**Drives:** the `projects/` subtree in §1, the `repo/*` table in the tag vocabulary, and §1's placement rules 2 and 3.

Two things worth eliciting explicitly, because they are the rules that decay first:

- **Does knowledge span several repos of one program?** Yes → that program gets a `cross-cutting/` child, owned by no single repo.
- **Is there a metarepo or workspace repo?** Yes → it gets its own child folder for knowledge *owned by* the workspace machinery (manifests, CI policy, orchestration), which is a different thing from cross-cutting.

**Slugs need not match directory names.** Record the mapping in the `repo/*` table — a vault calling a project `identity` while the checkout is `acme-identity-service` is normal, and the table is what lets a later skill match them.

## Q3 — Which technologies recur?

*The stacks you keep writing notes about.*

**Default:** derive from the repos in Q2 — read their manifests (`package.json`, `*.csproj`, `pyproject.toml`, `go.mod`, compose files) rather than asking the user to list what the code already states. Propose the list; ask them to trim it.

**Drives:** `tech/*` in the tag vocabulary.

Keep it to what has actually been written about. A tag nobody uses is worse than a tag that has to be added later — additions are cheap and the vocabulary is explicitly extensible.

## Q4 — Which problem domains recur?

*The cross-cutting concerns you return to: auth, CI, local setup, observability, performance.*

**Default:** `area/local-setup` and `area/ci`, plus `area/vault-meta` for the contract notes themselves.

**Drives:** `area/*` in the tag vocabulary.

The distinction worth stating if the user hesitates: `tech/*` is *what it is built with*, `area/*` is *what problem it solves*. A note about debugging OIDC in Docker carries both.

## Q5 — Where are the repos on this machine?

*Absolute paths that `vam-project-recon` should search, in priority order.*

**Default:** skip — write no `_meta/recon-environment.md`. Recon degrades to asking for a path once per session.

**Drives:** §1 of `recon-environment.template.md`. Only ask when the user intends to use `vam-project-recon`.

## Q6 — Any project-specific tooling?

*Code-search MCP servers, instruction servers, in-house CLIs a research pass should reach for when present.*

**Default:** none — the portable tooling is already listed in the recon skill's own `environment.md` §4 and needs no entry.

**Drives:** §4 of `recon-environment.template.md`.

**This is deliberately vault-side.** Internal service names are a fact about an employer, not about the plugin, and the plugin is public. They belong in the vault, behind the same trust boundary as the notes describing those same systems.

---

## Registering something new later

Extending an existing contract is a **patch**, and does not need the interview:

| New thing | Patch |
|---|---|
| a project | folder in §1 + row in `repo/*` + `cross-cutting/` child if it spans repos |
| a technology | one entry in `tech/*` |
| a problem domain | one entry in `area/*` |
| a note type | row in §2 + skeleton in §6 + entry in `type/*` — all three, or the type is unusable |
| a repo root or an MCP | a row in `recon-environment.md` §1 or §4 |

Adding a note type is the one that goes wrong: miss the §6 skeleton and every note of that type gets an improvised structure.

Always bump `updated` and extend the note's `source` line with what changed and when.
