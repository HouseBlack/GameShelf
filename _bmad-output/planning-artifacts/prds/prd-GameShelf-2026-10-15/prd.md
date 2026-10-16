---
title: "PRD: GameShelf"
status: draft
created: 2026-10-15
updated: 2026-10-15
---

# PRD: GameShelf

## Vision

GameShelf is a finished product, not a perpetual project. Once 1.0 works, the maintainer's job is done unless he adds a feature, accepts a user request, or fixes a bug. Version 1.0 is the goal; everything after it is a bonus.

Reliability is the product's most important quality, even though it will never appear as a feature in a list. If a person cannot reliably install and use the app, nothing else it does matters. Every requirement below is written to serve that standard, and the cross-cutting reliability requirements are stated as requirements rather than assumed.

## Features

### 1. Accounts and access

GameShelf 1.0 is a closed system with exactly one administrator and any number of members. There is no public signup at any time, including on an empty database. Each account is either `active` or `disabled`. Every active member can browse and download from every configured library; per-library permissions and multiple administrators are out of scope for 1.0 and may be considered after release.

**Initial and sole administrator**
- **FR-1.** The one administrator is created only through an explicit interactive CLI operation. There is no web-based first-user setup.
- **FR-2.** Until that administrator exists, the web application offers no signup or account-creation path.
- **FR-3.** CLI credential entry (bootstrap, password recovery) never exposes a password in logs, terminal output, or command history.
- **FR-20.** The system permits exactly one administrator. Invitations create only members, members cannot be promoted, and there is no multiple-admin or administrator-transfer function. Repeated or concurrent CLI bootstrap attempts cannot create a second administrator.
- **FR-21.** The administrator can change their own username and password.
- **FR-22.** Documented local CLI procedures let the operator (a) recover the administrator's password and (b) clear a sign-in lockout. These are explicit exceptions to the normal authenticated password-change rule.

**Sign-in and sessions**
- **FR-4.** Users sign in with a username and password and keep a durable session that survives application restart.
- **FR-5.** Members cannot invoke any admin action. A forbidden attempt changes no state.

**Invitations**
- **FR-6.** The administrator can generate a shareable invitation link from the UI. No email is sent.
- **FR-7.** An invitation is single-use, expires seven days after creation, and always creates a `member`.
- **FR-8.** The administrator can revoke an unused invitation. A revoked, expired, or already-used link cannot create an account.
- **FR-9.** Invitation tokens never appear in logs.
- **FR-23.** The invitee chooses their own username and password. The administrator does not preassign a username.
- **FR-24.** Opening an invitation does not consume it; successful account creation does.
- **FR-25.** Two accounts can never be created from the same invitation, including when the link is submitted twice at the same time.

**Usernames**
- **FR-26.** Usernames are unique, compared case-insensitively.
- **FR-27.** A member can change their own username in account settings after confirming their current password.
- **FR-28.** The administrator can change a member's username without knowing that member's password.
- **FR-29.** A username change preserves the account's identity, role, and settings.

**Password management**
- **FR-10.** The administrator can reset a member's password by issuing a temporary password. The temporary password is securely generated and is never logged or kept as recoverable plaintext.
- **FR-30.** An administrator-initiated reset invalidates all of that member's existing sessions.
- **FR-31.** A session started with a temporary password is blocked from normal application pages, APIs, and downloads. It may use only the password-change flow, the supporting resources that flow needs, and sign-out, until the password is changed.
- **FR-32.** Once the temporary password is replaced, it no longer works.
- **FR-11.** Any signed-in user, including the administrator, can change their own password after verifying their current password.
- **FR-33.** There is no public "forgot password" flow and no email-based recovery. Forgotten member passwords are handled through FR-10.
- **FR-12.** Passwords (temporary or chosen) never appear in logs.

**Disabling and deleting accounts**
- **FR-13.** The administrator can set a member to `disabled`. A disabled account keeps its role and settings, cannot sign in, and has all existing sessions invalidated.
- **FR-14.** Re-enabling an account requires a fresh sign-in. Sessions from before the disablement are not restored.
- **FR-15.** The administrator can permanently delete a member after an explicit confirmation. Deletion does not remove shared library content or metadata.
- **FR-16.** After an account is disabled or deleted, every subsequent protected request from it fails, including a new or resumed download (see FR-19). Terminating a download already in progress is not required for 1.0.
- **FR-17.** The system refuses to delete, disable, or demote the sole administrator.
- **FR-18.** Account lifecycle state is limited to `active` and `disabled`. There is no scheduled suspension or other account-state workflow. Sign-in lockout (FR-34 to FR-36, FR-42) is an authentication restriction, not an account state.

**Sign-in protection**
- **FR-34.** Five failed sign-ins for the same account within 15 minutes trigger a one-hour sign-in lockout. The administrator follows the same policy as members and is not exempt.
- **FR-35.** The lockout expires one hour after the attempt that triggered it. Attempts during the lockout never extend it.
- **FR-36.** A successful sign-in clears the failure counter only when the account is not currently locked. A sign-in attempt made during an active lockout cannot count as successful, even with the correct password; the attempt is refused (FR-42).
- **FR-37.** The administrator can see a member's actual lock status in the admin UI and can clear a member's lockout there.
- **FR-38.** If the administrator is locked out, they can wait for expiry or use the local CLI unlock procedure (FR-22). Existing authenticated sessions stay usable (FR-39).
- **FR-39.** Failed sign-ins do not invalidate existing authenticated sessions.
- **FR-40.** Sign-in attempts are also rate-limited per source, separately from the per-account policy in FR-34 to FR-36. 1.0 ships with a documented, configurable default whose exact threshold and window architecture proposes. The default gives meaningful protection without needlessly blocking several household users who share one public IP.
- **FR-42.** A valid password does not bypass an active lockout. A new sign-in stays refused until the one-hour lock expires, the administrator clears a member's lockout, or the operator uses the local CLI unlock. On expiry or explicit unlock, the lock and its failure counter are both cleared, so the next incorrect attempt does not immediately relock the account.
- **FR-43.** Source limiting applies whether or not the submitted username exists. A request rejected by the source limiter does not increment an account failure counter or extend an account lockout.
- **FR-44.** The client's source address is determined correctly behind the configured reverse proxy. Only explicitly configured proxies are trusted; client-supplied forwarding headers from other sources are ignored.
- **FR-45.** Failed sign-ins with a nonexistent username take part in the same identifier-based failure and lockout protection as real usernames, using the same username normalization as actual sign-in. This tracking creates no user account or permanent record, expires, and has bounded storage.
- **FR-41.** Unauthenticated responses never reveal remaining attempts, or whether a username exists, is disabled, or is locked, through different text, response structure, status behavior, or obvious processing differences. They use one consistent message: "Unable to sign in. Check your username and password, or try again later. Repeated unsuccessful attempts temporarily block sign-in."

