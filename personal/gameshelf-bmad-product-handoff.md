# GameShelf — BMAD Product Brief and PRD Handoff

Prepared for Derek Folds — October 15, 2026.
Status: planning only; project reset; no working or verified application.
Purpose: give the BMAD-Method planning agent in Antigravity a consolidated, current input for collaboratively creating the Product Brief and PRD.

## 1. Start here: authority, task, and boundaries

GameShelf is the official product name. Use `gameshelf` for repository/directory/service identifiers. Gameyfin is the existing application being replaced, not this project's name.

This document consolidates:
- `gameshelf-pre-bmad-handoff.md`.
- `ecc-plan.md`.
- `claude-plan.md`.
- Derek's subsequent product clarifications, including the latest library layout, archive-only downloads, account invitations, FOSS intent, and visual direction.

Authority order:
1. Derek's latest explicit decisions, captured here.
2. Settled constraints and acceptance requirements preserved here from the earlier handoff.
3. Historical plans as supporting references only.

This handoff supersedes contradictory scope, open-question lists, tooling instructions, and commands in the older documents. It is planning input, not a claim that final BMAD artifacts are approved or that software exists. Explicit user decisions are settled; proposed defaults below still require approval.

Immediate task:
- Use the installed BMAD workflow to help Derek create the Product Brief and PRD.
- Carry forward settled decisions without asking them again.
- Ask only focused questions that materially affect requirements.
- Separate product requirements from later technical verification and deployment decisions.

Do not scaffold, install packages, start application containers, repair Claude Code/ECC, publish a repository, change Gameyfin, or restart stack selection. Do not treat any historical implementation command as authorized execution.

Inspect the installed BMAD version/workflows when necessary. Do not invent slash commands, agent names, output paths, or workflow sequences from memory. Preserve BMAD-owned folders/instructions and `personal/`; do not create a competing planning system.

## 2. Executive product context

GameShelf is a self-hosted, browser-based game-library application intended to become a complete, working FOSS product released publicly. Derek's household is the first deployment and acceptance context, not the only intended audience.

Primary value:
- Reliably retain game identity, metadata, and manual corrections.
- Organize several independently configured libraries by platform and purpose.
- Let authorized users browse attractive game cards/details and download existing game archives through a browser.
- Provide a usable admin interface for library, metadata, account, and invitation management.
- Remain straightforward to operate, deploy, verify, back up, and restore.

This is not a hosted multi-tenant service, general-purpose distribution platform, or desktop launcher in the first release. FOSS distribution does not imply unrestricted access to a particular installation or authorization to distribute game content. Serve only content the operator/users are entitled to access and share.

Initial household size: probably 4–5 people; likely fewer than 10. Do not invent a maximum user limit or assume all future operators have the same library scale.

Derek wants AI agents to implement, test, and review almost all code. He approves requirements/architecture and performs short acceptance checks. He does not want to assemble snippets, manually repair hundreds of lines, or become the full-time debugger.

## 3. Latest decisions that override older documents

| Topic | Current decision |
|---|---|
| Product destination | Complete, working publicly released FOSS product, not a disposable experiment |
| Timing | No agreed one-week deadline or experimental release timebox |
| Versions/editions | One version/edition per game for the initial use case; no multiple-edition UI requirement |
| Libraries | Multiple configurable libraries, each with its own primary folder, name, and console/platform type |
| Library layout | Game archives live inside individually named game folders beneath a configured library root |
| Metadata identity | Parse the explicit bracketed IGDB ID from the game-folder name and look up that record |
| Downloads | Existing archives only; no downloadable unpacked folders or on-demand archive generation |
| Archive preparation | Derek prepares archives outside GameShelf; explicitly plans uncompressed `.7z` files |
| Resume | Resumable existing-archive downloads are desired; behavior must be tested before advertised |
| Administration | Working admin UI including creating, deleting, managing accounts, resetting passwords, and creating invitations |
| Invitations | Admin-generated shareable links; no email delivery |
| Email | Out of scope and not to be considered until well after 1.0 |
| Visual direction | PsySonic-inspired deep blue/navy surfaces with yellow accents; not pure black/white |
| Desktop app | Future possibility after server/site work; download location and optional extraction/archive cleanup |

