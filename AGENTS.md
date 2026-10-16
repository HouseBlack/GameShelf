<!-- bmad:context -->
<!-- Verified 2026-10-15 against a50ab5fc8476bc302bbdef649d357ce6719a64af (only README.md is tracked at that SHA; the BMAD install, planning artifacts, and personal handoff are untracked). Managed by bmad-project-context; edits inside this block are replaced on refresh. Keep anything you want preserved outside the markers. -->

## GameShelf

Self-hosted, browser-based library for pre-existing PC game archives; one installation serves a household. No application code exists yet; this repo currently holds the installed BMAD workflow and the planning artifacts. The stack is settled and not open for reselection: AdonisJS v7 (Edge templates, Alpine.js, Vite), TypeScript on Node 24, PostgreSQL 18 through Lucid, Japa plus Playwright, Docker Compose.

`personal/gameshelf-bmad-product-handoff.md` is the consolidated planning input and supersedes all older plans; read it before any product, discovery, or architecture decision. Authority order: Derek's latest explicit decisions, then the constraints settled in that handoff, then historical plans as reference only. The approved brief is in `_bmad-output/planning-artifacts/briefs/brief-GameShelf-2026-10-15/`.

## Policy

- Do not reopen the settled stack. No React, SvelteKit, Inertia, HTMX, Redis, microservices, or a second ORM/test runner without a concrete need and approval.
- Preserve `_bmad/`, `_bmad-output/`, `personal/`, `.agent/`, `.agents/`, `.github/`, `.opencode/`; add no competing planning system or rival `docs/PROJECT_BRIEF.md`. When scaffolding is eventually authorized, run it somewhere disposable, because a generator run in place can delete these.
- Game storage is read-only. Serve existing archives unchanged; never create, extract, rename, or delete a game archive, and never reintroduce server-side archive generation or unpacked-folder downloads.
- Never log passwords, tokens, or invitation credentials; keep personal paths and hostnames out of distributable defaults and images.
- Never stop, prune, or reuse unrelated Docker resources (`project_arcade`, `project_arcade_go`, `archon`). Gameyfin stays as the fallback and its database is never destroyed.
- No destructive migration, production deploy, proxy/hostname change, or `docker prune` without explicit approval and a rollback plan; never bypass a permission denial with an alternate command.

## Where things are

- Read config through the resolver, never by parsing the TOML directly: `uv run _bmad/scripts/resolve_config.py --project-root .`. It merges `_bmad/config.toml`, `_bmad/config.user.toml`, then the `_bmad/custom/` layers, in that order.
- `_bmad/config.toml` and `_bmad/config.user.toml` are installer-managed and rewritten on reinstall. Durable overrides go in `_bmad/custom/config.toml` (committed) or `_bmad/custom/config.user.toml` (gitignored).
- Work is driven by the installed skills in `.agents/skills/`. For "where am I, what next", use `bmad-help`; its catalog is `_bmad/_config/bmad-help.csv`.
- Record decisions as they happen: `uv run _bmad/scripts/memlog.py append --workspace <run-folder> --type decision --text "..."`. The memlog is append-only (`init`, `append`, `set` only; no edit or delete by design).
- Implementation artifacts belong in `_bmad-output/implementation-artifacts/`; configured project knowledge is `docs/`. Neither exists yet.

## Running and verifying

- Every `_bmad/scripts/*.py` is a PEP 723 inline-metadata script: invoke it with `uv run`, never plain `python`, or its declared dependencies are missing. `resolve_customization.py` and `render_skill.py` require Python >= 3.11.
- Skill script tests: `uv run --with pytest --with ruamel.yaml -m pytest <skill>/scripts/tests/<test>.py`, or run the test file directly when its header names a single-command form (`uv run scripts/tests/test_sprint_plan.py`).
- TODO: no build, test, lint, or run command exists yet and there is no `package.json`. Do not invent invocations; add the real ones here when the AdonisJS app is scaffolded, named after the settled tooling (Japa, Playwright, `node ace`, Docker Compose).

## Conventions that differ from defaults

- Product name is `GameShelf`; use lowercase `gameshelf` for repository, directory, and service identifiers. Gameyfin is the application being replaced, never this project's name.
- Game folders are named `Game Name [IGDBID]`. The bracketed ID is the identity hint; match on that exact provider record and never let a fuzzy title search silently replace an explicit ID or a manual correction.
- A provider failure, timeout, or malformed response must never clear known-good metadata; manual locks survive rescans.
- Persistence invariants (PostgreSQL as catalog source of truth, report success only after commit, filesystem identity separate from metadata identity, idempotent scans) are enumerated in handoff section 12; read them before changing anything that writes.

## Known pitfalls

- `.agent/skills/` and `.agents/skills/` hold byte-identical copies of the 42 installed skills; edit both or they diverge. `.opencode/commands/*.md` and `.github/agents/*.agent.md` are thin pointers into `.agents/skills/`, so never restate instructions in them.
- Only `README.md` is tracked by git, and there is no root `.gitignore`. The BMAD install, harness directories, `.freebuff/`, and `personal/` are untracked: never `git add -A`, stage explicit paths.
- Handoff section 6's folder example still shows `Game Name (Year) [IGDBID]`; the year was dropped, and `addendum.md` in the brief folder overrides that section. Parse the bracketed ID only.
- The bash tool here is mvdan/sh on Windows and lacks `head`, `diff`, `which`, and `wc`. Use the View/Grep/Glob tools or `python -c` instead of piping through them.

<!-- /bmad:context -->

## Skill boundaries

- BMAD artifacts, approved project decisions, and explicit user instructions define scope and architecture. Skills provide task-specific techniques; they do not independently authorize scope changes.
- Preserve the approved AdonisJS, Lucid, Edge, Alpine, and PostgreSQL stack. Do not add dependencies or competing planning systems without approval.
- Use one primary visual-design skill per task. A separate review skill may audit the result without replacing the approved design direction.
- Architecture skills must propose significant refactors for approval before implementing them.
- Documentation must distinguish planned, implemented, tested, and deployed work.
- Do not create issues, post comments, change tracker states, or perform other external writes without user approval.

## GameShelf visual direction

- Design a practical game-library application, not a marketing website.
- Follow the approved blue-and-yellow visual direction.
- Avoid pure black and pure white backgrounds.
- Prioritize game artwork, readable information, browsing, filtering, clear status indicators, and obvious download actions.
- Maintain a consistent design system across screens.
- Use the approved Edge/Alpine frontend. Do not add frameworks, animation libraries, external font services, or other dependencies without approval.