**Acceptance checks for this group**
- **AC-1.** A second administrator cannot be created through any web path, invitation, or promotion; deleting, disabling, or demoting the sole administrator is refused and changes no state. Repeated or concurrent CLI bootstrap attempts cannot create a second administrator (FR-17, FR-20).
- **AC-2.** Submitting one invitation twice, sequentially or concurrently, yields exactly one account; revoked and expired links create none (FR-8, FR-25).
- **AC-3.** A temporary-password session is refused on normal application pages, APIs, and downloads, and allowed only the password-change flow, its necessary supporting resources, and sign-out. The old temporary password fails after replacement (FR-31, FR-32).
- **AC-4.** The fifth failure within 15 minutes locks sign-in for one hour, and attempts during the lock do not extend it (FR-34, FR-35). Separately: a successful sign-in before lockout resets the failure counter (FR-36); a correct password submitted during lockout still fails (FR-42); on expiry, on administrator unlock, and on CLI unlock the lock and counter clear, and the next wrong attempt does not relock (FR-37, FR-42).
- **AC-7.** Failed sign-ins for a nonexistent username are limited and locked like real ones. The response text, structure, and status are identical for unknown, disabled, and locked accounts. The tracking expires, is bounded, and creates no account record (FR-41, FR-45).
- **AC-8.** Source limiting rejects excess attempts whether or not the username exists, and a source-limited request changes no account counter or lockout (FR-40, FR-43). Through the intended reverse-proxy setup, the real client address is limited. A forged forwarding header from an untrusted source does not change the key (FR-44).
- **AC-5.** After disablement or deletion, a new download and a resumed download both fail, and a re-enabled account holds no old session (FR-14, FR-16).
- **AC-6.** No password, temporary password, or invitation token appears in application logs or CLI output after any flow above (FR-3, FR-9, FR-12).

**Known limitation (1.0, accepted).** Anyone who knows the administrator's username can deliberately trigger sign-in lockouts of the administrator. Source limiting reduces this but does not fully prevent it. The administrator's recovery path is the local CLI unlock (FR-22), and existing sessions are unaffected. No additional anti-lockout system is required for 1.0. User-facing documentation states this plainly.

### 2. Libraries and discovery

An administrator configures one or more libraries. Each library points at a folder of game items on read-only storage. GameShelf never creates, extracts, renames, or deletes anything in that storage.

Game items follow this convention (the year is no longer used):

```text
/library/PCgames/pre-installed/
  Assassins Creed [128]/
    Assassins Creed [128].7z
```

**Libraries**
- **FR-46.** The administrator can create, edit, and remove libraries in the admin UI. Each library has a name, a primary folder, and a platform.
- **FR-47.** Multiple libraries are the target for 1.0. If implementing them materially jeopardizes delivering a dependable, complete release, single-library support is the agreed first scope-reduction option. The concrete blocker is raised with the maintainer, and applying the fallback is confirmed with him before it is cut. This is an agreed contingency, not a feature to remove casually, and not a release deadline.
- **FR-72.** Removing a library is a deliberate administrator action that also removes that library's catalog records. It requires explicit confirmation stating that the library's saved matches and manual corrections will be lost. It never deletes or modifies the game folders or downloadable files. Re-adding the library is a fresh import; no preservation mechanism exists in 1.0. A missing mount, inaccessible root, scan failure, or missing game never triggers this behavior (FR-70).
- **FR-48.** A library root cannot equal, contain, or sit beneath another configured library root. Validation compares normalized, resolved paths, not string prefixes.
- **FR-49.** Only configured library roots are readable by the application. Path traversal and symlink escapes out of a root are rejected.
- **FR-50.** A rejected library configuration explains the conflict and leaves the existing configuration unchanged.
- **FR-51.** The platform is chosen from the curated list in FR-71; free text is not allowed. It is organizational only: library labels, navigation, and filtering. It never restricts or changes metadata searches, never causes a different game to be selected, and never overwrites a manual match.
- **FR-71.** The curated platform list for 1.0 is: PC (Windows), Linux, macOS; Nintendo: NES, SNES, Nintendo 64, GameCube, Wii, Wii U, Switch, Game Boy, Game Boy Color, Game Boy Advance, DS, 3DS; Sony: PlayStation, PlayStation 2, PlayStation 3, PlayStation 4, PlayStation 5, PSP, PS Vita; Microsoft: Xbox, Xbox 360, Xbox One, Xbox Series; Sega: Master System, Mega Drive / Genesis, Game Gear, Saturn, Dreamcast; Arcade; and Other / Unspecified. These are organizational labels, not promises of emulator compatibility or special format support. There is no platform-management subsystem in 1.0.

**Discovery**
- **FR-52.** The administrator starts scans manually and sees progress and actionable errors. Scanning is read-only.
- **FR-53.** Only direct child folders of a library root are game items. Nested folders inside a game are never imported as separate games.
- **FR-54.** The convention is `Game Name [128]`. The IGDB ID is the final bracketed group of the game folder's name, ignoring trailing whitespace, and counts only if it contains a valid positive integer. The system does not search backward for an earlier group when the final one is malformed, so `Game [128] [Deluxe]` does not conform and does not select 128. No year is required, and the ID is never taken from an archive filename.
- **FR-55.** A folder with a missing or malformed ID is still imported as an unmatched game and can be filtered and inspected in the administrator's library catalog view (FR-135) rather than requiring a competing separate queue workflow.
- **FR-56.** A syntactically valid ID that IGDB cannot resolve produces an actionable matching error. It never silently falls back to a title-based replacement, and it never erases previously saved metadata. (Matching behavior is detailed in group 3.)
- **FR-57.** A rescan is idempotent. A game item is identified by its library plus its normalized relative path, so repeated or overlapping scans create no duplicate items.