Timing clarification: Derek's earlier “a few days/about a week” comment meant he might become frustrated and abandon the effort. It was not a project deadline, a promised delivery date, or a scope-defining experiment. Do not frame the Brief around a one-week MVP.

Repository clarification: older documents selected private visibility. Public FOSS release is now the eventual destination. Whether development starts privately is not newly decided. No immediate publishing is authorized; FOSS license and release logistics remain release decisions.

Archive terminology: the concrete example and explicit format are `.7z`. Derek later said “only zip files” while contrasting archives with folders. Treat the settled requirement as pre-existing downloadable archives, not a reversal to ZIP-only or permission to convert `.7z` to `.zip`. Document the supported first-release extension list during PRD work; do not repackage files.

## 4. Why GameShelf exists

Derek currently runs Gameyfin in Docker in a dedicated Proxmox LXC through NPMplus/Tailscale. Existing hostname: `gameyfin.houseblack.org`.

Observed issues:
- Quick scans sometimes created duplicate entries, including a misleading 0 B duplicate.
- Baldur's Gate 3 was sometimes rematched as the Toolkit.
- Manual matching of roughly 24 games appeared successful, but visible metadata intermittently disappeared after browser reload.
- Logs did not establish whether writes failed, subsequent updates overwrote them, or the UI displayed persisted records incorrectly.
- Release-date parsing warnings and malformed/HTML provider responses occurred but were not proven causes.
- Broad debug/live logging produced excessive output and browser instability.

No proven H2 corruption or other root cause is established. Do not claim one.

Product lessons: persisted identity and metadata must be trustworthy, success must follow committed writes, rescanning must be conservative, errors must be actionable, and logging must be bounded.

Keep Gameyfin working as the fallback until an isolated GameShelf pilot passes. Do not migrate or destroy the Gameyfin database. GameVault and an experimental Gameyfin fork are historical alternatives, not current tasks.

## 5. Users and essential journeys

### Administrator / self-hosting operator
- Explicitly creates the initial administrator through an interactive CLI operation.
- Configures multiple library roots, names, and console/platform types in the admin UI.
- Starts scans, sees progress/errors, and handles unmatched or unavailable content.
- Reviews exact-ID matches and manually corrects metadata when necessary.
- Creates/manages/deletes user accounts and resets passwords through the admin UI.
- Creates shareable invitation links without SMTP/email infrastructure.
- Deploys, verifies, backs up, restores, and updates the installation using documented procedures.

### Member
- Signs in using an existing account or the approved invitation flow.
- Browses configured libraries, searches games, and opens details.
- Downloads an authorized existing archive unchanged.
- Can resume an interrupted download where the verified client/server behavior supports it.
- Cannot manage libraries/metadata/accounts or bypass download authorization.

### Future self-hoster
- Uses the same product with their own roots, naming conventions, platforms, credentials, and users.
- Must not be forced to use Derek's absolute paths or personal service hostname.
- Manual matching remains useful when their folders lack Derek's IGDB-ID convention.

Do not automatically infer public signup, multi-tenancy, per-library ACLs, or an enterprise permissions system from the FOSS objective.

## 6. Library organization and discovery

Derek's intended mount:

```text
/mnt/gaming -> /library inside Docker
```

This is the intended deployment mapping, not independent verification of a new LXC mount or the underlying Proxmox host path. Real game storage must be mounted read-only for GameShelf.

Confirmed example:

```text
/library/PCgames/pre-installed/
  Assassins Creed (2007) [128]/
    Assassins Creed (2007) [128].7z
```