**Downloadable files**
- **FR-58.** Downloadable files are discovered only directly inside a game folder. Files in nested game-content directories are not exposed.
- **FR-59.** The downloadable file types are, case-insensitively, `.7z`, `.zip`, `.rar`, `.iso`, and `.chd`. `.iso` and `.chd` are an explicit allowance for disc images in emulation-oriented libraries.
- **FR-60.** Downloadable files are existing opaque files, served as their original bytes. GameShelf does not inspect, extract, convert, mount, repackage, or launch them. Downloads of unpacked folders and server-generated archives remain out of scope.
- **FR-61.** Every downloadable file in a game is listed with its filename and size. The system does not choose one automatically or combine files.
- **FR-62.** Supported split-volume patterns are recognized by their complete filename pattern, and each existing part is listed individually: `Game.7z.001`, `Game.7z.002`…; `Game.zip.001`, `Game.zip.002`…; `Game.part01.rar`, `Game.part02.rar`…; and legacy RAR sets `Game.rar`, `Game.r00`, `Game.r01`…. Numbered formats do not require an unsplit `Game.7z` or `Game.zip` beside them. A recognized part whose starting volume is absent (`.001`, `.part01`, or the legacy `.rar`) is still listed and flagged as missing its starting volume; the system never hides it or claims the set is complete. The UI states that all parts of a set are needed. An arbitrary `.001` file that does not match a supported archive naming pattern is not listed. There is no "download everything" packaging, extraction, recombination, or archive-integrity verification in 1.0.
- **FR-63.** A non-empty game folder with no downloadable file is still imported. It shows "No downloadable file", offers no download control, and shows the administrator an actionable status.
- **FR-64.** An unknown size is never displayed as 0 B. Size calculation does not run in page-render requests.

**Ignore rules**
- **FR-65.** Scanning skips dot-prefixed game folders and truly empty game folders. It ignores unfinished-download files whose names end in `.tmp`, `.part`, or `.crdownload`; `.part01.rar` is a split-set part, not an unfinished download.
- **FR-66.** No name-based exclusions of "support" or "staging" folders are built in. Any further exclusion is explicit and documented.

**Missing and moved items**
- **FR-67.** A previously discovered game that is absent after a successful scan is marked unavailable. Its identity, metadata, manual match, and lock state are retained.
- **FR-68.** An unavailable game is clearly marked as such, and its downloads are disabled. If the same path returns, availability is restored without rematching or losing saved metadata.
- **FR-69.** For a moved or renamed folder, the administrator can explicitly relink an item to an eligible game folder under a configured library root. The relink preserves the item's catalog identity and manual metadata. If the destination is already represented by another item, the relink is refused with an actionable explanation; records are never silently merged or overwritten. Items are never merged merely because they share an IGDB ID.
- **FR-70.** An inaccessible root, missing mount, or failed scan is reported as a library or scan error. It is not treated as evidence that games were deleted, and it marks no game unavailable and deletes no record. Administrator removal of a library (FR-72) and filesystem disappearance are different events.

**Open for this group:** catalog-size and concurrent-download targets are deliberately left open until real library counts and a realistic deployment and test envelope exist. They are not inferred from household size or turned into approved SLAs. Architecture proposes measurable test targets, labeled as proposals, and they are resolved before performance acceptance criteria are approved.

**Acceptance checks for this group**
- **AC-9.** Overlapping roots are rejected by resolved path, including via a symlink, with a clear explanation and no configuration change (FR-48, FR-49, FR-50).
- **AC-10.** A scan imports only direct child folders, creates a game with no valid ID as unmatched, and never imports a nested folder as a game (FR-53, FR-55).
- **AC-11.** Running a scan repeatedly and concurrently creates no duplicate item (FR-57).
- **AC-12.** Each supported extension, in any letter case, is listed; each split-volume pattern is recognized with its parts listed individually, including numbered sets with no unsplit base; a recognized part with no starting volume is listed and flagged; an arbitrary `.001` file that matches no supported pattern, and a `.tmp`, `.part`, or `.crdownload` file, are not listed; `.part01.rar` is listed (FR-59, FR-62, FR-65).
- **AC-13.** A non-empty folder with no downloadable file shows "No downloadable file", has no download control, and shows no 0 B size (FR-63, FR-64).
- **AC-14.** After a scan, nothing in storage is created, changed, renamed, or deleted (FR-52, FR-60).
- **AC-20.** Folder names ending in a valid positive integer in brackets select that ID, tolerating trailing whitespace; `Game [128] [Deluxe]`, `Game [abc]`, `Game [0]`, and a name with no bracket produce unmatched items; no earlier group is used (FR-54, FR-55).
- **AC-21.** Removing a library requires a confirmation that names the loss of saved matches and manual corrections, deletes only that library's catalog records, and leaves every folder and file untouched. Re-adding it performs a fresh import (FR-72).
- **AC-22.** A game absent after a successful scan becomes unavailable with metadata, manual match, and lock state intact and its download disabled; when the path returns it is available again with no rematching. An unreadable or unmounted root produces a library error and marks no game unavailable (FR-67, FR-68, FR-70).
- **AC-23.** Relinking to an eligible folder keeps the item's identity and manual metadata; relinking to a folder already represented by another item is refused with an explanation and changes no record (FR-69).

### 3. Metadata, matching, and artwork

IGDB is the sole provider of structured game metadata, game identity, and imported screenshot and trailer/video associations in 1.0. Cover and backdrop artwork may come from several sources (FR-109), but artwork sources never take part in identity matching and never overwrite structured fields. Every game item keeps its committed identity and provenance (FR-73) separate from the outcome of its most recent lookup, refresh, or artwork operation (FR-76), so a provider's bad day never erases a good catalog.