For this library, the configured primary folder would be:

```text
/library/PCgames/pre-installed
```

Each library is independently configured. Intended categories include:
- Pre-installed PC games.
- PC games requiring installation.
- Separate libraries for emulation-related collections/platforms.

Emulation libraries organize and distribute permitted archives; emulator launching/orchestration is not implied.

Required library administration:
- Multiple libraries.
- A name for each library.
- A primary folder/root for each library.
- A console/platform type for each library.
- Manual scan initiation and visible state/errors.

Folder naming convention:

```text
Game Name (GameYear) [IGDBID]
Assassins Creed (2007) [128]
```

Use the bracketed ID as an explicit identity hint, not a fuzzy title guess. The containing game folder is the authoritative discovery input for this convention. Matching does not depend on the archive filename also containing the ID, although Derek's example includes it.

Discovery interpretation for PRD approval:
- The known example supports direct child game folders under each configured root.
- Exact depth/recursive behavior, supported-archive selection within each folder, and ignore rules still need to be made explicit.
- Do not recursively import nested support directories as independent games.
- One edition/version does not necessarily prove that every folder contains exactly one downloadable file; resolve multiple-archive behavior separately.
- Allowlisted roots only; reject traversal and symlink escapes.
- No arbitrary absolute-path download routes.
- Define Linux case/path normalization and root-overlap behavior.
- Ignore staging/temp/empty support folders under documented rules.
- Unknown size must not be shown as 0 B.
- Size computation/caching belongs in bounded jobs, not expensive page-render requests.

## 7. Metadata requirements

IGDB is the first metadata provider. Verify credentials, API usage rules, and caching/artwork terms before implementation; no claim of completed verification is made here.

Expected behavior:
1. Discover a game folder and its supported archive file(s).
2. Parse a valid bracketed IGDB ID when present.
3. Look up that exact provider identity.
4. Persist provider identity, metadata, provenance, and match state.
5. Show committed records on subsequent page renders.
6. Keep manual provider search/matching and an unmatched workflow for absent, invalid, or unresolved IDs.

Do not let a title-based search silently replace an explicit ID or a manual correction. A failed lookup must not erase known-good metadata.

Persist provider name/ID, provenance, matched-at time, and manual-lock state. Distinguish ID-derived initial matches from manual decisions; automatic initial matching does not automatically define a later refresh policy.

Artwork is cached locally, with error/fallback handling. Routine browsing/page rendering should not require fresh provider calls. Provider calls, retries, caching, and database work must be bounded.

Exact behavior for conflicting folder IDs, changed folder IDs, manual rematching, and refresh actions should be expressed in the PRD without weakening the settled manual-lock/persistence guarantees.

## 8. Accounts, invitations, and administration

Settled:
- Session login with admin/member roles.
- PostgreSQL/database-backed sessions.
- Public signup disabled from first boot, including an empty database.
- Initial admin created through an explicit interactive CLI operation; no web first-user setup.
- Functional admin UI for account creation, deletion, management, password reset, and invitations.
- Invitations are admin-generated shareable links.
- Email delivery and email-dependent recovery are out of scope until well after 1.0.

Invitation onboarding is a controlled exception to closed registration, not unrestricted public signup.

PRD details to settle with focused questions/proposed defaults:
- Invitation expiry, use count, revocation, and assigned role.
- How an admin resets a password without email; whether first-login password change is required.
- Whether account disablement accompanies deletion.
- Protection against deleting/demoting the last administrator and self-lockout.
- Session invalidation after account deletion/disablement or password reset.
- Whether all members see/download all libraries or selective access is actually needed.

These are behavior details, not permission to omit requested administration. Do not invent approved token lifetime, role policies, or reset UX. Secrets, passwords, and invitation credentials must not appear in logs or committed files.

## 9. Downloads — archive-only, unchanged serving

First release downloads pre-existing supported archive files. It does not download unpacked game directories.

Remove from current scope:
- ZIP64 streaming archive generation.
- Archive-generation worker jobs.
- On-demand STORE ZIP creation.
- Archive-generation caches and their storage decisions.
- Enumeration/repacking of archive contents for downloads.
- Generated-directory-download resumability questions.

Retain:
- Authorization for every archive download, including range requests.
- Path traversal and symlink-escape rejection.
- Original archive bytes served without conversion/repackaging.
- Large-file streaming with bounded memory/backpressure and no root-disk staging.
- Correct handling of disconnects and read/permission failures.
- Desired HTTP range/resume support verified end-to-end.
- Tests with files over 4 GiB and representative 75–150 GB games where practical.
- Unicode/deep filesystem paths and explicit unavailable/missing states.
- Actual NPMplus timeouts/buffering and large-download behavior verified before pilot.

Archive “uncompressed” refers to Derek's archive preparation choice, not a server requirement to decompress or inspect content. GameShelf does not create, extract, delete, or rename game archives.

Exact supported extensions, selection when more than one archive exists, and missing/changing-file behavior belong in the PRD. Do not advertise browser resume behavior beyond demonstrated results.

## 10. UI and visual direction

Reference: Derek supplied a screenshot of PsySonic, a music-player application. It is a visual reference, not approved source code or an asset license.

Desired character:
- Deep blue/navy base backgrounds, not pure black or white.
- Layered blue surfaces for navigation, panels, and cards.
- Yellow accent color for important actions and active states.
- Artwork-led presentation with readable text and overlays.
- Rounded cards, restrained borders, and clear hierarchy.
- Attractive covers, backdrop, details, screenshots, and download controls.

Previous preference remains a starting point: 1 px backdrop blur, 75% brightness, dark gradient beneath text, with configurable blur/brightness. Exact palette tokens, measurements, and layout are not finalized.

Borrow visual language rather than music-specific queue/player features. Gameyfin can remain a browsing/presentation reference but not a source of copied code/assets. No hardcoded personal branding or music-player controls are required.

Keep keyboard focus, readable errors, responsive layouts, accessible labels, and readable contrast. The UI must remain visually coherent when artwork is missing. Supply the original screenshot separately to the BMAD/design agent if visual inspection is needed; this Markdown includes its description, not the image itself.

## 11. First-release scope and explicit non-goals

In scope:
- Closed authentication, roles, CLI initial admin bootstrap, durable sessions.
- Working admin interface for libraries, metadata, accounts, password resets, and shareable invites.
- Multiple configurable libraries with names, roots, and platform/console types.
- Manual scans; explicit discovery rules; stable items; missing/unmatched states.
- Folder-ID-driven IGDB lookup plus manual matching/correction.
- Durable metadata/provenance/manual locks and locally cached artwork.
- Searchable/paginated game cards and detail pages.
- Authorized unchanged existing-archive downloads and verified resume support.
- Visible job/scan progress and actionable errors.
- Blue/yellow responsive UI.
- Repeatable deployment, verification, persistence tests, backup/restore, and isolated pilot.

Deferred/out of scope:
- Email, SMTP, email invitations, and email-based recovery until well after 1.0.
- Downloadable unpacked folders and server-generated archives.
- Desktop launcher/install client.
- Gameyfin database import.
- Torrents, plugin marketplace, emulator orchestration.
- Public self-registration, SSO/OAuth.
- Competing metadata providers/matchers and automatic metadata replacement.
- Sophisticated filesystem watchers; manual scans initially.
- Live WebSocket log viewer.
- Multi-tenant hosting and whole-library cryptographic hashing.
- Server-side extraction/deletion/renaming of game storage.

Future desktop direction: after the server/site work, explore a companion app with a chosen download location, optional automatic extraction, and optional archive deletion. Electron is only an example Derek mentioned, not a selected technology. Client-side cleanup does not relax the server's read-only storage rule.