**Identity and match state**
- **FR-73.** An item's committed metadata identity is the selected IGDB record: provider name, IGDB record ID, matched-at time, and how the match was made — derived from the folder ID (FR-54) or decided manually. Filesystem item identity (FR-57) stays separate from metadata identity: a metadata change never alters the item's path, and a path change never silently alters the metadata identity.
- **FR-74.** A folder with a valid bracketed ID is initially matched to that exact IGDB record; FR-56 covers a syntactically valid ID that IGDB cannot resolve. Initial matching never falls back to a fuzzy title guess.
- **FR-75.** The administrator sees, per item, the committed match state: **Unmatched**, **Matched from folder ID**, or **Manually matched**. These describe the saved identity, not the last operation.
- **FR-76.** The outcome of the latest lookup, refresh, or artwork operation is tracked and displayed separately from the committed match state. A previously matched game remains matched while also showing a latest-refresh error requiring attention. Statuses are concise and actionable and carry retry controls; the administrator never has to read raw logs to learn what is wrong.
- **FR-77.** A provider failure, timeout, invalid response, empty result, or authentication error never clears known-good metadata, never changes the committed identity, and never removes a manual decision.

**Manual matching**
- **FR-78.** The administrator can find a game in two explicit, separate modes: search IGDB by title and choose a result, or enter an exact IGDB ID and retrieve that exact record without depending on title search. The UI never ambiguously treats a numeric title as an ID.
- **FR-79.** Before a manual match is applied, the UI shows enough identifying information to confirm the choice: at minimum title and IGDB ID, plus release, platform, and cover information when available.
- **FR-90.** A failed title search does not prevent exact-ID lookup. A failed exact-ID lookup never silently selects a title-search result; nothing is applied without an explicit administrator choice.
- **FR-91.** The library's platform label (FR-51) is organizational only. It never filters, restricts, or overrides a manual selection.
- **FR-92.** A successful manual match is saved as a manual identity decision (FR-75) and is protected from automatic replacement (FR-94, FR-95). Success is reported only after the database write commits. A later artwork failure does not undo — or report failure of — an already committed metadata match.
- **FR-93.** The manual identity lock protects which game is selected; it does not freeze every metadata field. An explicit refresh of the selected identity may update unlocked fields while preserving locked ones (FR-99, FR-104).

**Folder ID changes and identity protection**
- **FR-94.** A manual identity decision wins over the folder name. When the folder's bracketed ID differs from the manually matched identity, the difference is shown as a discrepancy notice; the folder's ID is never applied automatically.
- **FR-95.** For a game matched from its folder ID without a manual identity decision, a changed folder ID on a later scan is a proposed identity change, not permission to replace the existing match. The discrepancy is shown, and the administrator explicitly applies the new ID or retains the existing match; nothing is replaced automatically. A folder-ID rename is evaluated only after the item is explicitly relinked under FR-69; this requirement implies no automatic move detection and no merging.
- **FR-96.** Unlocking an identity permits a later explicit identity change. Unlocking alone changes nothing and never authorizes a scan to replace the match. The identity lock is distinct from the per-field locks in FR-97 to FR-102.

**Per-field editing and locking**
- **FR-97.** The administrator can edit these user-facing fields independently: display title, description/summary, display release date/year, genres, developers, publishers, and the cover and backdrop selections (FR-113). This is not an editor for every raw IGDB property. Any additional field exposed as editable in 1.0 obeys the same edit/lock rules.
- **FR-98.** Each editable field can be saved independently and locked independently against provider updates. Saving an edit locks that field by default. An existing value can be locked without changing it, and a field can be unlocked without changing its value; lock state is visible beside the field.
- **FR-99.** Locked fields survive scans, provider refreshes, library refreshes, page reloads, new sessions, and application restarts. Other unlocked fields remain independently refreshable, and an explicit refresh (FR-104) may update unlocked fields only.
- **FR-100.** Explicitly clearing a field is an intentional manual value — including an empty list — and is distinct from a provider omitting that field. Saving a clear locks the field by default like any other edit.
- **FR-101.** A list-valued field (genres, developers, publishers) locks as a whole field, not per entry. The administrator can, for example, keep only one studio in the developer list and lock the list. (This example describes intended editing behavior; it is not an assertion about current IGDB data.)
- **FR-102.** Local edits are local overrides. They never modify IGDB, a shared provider record, or another item's values. Field provenance — manual or provider, with enough audit information to explain each change — is preserved. Architecture chooses the storage model; no event-sourcing system is implied.

**Explicit refresh and rematch**
- **FR-103.** The administrator can trigger a metadata refresh for one item or for a whole library. There is no scheduled automatic metadata refresh in 1.0.
- **FR-104.** A refresh uses the currently selected IGDB identity. It updates eligible unlocked fields only; it never changes the selected identity and never locks or unlocks fields. Omitted fields and failed operations retain their known-good values.
- **FR-105.** A refresh shows progress and per-item failures; a batch containing failures is never reported as wholly successful. Scans may initiate initial matching of newly discovered or unmatched items (FR-74); they never refresh already matched items.
- **FR-106.** A deliberate manual rematch to a different IGDB record is a distinct action from refresh. The UI shows that the identity will change. Field and artwork locks are preserved by default, with a warning that locked overrides remain even if they came from the previous match; the administrator can deliberately unlock or replace those values.
- **FR-107.** A library refresh is never an implicit destructive reset. No blanket "force overwrite locked fields" function exists in 1.0.
- **FR-108.** Before a background refresh or artwork result is committed, the system revalidates that the item's identity, the relevant lock states, and the field versions have not changed since the work started. A stale result never overwrites a field or artwork slot the administrator edited, locked, or rematched meanwhile; the administrator's change wins.

**Artwork sources**
- **FR-109.** For automatic selection of initial or replacement artwork, sources are tried in priority order: SteamGridDB, then IGDB, then Steam. Artwork sources are artwork sources only: they never overwrite the selected IGDB identity, title, description, developers, or any structured field, and they are not competing structured-metadata matchers. No other artwork providers and no general plugin framework are in scope for 1.0.
- **FR-110.** Source priority applies per artwork slot. A good cover does not prove the same source has a suitable backdrop. Only assets appropriate for the slot are used; an unsupported artwork type is a reason to try the next source, never a reason to distort an unrelated image.

**Artwork selection**
- **FR-111.** The administrator can browse suitable SteamGridDB artwork from inside GameShelf, preview candidates, and choose a cover or a backdrop for the game. Opening another website and copying image URLs is not the workflow. The feature is selection only: it does not upload content to SteamGridDB and does not include a local image-editing tool.
- **FR-112.** For each selected asset, the source and the asset's stable identity or source URL are stored where available. The image is fetched and validated into application-managed local storage before the new image is reported as successfully installed.
- **FR-113.** Manually choosing artwork locks that artwork slot by default. Cover and backdrop have separate selections and separate locks. A working manually chosen image is never replaced merely because another provider has higher priority.
- **FR-114.** When source identities are ambiguous, GameShelf never silently chooses another game's artwork; the administrator resolves the association. SteamGridDB association uses verified external-ID mappings where available, with an administrator search/selection fallback when needed; mapping ambiguity never changes the IGDB match. IGDB IDs, SteamGridDB game IDs, and Steam app IDs are kept separate and never assumed interchangeable.
- **FR-115.** Missing artwork API credentials produce a clear administrator configuration status, and configured fallback sources continue to work.

**Local-first artwork with controlled online fallback**
- **FR-116.** The selected local artwork is always tried first. If the local image exists and is usable, it is used; viewing a page — or the existence of a higher-priority source — never triggers provider queries.
- **FR-117.** When the local image for an unlocked slot is missing, unreadable, or invalid, the system attempts bounded online recovery: it prefers re-fetching the currently selected asset when its source information is still available (recovering an existing selection is not a new provider-priority contest), and only if recovery fails or no selection exists does it try suitable replacement artwork in FR-109 order. A successful recovery or replacement is cached locally when storage is usable.
- **FR-118.** If local artwork storage remains unusable but an approved remote source is available, the application may deliver the image through bounded, application-mediated remote delivery so artwork remains visible. Provider credentials are never exposed to the browser. An administrator-visible cache/storage error is shown; remote delivery never pretends to repair local storage. If all sources fail, a placeholder and an actionable status are shown.
- **FR-119.** When the local image for a locked slot is missing, no replacement, no online recovery, and no clearing happens automatically. A placeholder and an administrator-visible missing-artwork status are shown, and the administrator can explicitly repair or re-fetch the exact selected asset, or deliberately choose another image; repairing the same selection need not unlock the slot. An explicitly cleared-and-locked slot remains intentionally empty and triggers no fallback.
- **FR-120.** Normal page rendering reads persisted data and local asset references only and never waits on metadata or artwork providers. Artwork recovery is the narrow exception: it runs isolated and bounded (through an image endpoint and/or background work) without blocking the page. Recovery is deduplicated per slot, concurrency and timeouts are bounded, and failure cooldowns prevent repeated page views or broken storage from creating a provider-request storm. A placeholder may appear while asynchronous recovery runs; remote art is not promised when all providers or the network are unavailable. One failed image never blocks other images, metadata operations, or downloads.
- **FR-121.** Artwork is stored in writable application storage, never in the read-only game library. Artwork selections and locks, and required cached assets, are included in backup and restore planning (AC-39).

**Screenshots and trailer/video associations**
- **FR-126.** During background metadata import and refresh, GameShelf imports IGDB-associated screenshots and trailer/video associations for the selected IGDB record and persists those records in PostgreSQL. Screenshot images and gallery thumbnails are fetched, validated, and cached into writable application storage so rendering a detail page or opening the screenshot gallery never queries IGDB. Video associations are persisted by their external video identifier and metadata; downloading or rehosting video files is not required in 1.0.
- **FR-127.** Imported screenshot and video collections are read-only in 1.0 (no manual screenshot/video upload, video editor, or arbitrary media-source management) and are kept strictly separate from the cover and backdrop slots. Refreshing screenshots or videos never overwrites a selected cover or backdrop and never alters field or artwork locks. A provider or media-fetch failure during import or refresh never discards previously saved usable metadata, screenshots, or video associations.

**Provider access limits, credentials, and integration verification**
- **FR-122.** Provider-aware rate limiting and bounded retry behavior are mandatory for every provider. Each provider's own documented rate and concurrency limits apply with conservative headroom, coordinated across all application and worker processes that share the same credentials, so starting more workers never multiplies a provider budget. Retries are bounded with backoff and jitter, and Retry-After is honored when supplied. 429 responses and provider outages never clear metadata and never cause endless retry. Authentication and configuration errors prompt corrective action instead of a tight retry loop. Batch-refresh work and artwork fallback fan-out are bounded. Useful lookup results and failures are cached for appropriate periods, and identical calls from multiple users are not repeated. API credentials stay server-side and out of logs, committed files, and browser responses. Downloaded images are validated for content, size, and allowed source/redirect destinations; no arbitrary-URL fetch or proxy feature is implied.
- **FR-123.** The IGDB documentation consulted during planning states four requests per second and up to eight open requests. These are treated as provider ceilings to recheck at implementation, not throughput targets to saturate.
- **FR-124.** Each secondary artwork source is a verification gate. SteamGridDB's supported retrieval routes, authentication, caching/usage terms, and attribution requirements are verified before implementation (planning research on 2026-10-15 found its official API v2 documented, with no published numeric rate limit and terms not yet certified; see the addendum). Steam's concrete permitted artwork integration is likewise a verification gate: if it cannot be implemented through an acceptable supported route, the blocker is reported and a bounded decision is requested. Restrictions are never scraped around, and the feature is never silently declared complete. No SteamGridDB rate limit is invented and no supported Steam artwork API is assumed. Missing optional sources never break IGDB matching or local browsing.
- **FR-125.** Provider-dependent behavior is verified primarily with deterministic provider fixtures, plus a small, explicitly identified real-provider smoke test. Mock tests never count as verification of live provider behavior.

**Open for this group:** the Steam artwork fallback is a live verification gate (FR-124). Planning research on 2026-10-15 found no officially supported Steam artwork route, so implementing that fallback awaits either an acceptable supported route or a bounded decision from the maintainer. SteamGridDB terms and attribution requirements are verified before implementation. Neither gate blocks IGDB matching or local-first artwork.