## 12. Persistence and correctness invariants

These are GameShelf requirements, not claims about Gameyfin internals:
- PostgreSQL is the catalog source of truth.
- Reloads render committed records; report save/match success only after commit.
- Save failures are visible and actionable.
- Separate filesystem item identity from metadata identity; metadata rematches cannot overwrite paths.
- Stable item IDs and uniquely constrained root plus normalized relative path.
- Repeated scans/retries are idempotent; no duplicate item/provider records.
- Scans update filesystem availability/size, not manually locked metadata.
- Provider timeout/no result/invalid HTML or JSON/auth failure cannot clear known-good metadata.
- Missing provider fields are distinct from explicit user-clearing actions.
- Missing/moved files do not silently delete catalog metadata; exact reconciliation UX still needs definition.
- Prevent overlapping scans with verified database ownership/leases/locking; enqueue deduplication alone is insufficient.
- Concurrent manual/background changes cannot silently lose the manual decision.
- Audit meaningful changes with request/job correlation IDs.
- Bound external calls, database concurrency, memory use, and log volume.
- Never log passwords/tokens or build a self-amplifying live-log system.

## 13. Settled technical baseline — do not reopen selection

| Concern | Selected baseline |
|---|---|
| Framework | AdonisJS v7 stable, official Hypermedia starter; verify compatibility at implementation |
| Language/runtime | TypeScript, Node 24 and compatible npm 11 baseline |
| Rendering | Edge templates, Alpine.js, starter Vite setup |
| Styling | Conventional reusable CSS/theme variables; no additional frontend framework |
| Database | PostgreSQL 18, explicit tested patch/tag or digest; Lucid models/migrations |
| Sessions | Database-backed PostgreSQL sessions |
| Tests | Japa and Playwright-backed browser tests |
| Deployment | Docker Compose, production build, non-root runtime |
| Foundation services | web, db, one-shot migrate; separate worker later |
| Development isolation | `gameshelf-dev`, dedicated resources/network; localhost web binding |
| Development port | 3333 proposed if free; not a verified allocation |
| Database exposure | No published host port |
| PostgreSQL volume | Agreed PostgreSQL 18 persistent mount at `/var/lib/postgresql` |

Do not add React, SvelteKit, Inertia, HTMX, another ORM/test runner, Redis, paid tooling, or microservices without a concrete need and approval. No PHP or forced Python switch.

Exact package patches/digests are not locked by these documents. Historical plans reported core 7.6.0 and initializer 3.4.0; these are not installed-state facts or current independently verified release claims.

## 14. Queue risk gate — later architecture decision

Historical research described `@adonisjs/queue` as experimental. A PostgreSQL adapter was proposed to avoid another service; this is conditional, not unconditional production approval.

Before adoption, inspect installed-version documentation and prove:
- Real separate-worker execution.
- Retry behavior and duplicate dispatch.
- Termination/restart and stale-job recovery.
- Scan ownership/concurrency safety.
- No duplicate imports after interruption.

Fake-adapter tests alone are insufficient. Stop for a decision if the risk gate fails; do not silently introduce Redis or invent a custom persistent queue.

Archive-only downloads remove archive-generation jobs, not the need for bounded scanner/provider/size work and durable job behavior.

## 15. Selected AI workflow and working style

Planning: Antigravity with Gemini and Claude, using BMAD-Method.
Building/reviewing:
- Crush TUI in Windows Terminal with DeepSeek V4.1 Flash, High Reasoning.
- Freebuff with MiMo 2.6 Pro.

These are Derek's supplied model names/settings, not independently verified compatibility or performance claims. Do not substitute model names or claim benchmark superiority.

Claude Code was rejected after permission/classifier/hook problems. Do not reinstall ECC, repair Claude Code permissions, recommend returning to it, or carry its hooks into the current workflow unless Derek asks.