**Acceptance checks for this group**
- **AC-24.** With a fixture where title search returns no result, exact-ID lookup retrieves and applies that exact record (FR-78, FR-90).
- **AC-25.** A manual match and its provenance survive 20 reloads, a new session, and an application restart; the 24-game fixture comparison against PostgreSQL remains required (FR-73, FR-92, FR-102).
- **AC-26.** A changed folder ID cannot overwrite a manual identity decision and cannot silently replace another existing match; the discrepancy is shown and applying or retaining is an explicit administrator action (FR-94, FR-95).
- **AC-27.** A developer list is reduced to one selected studio, saved, and locked; a later refresh updates an unlocked description while preserving the locked developer list (FR-101, FR-104).
- **AC-28.** An explicitly cleared-and-locked field remains empty after scans and refreshes (FR-99, FR-100).
- **AC-29.** Unlocking a field alone changes no displayed data; an explicit subsequent refresh may update that field (FR-98, FR-99).
- **AC-30.** Identity locks and field locks behave independently: selecting another IGDB record preserves locked overrides unless the administrator deliberately changes them (FR-93, FR-96, FR-106).
- **AC-31.** A provider failure leaves the existing match and metadata intact while the latest operation's error is displayed with a retry control (FR-76, FR-77).
- **AC-32.** The SteamGridDB picker previews and saves a selected cover or backdrop, preserves the asset's source identity, and locks only the selected slot (FR-111, FR-112, FR-113).
- **AC-33.** When local artwork is available, ordinary browsing makes no provider requests (FR-116).
- **AC-34.** Missing unlocked artwork triggers deduplicated, bounded recovery of the selected asset, and then the FR-109 source priority if replacement is needed (FR-117, FR-120).
- **AC-35.** Missing locked artwork triggers no automatic provider recovery or replacement; explicit administrator repair of the exact selection is available and needs no unlock (FR-119).
- **AC-36.** Provider failures and cache-write failures never break page rendering; approved remote delivery or a placeholder is used as appropriate, and the administrator sees the error (FR-118, FR-120).
- **AC-37.** Rate and concurrency budgets stay respected across concurrent refresh and image-recovery work; simulated 429, timeout, invalid-content, and authentication failures exercise bounded behavior (FR-122, FR-123).
- **AC-38.** A stale refresh or image download cannot overwrite a field or slot the administrator edited, locked, or rematched while that work was in progress (FR-108).
- **AC-39.** Accounts, metadata overrides, identity locks, artwork selections and locks, and required local artwork survive the documented isolated restore test (FR-121).
- **AC-40.** Background import/refresh persists IGDB screenshot images/thumbnails locally and video associations in PostgreSQL without overriding locked cover/backdrop slots or discarding prior usable media on failure; opening the detail page or screenshot gallery makes no IGDB requests (FR-126, FR-127).

### 4. Browsing and detail

Browsing is designed around one library at a time, artwork-led cover grids, bounded infinite scrolling with `#`–`Z` jump navigation, and rich game detail pages featuring backdrops, screenshots, playable trailers/videos, and clear download controls.

**Library navigation and home view**
- **FR-128.** The application supports multiple libraries in 1.0 even when an installation initially configures only one. Browsing is scoped to one library at a time; there is no combined "All Games" cross-library view.
- **FR-129.** A clear library switcher/navigation is provided. When only one library exists, it opens directly. When multiple libraries exist, the application returns to the user's last selected accessible library when possible, and otherwise shows the library selection.
- **FR-130.** The selected library's organizational platform label (FR-51) is displayed in the navigation/header context and stays separate from IGDB's game-platform metadata. No redundant organizational-platform filter is shown inside a library.
- **FR-131.** The library home view is a straightforward cover grid. No featured/hero carousel or promotional banner is required on the library home view in 1.0 (this does not remove the backdrop from game detail pages).