When implementation is authorized:
- One active builder per branch/worktree; independent review by the other.
- Never let Crush and Freebuff simultaneously modify the same working tree.
- Canonical requirements/state remain harness-neutral; thin harness pointers refer to shared material.
- Research installed APIs, write meaningful tests, implement, run checks, review, fix, record a known-good commit, update state.
- Use practical, bounded tasks and conventional framework patterns.
- Provide one-command verification and complete operator procedures, not snippets to assemble.
- A change of harness does not imply an architecture change.

## 16. Current state and historical environment

Derek reset the project after an unsuccessful Claude Code attempt, preserving BMAD-related folders and `personal/`. Exact surviving contents have not been independently rechecked. Do not assume app files, Git history, dependencies, APP_KEY, containers, or a known-good commit survive.

The failed attempt partially copied/edited starter files but installed/built/tested nothing and launched no app. It is not evidence that AdonisJS failed. Do not resume its numbered tasks or repair its partial edits.

Historical inspection only:

| Item | Last reported value |
|---|---|
| OS | Windows 11 Pro 10.0.26200, 64-bit |
| Shell | PowerShell 7.6.0; earlier Git Bash |
| Workspace | `F:\Projects\git\gameshelf` |
| Node/npm | 24.12.0 / 11.8.0 |
| Git | 2.45.1.windows.1 |
| Docker | 29.8.2, Docker Desktop, Linux containers, WSL2 |
| Compose | 5.5.1 via `docker compose` |
| WSL | Ubuntu and docker-desktop, WSL2 |
| RAM | Approximately 63.6 GB total; free RAM fluctuates |
| CPU | Ryzen 5 9600X, 6 cores / 12 threads |
| Disk | Approximately 212 GB free on F:, 338 GB on C: |

Docker was running at the last inspection. Verify only what is necessary when implementation resumes; no preflight marathon now.

Historical directories included `_bmad`, `_bmad-output`, `.agent`, `.agents`, `.freebuff`, `.github`, and `.opencode`. Preserve surviving BMAD/personal/configuration material.

Unrelated Docker projects/resources include project_arcade, project_arcade_go, archon, and others. Never stop, prune, delete, or reuse them. Gameyfin is on Proxmox, not this development host.

The old LXC used `192.168.4.119`; do not assume reuse. Pilot LXC resources/address/hostname remain later deployment decisions. `/mnt/gaming` is known inside the existing LXC; the underlying Proxmox host path is not established.

## 17. Correct inherited planning defects

Retain useful testing/deployment ideas from ECC and Claude, but neither plan is approved verbatim. Correct these issues in subsequent architecture/implementation planning:
1. Disable public signup before any admin exists, not only afterward. Approved invite acceptance remains separate.
2. Pin the chosen initializer instead of using `@latest` while claiming reproducibility.
3. Verify conflicting initializer/configuration flags against the actual published package/help; do not copy disputed `--db`/`--skip-migrations` behavior.
4. A pinned initializer can still fetch moving starter source; record resolved starter revision where possible.
5. Preserve BMAD/configuration/personal folders through a unique empty scratch scaffold when implementation is authorized.
6. Explicitly switch to PostgreSQL and remove unwanted SQLite driver/development paths.
7. Inspect password hooks before bootstrap; avoid double hashing. Test hashing, credential verification, and no rehash on unrelated updates.
8. Generate a fresh APP_KEY, keep it stable across recreation, and never reuse a shipped example key.
9. Keep DB-dependent tests inside the Compose network and browser binaries/dependencies in the actual browser-test environment.
10. Test valid CSRF submissions and invalid/missing-token rejection.
11. Test account persistence and existing-session continuity separately; a new login/session does not prove the old session survived.
12. Health should include a bounded database check, return 503 on database failure, and disclose no secrets/stack traces.
13. Start healthy DB -> successful one-shot migration -> web. Admin bootstrap is not automatic migration startup.
14. Preserve the PostgreSQL 18 agreed volume mount at `/var/lib/postgresql`.
15. Restore backups into a separate disposable target; never destroy the baseline to prove restore.
16. Verify GameShelf-scoped isolation while preserving unrelated resources; shared Docker need not contain only GameShelf objects.
17. No GitHub authentication/publishing prerequisite for local work.
18. Remove generated-archive work and obsolete invite/email deferrals from inherited plans.
19. Follow installed BMAD artifact paths rather than blindly creating the older plans' rival `docs/PROJECT_BRIEF.md` workflow.

Use focused, version-matched official documentation during implementation; record URL, fetch date, and version in a vendor manifest. Earlier `llms.txt` retrieval failed; do not assume it exists. No premium tooling prerequisite.

## 18. Staged delivery outline — not authorization to implement

Use these as sequencing input, not frozen BMAD epics or a delivery-date commitment.

### Foundation, checkpoint A
Verify initializer behavior, preserve folders, create isolated production-capable web/db/migrate deployment, deliberately use PostgreSQL, enforce LF/secrets rules, non-root runtime, localhost development binding, closed signup, bounded health, and real boot/migration checks.

### Foundation, checkpoint B
CLI initial admin, roles, durable sessions, hashing/CSRF/authorization/browser tests, isolated test DB, one verification command, restart/recreation checks, disposable restore smoke test, known-good commit.

### Persistence vertical slice
Mock provider, one fixture item/card/detail, committed saved match, explicit identity, manual lock, and concurrency/regression tests.

### Real metadata provider
IGDB exact-ID lookup and manual search, bounded requests, caching/artwork, failure retains metadata, no routine render-time provider dependency.

### Libraries, scanner, and worker
Multi-library administration and discovery rules, queue risk gate, stable/idempotent items, missing/size states, visible progress, interruption/concurrency tests.

### Product administration, downloads, and UI
Complete account/invite UI, safe existing-archive serving, range/resume and large-file tests, library navigation/search/pagination/details, blue/yellow styling and accessibility. Account/invite work may be sequenced earlier as dependencies dictate; it remains required for the first release.

### Isolated pilot and release readiness
Small real sample in a new LXC, NPMplus/Tailscale checks, real-download measurements, backup/restore and rollback. Replace the existing hostname only after acceptance and explicit authorization. Public FOSS release needs an approved license and usable documentation; do not imply publication is already authorized.

## 19. Acceptance baseline for the PRD

Translate these into traceable, testable requirements rather than simply copying broad feature labels.

### Metadata and catalog
- One manual match survives 20 reloads, a new session, and application restart.
- Match 24 fixture games; reload and compare saved IDs/metadata against PostgreSQL.
- Folder ID `[128]` selects the corresponding IGDB identity, not a fuzzy title guess.
- Rescans cannot overwrite manual locks; include BG3 versus Toolkit regression.
- Provider failures retain prior metadata and expose actionable errors.
- Concurrent manual/background changes cannot silently lose a manual decision.
- Repeated/overlapping scans do not duplicate root/path records.
- Explicit depth/ignore/archive-selection rules are tested.
- Missing/moved content does not silently destroy metadata.
- Multiple configured roots preserve their identity/settings through restart and restore.

### Accounts and security
- Public signup is blocked even on an empty database.
- CLI initial administrator creation works without logging credentials.
- Account-management and invite requirements have UI and authorization coverage.
- Members cannot invoke admin actions; forbidden attempts cause no database state change.
- Account deletion/reset/invite lifecycle policies are tested once defined.
- Accounts and pre-existing sessions have separate restart/recreation checks.
- Valid CSRF submissions work; invalid/missing tokens fail.
- Archive downloads and range requests require authorization.
- Traversal/symlink attacks fail when discovering/resolving downloadable files.