**Search, filters, and sorting**
- **FR-132.** Users can search within the currently selected library across persisted display title, developers, publishers, and genres using case-insensitive text matching, prioritizing title matches over secondary metadata matches. When a matched title is unavailable, the folder-derived display name is used. Search queries persisted PostgreSQL data only and never queries IGDB or any external provider. No semantic search, external search service, or advanced query language is required in 1.0.
- **FR-133.** Members can filter the selected library by genre, where selecting multiple genres matches any selected genre (`OR` within genres), and search and separate filter categories combine (`AND` across categories) to narrow results. A game-platform filter (based on persisted IGDB platform metadata, distinct from the library's organizational platform label) is shown only when the selected library actually contains meaningful, varied game-platform values, without implying that every listed IGDB platform has a downloadable build in that library. Members are never shown availability or maintenance-status filters.
- **FR-134.** Users can clearly reset active search and filter criteria. The UI distinguishes an empty library from a library where no games match the current search or filters.
- **FR-135.** In the administrator's library catalog view, the administrator has combinable maintenance filters for: **unmatched identity**, **latest metadata-refresh error**, **latest artwork error**, **folder-ID discrepancy**, **missing starting volume**, **unavailable**, and **no downloadable file**, and can include and inspect entries hidden from members. These catalog filters reuse the status definitions from groups 2 and 3 (leaving metadata, artwork, file availability, and matching states distinct — a failed refresh is never treated as an unavailable game) and serve as the administrator's inspection workflow without requiring a competing separate queue.
- **FR-136.** Catalog sorting within the current library and active filters supports: **Display Title A–Z** (default), **Display Title Z–A**, **Release Date/Year (Newest First)**, **Release Date/Year (Oldest First)**, and **Recently Added (Newest First)**. "Recently added" means when the catalog entry was first added to this GameShelf library, not the latest scan, metadata refresh, or artwork update. Release sorting uses the persisted display release value, respecting year-only precision rather than inventing a month/day, and places unknown release values last in both ascending and descending directions. Every sort order uses deterministic tie-breaking so equal values never cause duplicated or skipped entries across loaded batches.

**Cover cards, infinite scrolling, and `#`–`Z` jump rail**
- **FR-137.** Catalog cards normally display only the game's cover artwork. Titles, release years, platform labels, and status badges are not permanently rendered beneath ordinary member cards. At a comparable desktop viewport, visual density and proportions target Gameyfin's seven-across reference layout (approximately `179.043 × 253.65` CSS pixels per card) while preserving cover aspect ratio and adapting the column count responsively to available width rather than forcing seven columns on smaller screens.
- **FR-138.** When a game's cover artwork is missing, the card displays a visually coherent placeholder with a readable display title so the game remains identifiable without hover. Every card provides an accessible game-title name and a clear keyboard focus state; essential identification never depends on pointer hover alone. Administrator-only diagnostic indicators are permitted for administrators without altering the ordinary member card presentation.
- **FR-139.** The catalog uses infinite scrolling on a single catalog page rather than visible numbered pagination, backed by bounded server batches so the application never fetches the entire library, all covers, or all file records up front. The initial and subsequent batch size defaults to a tunable 42 games (six rows at the seven-column desktop reference). The catalog includes a keyboard-operable "Load more" fallback control, clear loading, error, and retry feedback, and an end-of-results indicator; automatic batch loading never steals keyboard focus.
- **FR-140.** Navigating to a game detail page and returning restores the selected library, active search, filters, sort order, `#`–`Z` jump state, and scroll/browsing position rather than resetting to the top of the library. Large-library catalog responsiveness, including long scrolling sessions, is verified within the approved Edge/Alpine frontend without introducing an unapproved frontend framework or virtualization dependency.
- **FR-141.** A vertical `#`, `A`–`Z` jump rail is provided on desktop, with a compact accessible equivalent on smaller viewports. When sorted by **Display Title A–Z**, selecting a character jumps directly to the first matching game within the current filtered library results — even when that game's batch has not yet been loaded — and allows browsing to continue forward from that position without requiring all preceding batches to load first. Selecting a character is a positional jump, not a permanent single-letter filter.
- **FR-142.** The `#`–`Z` jump rail matches against the persisted display title case-insensitively, ignores leading whitespace, does not strip leading articles (such as "The"), and groups titles whose first non-whitespace character is not `A`–`Z` (including digits and symbols) under `#`, consistent with the catalog's `A`–`Z` title ordering. Characters with no matching entries in the current filtered set are visually indicated as empty/disabled, and the jump rail is hidden or disabled when any sort order other than **Display Title A–Z** is active.

**Game detail page, screenshots, videos, and downloads**
- **FR-143.** The game detail page presents the persisted display title, description/summary, display release date/year, genres, developers, publishers, cover, and backdrop, and remains visually coherent and fully usable when any optional metadata, artwork, screenshot, or video is absent. Normal HTML rendering of the detail page reads persisted PostgreSQL records and local asset references only; it never queries external providers or calculates file sizes during page rendering.
- **FR-144.** When persisted screenshots exist for the game (FR-126), the detail page displays a screenshot gallery using locally cached thumbnails and an enlarged viewer with keyboard-operable open, close, previous, and next controls. Opening or navigating the gallery makes no external provider requests.
- **FR-145.** When persisted IGDB trailer/video associations exist for the game (FR-126), the detail page presents them for inline playback using click-to-load external playback based on the persisted video identifier. Click-to-load video playback is an explicit, user-initiated narrow exception to local-only rendering: no external video host is contacted until the user activates playback, and autoplay is never used. The UI states clearly that video playback requires an internet connection and the external video host. If a video is unavailable, blocked, or cannot play inline, the UI displays a clear failure state and an optional direct external watch link without breaking metadata, screenshots, or downloads.
- **FR-146.** The detail page's download section lists every discovered downloadable file with its persisted filename and size, preserves split-volume grouping and missing-starting-volume warnings (FR-61, FR-62), shows "No downloadable file" with no download control when none exists (FR-63), and disables download controls with a clear explanation when an administrator views an unavailable entry (FR-68).

**Backdrop presentation**
- **FR-147.** Game detail backdrops use the approved deep-navy/layered-blue and yellow visual direction with a default treatment of `1px` blur, `75%` brightness, and a dark gradient beneath text so foreground content and controls remain readable.
- **FR-148.** A global administrator setting to adjust backdrop blur and brightness (persisted, validated within safe readable bounds, and equipped with a reset-to-defaults action; never per-user or per-game in 1.0) is an optional nice-to-have for 1.0. If implementing the configurable controls adds disproportionate effort, 1.0 ships with the fixed defaults in FR-147 and defers the setting; screenshots and videos (FR-126, FR-144, FR-145) are never deferred to make room for this cosmetic setting.

**Member versus administrator visibility and detail-page administration**
- **FR-149.** Games marked **unavailable** (FR-67) and games with **no discovered downloadable file** (FR-63) are hidden from ordinary members across the catalog grid, search results, genre/platform filter counts, total counts, and `#`–`Z` jump rail.
- **FR-150.** Direct detail-page requests by a member enforce FR-149: requesting the URL of an entry that is unavailable or has no downloadable file returns the normal not-found response and never exposes an administrator-only entry.
- **FR-151.** An available game that has at least one discovered downloadable file remains visible and downloadable to members even if it is **unmatched**, has incomplete metadata, lacks cover/backdrop/screenshot/video media, has a **folder-ID discrepancy**, or has a **latest metadata-refresh or artwork error**. For such entries, members see the persisted metadata (or folder-derived display title) and placeholder artwork as needed, without raw administrative error messages.
- **FR-152.** A **missing starting volume** on a split archive set (FR-62) is a distinct warning on an otherwise present file set, not an automatic "unavailable" state; the entry remains governed by the split-volume warning and download-safety rules rather than silently hiding the game or claiming the set is complete.
- **FR-153.** Catalog and detail visibility relies on persisted database state rather than scanning the filesystem on page requests. Because underlying files can still disappear between scans, initiating a download safely revalidates file availability at download time (FR-84, FR-87) and returns a controlled, readable explanation rather than a broken response or false success if the file is no longer accessible.
- **FR-154.** When signed in as the administrator, the administrator can discover and open hidden/unavailable/no-file entries via the catalog maintenance filters (FR-135) and direct detail URLs, sees readable administrative diagnostics on the detail page, and can invoke **Match / Rematch**, **Edit / Lock fields**, **Choose Artwork**, and **Refresh** directly from the game's detail page. All administrator visibility and actions are enforced server-side (FR-5), never merely by hiding controls in the browser.

**Acceptance checks for this group**
- **AC-41.** With one library configured, the catalog opens directly into it; with multiple libraries configured, the switcher navigates one library at a time (with no "All Games" view) and restores the last selected accessible library on return (FR-128, FR-129, FR-130, FR-131).
- **AC-42.** Member search matches case-insensitively across persisted display title, developers, publishers, and genres (ranking title matches first and falling back to folder-derived display name when unmatched) with zero external provider requests; genre `OR` filtering, conditional game-platform filtering, filter reset, and empty-library vs. no-results states work as specified (FR-132, FR-133, FR-134).
- **AC-43.** Administrator maintenance filters (`unmatched identity`, `latest metadata-refresh error`, `latest artwork error`, `folder-ID discrepancy`, `missing starting volume`, `unavailable`, `no downloadable file`) combine accurately and surface entries hidden from members without conflating refresh/artwork errors with file unavailability (FR-135).
- **AC-44.** All five sort orders (`Title A–Z`, `Title Z–A`, `Release Newest`, `Release Oldest`, `Recently Added`) sort deterministically within the active library and filters, handle year-only release precision without invented dates, place unknown release dates last in both directions, and never duplicate or skip items across infinite-scroll batches (FR-136, FR-139).
- **AC-45.** Ordinary member cards display cover artwork only (with a titled placeholder when cover art is missing), expose an accessible title and visible keyboard focus state without relying on hover, load in bounded batches via infinite scroll plus a keyboard-operable "Load more" control without stealing focus, and restore library, search, filters, sort, and scroll position when returning from a detail page (FR-137, FR-138, FR-139, FR-140).
- **AC-46.** In `Title A–Z` sort, selecting `#` or a letter in the jump rail jumps directly to the first matching entry in the filtered library (including an entry whose batch was not yet loaded) without filtering out subsequent letters, keeping leading articles like "The" under `T` and non-`A`–`Z` leading characters under `#`; empty letters are indicated and the rail is disabled/hidden in non-`A`–`Z` sorts (FR-141, FR-142).
- **AC-47.** The game detail page renders persisted metadata, backdrop (with default `1px` blur, `75%` brightness, and dark gradient), locally cached screenshot thumbnails with a keyboard-operable enlarged viewer, and click-to-load video playback with no autoplay and zero IGDB or external video requests on initial page load; unavailable/blocked videos show a controlled failure state and optional external watch link without breaking the page (FR-143, FR-144, FR-145, FR-147).
- **AC-48.** Games marked `unavailable` or having `no downloadable file` do not appear in a member's catalog, search, filter counts, or `#`–`Z` rail, and a direct member request to their detail URL returns not-found; available games with downloadable files remain visible and downloadable to members even when unmatched, missing art/media, or carrying a folder-ID discrepancy or latest refresh/artwork error, without leaking raw admin errors (FR-149, FR-150, FR-151, FR-152).
- **AC-49.** Admin controls (`Match`, `Edit/Lock`, `Choose Artwork`, `Refresh`) and diagnostic statuses appear directly on the game detail page for the administrator, and any member request to those endpoints is rejected server-side with no state change (FR-5, FR-154).

### 5. Downloads

All download behavior serves the reliability goal: a download must start, resume, and finish correctly, in a browser or with a standard download manager. Internet Download Manager (IDM) is the representative download-manager test.

**Authorization**
- **FR-19.** Every archive download, including range and resume requests, requires a valid authenticated session and authorization to use the application. Disabled or deleted accounts, and sessions restricted to a mandatory password change, cannot download. Related: FR-16.
- **FR-80.** Every initial, segmented, resumed, and `HEAD` request applies FR-19, including file-information requests download managers make.
- **FR-81.** A request with missing or expired authentication gets an authentication failure. It is never served a login HTML page that looks like a successful download.
- **FR-82.** No public links, authentication bypass, custom IDM plugin, or separate download-token system is required. A protected URL copied without authentication is not expected to work.

**Serving**
- **FR-83.** Ordinary browser downloads work without any download manager.
- **FR-84.** Files are served unchanged from their original bytes, with no repackaging and no staging on the root disk. Downloads use stable application URLs based on item and file identity; absolute filesystem paths are never exposed. At download initiation, the server safely revalidates that the item is permitted and the underlying file is still readable on disk; if a file disappeared or became unreadable after the last scan, the server returns a controlled, readable error rather than a broken stream or false success (FR-153).
- **FR-85.** Byte-range requests and resume are supported. Multiple independent single-range requests for the same file can run concurrently, which allows segmented downloading. Multipart responses with several ranges in one request are not required.
- **FR-86.** Range responses, file lengths, and filenames are correct, and the file carries validators, so resuming a file that has changed does not silently combine different versions.
- **FR-87.** Large files stream with bounded memory, and a client disconnect or a read or permission failure is handled cleanly.
- **FR-88.** Connection limits are bounded but accommodate useful segmented downloading; a file is not limited to one connection. Architecture documents and tests the chosen limits.
- **FR-89.** No speed improvement is promised. Results depend on the network, storage, and client. The tested IDM and browser workflow, and its limitations, are documented.

**Acceptance checks for this group**
- **AC-15.** With IDM's browser integration, a signed-in download is intercepted and completes; the downloaded file's contents match the original (FR-83, FR-84).
- **AC-16.** Through the intended NPMplus deployment, concurrent segments and pause/resume work (FR-85).
- **AC-17.** After session expiry, or account disablement or deletion, new segment and resume requests fail; an unauthenticated request is not answered with a login page (FR-16, FR-19, FR-80, FR-81).
- **AC-18.** Replacing or changing a file during a resume is handled safely, not by silently merging versions (FR-86).
- **AC-19.** A large file is served with bounded memory, with no repackaging and no root-disk staging (FR-84, FR-87).
- **AC-50.** If a downloadable file disappears on disk between scans, attempting to download it returns a controlled, readable error rather than an unhandled failure or empty/corrupt download (FR-84, FR-87, FR-153).

_Remaining feature group (6. Administration and operations, followed by cross-cutting reliability and non-functional requirements) follows._