### Jobs and operations
- Jobs survive browser closure and handle worker interruption without duplicate imports.
- Backup restores accounts, settings, library paths, metadata/matches, and other required application state into an isolated clean target; document separately persisted assets/secrets.
- Production image, migrations, and eventual worker function outside the development server.
- Database-aware health is bounded and accurately reports failure.
- Logging is bounded and secret-free.
- Verification has one documented command and real results.
- Every completion claim names actual checks and lists anything not run.

### Downloads and user experience
- Existing archives are served unchanged, not repackaged.
- Files over 4 GiB and representative large games have bounded memory/no root-disk staging.
- Disconnect cleanup and missing/permission failures are handled.
- Resume claims correspond to tested behavior through the intended proxy/deployment.
- Members can find a game, view details, and download it without a desktop client.
- Missing artwork/errors remain usable and the blue/yellow interface stays readable/responsive.

No generated ZIP/ZIP64 archive acceptance is required because that feature is out of scope.

## 20. Remaining PRD details — not a fresh discovery questionnaire

There is enough settled context to begin the Brief. Resolve the following during focused PRD work, using clearly labeled proposed defaults where appropriate:

1. Discovery: direct-child versus configurable recursive depth; ignore rules; overlapping roots.
2. Archives: supported first-release extensions (explicit `.7z`, clarify `.zip` and any others); multiple archive selection/split-set behavior; what to show when no supported archive is present.
3. Metadata: absent/malformed/invalid bracketed IDs, changed IDs, manual override precedence, and explicit refresh behavior. Manual fallback and preservation are already required.
4. Platform field: console/platform taxonomy and whether it filters IGDB search or is organizational only.
5. Accounts/invites: expiry/use/revocation/role, no-email password-reset UX, last-admin protection, session invalidation, and whether any selective library access is needed.
6. Missing/moved items: unavailable state and deliberate relink/reconciliation UX without silent deletion.
7. Scale/performance: approximate item counts, concurrent downloads, and realistic resource targets for meaningful acceptance; do not invent SLAs from the household user count.
8. Release: FOSS license and operator/contributor documentation before public distribution.

Later, not Brief blockers: pilot LXC address/resources/test hostname, queue adapter/version approval after the risk gate, exact package/image pins, and live environment verification.

Do not re-ask product name, stack, development OS, chosen harnesses, CLI bootstrap, durable sessions, multiple libraries, archive-only downloads, email exclusion, FOSS intent, or the rejection of a one-week deadline.

## 21. Safety and documentation rules

Never commit secrets or bake personal documents/credentials into images. Keep personal paths, hosts, and fixture content out of distributable defaults where inappropriate.

Never run Docker prune, remove unrelated volumes, destroy the working database, deploy to production, change proxies/hostnames, or run destructive migrations without explicit approval and a rollback plan. Never bypass permission denials using alternate commands/tools.

Maintain one canonical approved Brief/PRD and compatible architecture/decision/state/acceptance/vendor records, integrated with BMAD-owned outputs. Historical plans remain references, not parallel instructions. Keep shared state independent of builder harnesses.

## 22. Suggested opening prompt for the BMAD agent

> Read this GameShelf handoff as the current consolidated planning input. We are starting fresh before creating the Product Brief and PRD using the installed BMAD-Method workflow in Antigravity. Carry forward the settled decisions, especially multi-library configuration, folder-based IGDB IDs, existing-archive-only downloads, admin account management/shareable invitations without email, public FOSS intent, and the blue/yellow visual direction. The stack and AI workflow are settled. Do not scaffold, install, repair Claude Code, publish, or reopen framework selection. Help me create the Brief and PRD collaboratively, asking only material remaining questions and separating proposed defaults from approved requirements. There is no one-week delivery deadline. Consult the installed BMAD workflow rather than inventing commands or output paths.

Source note: this is a consolidation of user-provided documents and subsequent explicit decisions, not independent verification of releases, installed tools, current filesystem state, or implemented behavior. The PsySonic screenshot should be shared separately if available.
