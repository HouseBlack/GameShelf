# GameShelf Product Brief: Addendum

Supporting detail that does not fit the 1–2 page brief. Source: `personal/gameshelf-bmad-product-handoff.md` and the brief conversation of 2026-10-15.

## Competitive notes (internal, not for public docs)

GameShelf's public materials should not name other projects. Notes for internal planning only.

| Project | Stack | Observed by Derek | Notes |
|---|---|---|---|
| Gameyfin | Kotlin / Java | Installed and used. Issues sit open, and the maintainer has said he will not have time until winter. | Observed in use: duplicate entries, wrong rematches, and metadata that vanished on reload. A search summary suggested active development (v2.4.x line, a desktop client), so verify dates before ever quoting activity publicly. |
| GameVault | C# | Reddit consensus reported as "paywalled, never using it". | Server is source-available, not FOSS. GameVault+ is a recurring subscription (about €4.99/month at the time of the search) covering cloud saves, Steam/Discord/Playnite integration and themes. The core library is free. No lifetime option, per Derek's reading of their site. |
| Drop OSS | Rust / Vue | Tried. Considered the closest peer. | Young and less active than initially promised. Created in response to GameVault+. |

The positioning is trust, meaning reliability and simplicity, and not a feature race. Verify any factual claim before publishing it.

## Resolved decisions that shape the PRD

- **Folder naming:** `Game Name [IGDBID]`, for example `Assassins Creed [128]`. Only the bracketed ID matters. The year was dropped, which overrides handoff section 6.
- **Cut order:** Multiple libraries are the 1.0 target and a core feature. If they materially jeopardize a dependable, complete release, single-library support is the agreed first scope-reduction option. Raise the concrete blocker and confirm the fallback with Derek before cutting it. This is a contingency, not a deadline. Resumable downloads are core and not a cut candidate. All other in-scope items are core.
- **Privacy:** No telemetry, tracking, crash reporting, or maintainer-run servers. Logs are exported only by the user. Update checks run user-side if at all.
- **Release posture:** Hobby project, FOSS, free forever, community PRs welcome from release, donations accepted but unlocking nothing.
- **1.0 acceptance:** Hands-off month for the household, including the remote member on Starlink, with no intervention beyond a container restart.

## Roadmap wish list (unordered, post-1.0)

Windows and Linux desktop apps, a fully mobile-friendly website, themes, email invitations, user signups, cloud saves, and more. Prioritize closer to or after 1.0.

## Open details carried into the PRD

From handoff section 20, none of which block the brief:
1. **Discovery:** direct-child versus recursive depth, ignore rules, and overlapping roots.
2. **Archives:** supported extensions (`.7z` is explicit), multiple-archive and split-set behavior, and what to show when none is found.
3. **Metadata:** absent, malformed or changed bracketed IDs, manual override precedence, and refresh behavior. *(Resolved in PRD group 3, 2026-10-15.)*
4. **Platform field:** taxonomy, and whether it filters IGDB search or is organizational only.
5. **Accounts and invites:** expiry, use count, revocation, role, no-email password reset, last-admin protection, session invalidation, and whether selective library access is needed.
6. **Missing or moved items:** unavailable state and relink UX without silent deletion.
7. **Scale and performance:** item counts, concurrent downloads, and realistic resource targets, without inventing SLAs from the household size.
8. **Release:** FOSS license and operator and contributor documentation.

## Reconciliation after PRD Feature Group 3 (2026-10-15)

Derek's group-3 decisions expand scope and revise identity/lock semantics. Where older material conflicts with them, these rulings apply; the PRD carries the requirements.

- **Artwork sources (scope expansion).** The older IGDB-only artwork baseline is superseded: automatic artwork selection tries SteamGridDB, then IGDB, then Steam, per artwork slot. IGDB remains the sole structured-metadata and identity provider. SteamGridDB and Steam are artwork-only sources, not competing structured-metadata matchers, so handoff section 11's "competing metadata providers/matchers ... out of scope" line keeps its meaning unchanged. No other artwork providers and no plugin framework are in scope.
- **Changed folder IDs.** The proposed automatic re-match default for already matched items is rejected. A changed folder ID is a proposed identity change requiring an explicit administrator decision; it never overwrites a manual identity decision and never silently replaces an existing match. Folder IDs drive initial matching only.
- **Per-field editing and locking (scope expansion).** Per-field editing with independent per-field locks is required for 1.0 (display title, description/summary, display release date/year, genres, developers, publishers, cover and backdrop). Saving an edit locks the field by default; an explicit clear is a manual value; list locks are whole-field. The identity lock is distinct from field locks: a manual match protects which game is selected, not every field value.
- **Provider work during browsing.** Handoff sections 7 and 18 ("routine browsing/page rendering should not require fresh provider calls", "no routine render-time provider dependency") stand as written for page rendering, but bounded, deduplicated artwork recovery and application-mediated remote image delivery are the sanctioned narrow exception when local artwork is missing. Metadata rendering itself never waits on providers.
- **Open details item 3 above is resolved** by PRD group 3 (identity, precedence, refresh, and rematch behavior).

## Reconciliation after PRD Feature Group 4 (2026-10-16)

Derek's group-4 decisions refine browsing, media, and visibility requirements:

- **One library at a time.** Multiple libraries remain in 1.0 scope, and users browse one library at a time via a library switcher (no combined "All Games" cross-library view). The library home is a straightforward cover grid; no featured/hero carousel is required in 1.0.
- **Infinite scrolling and `#`–`Z` jump rail replace numbered pagination.** Handoff section 11's "Searchable/paginated game cards" is superseded on presentation mechanics: the catalog uses infinite scrolling on a single page backed by bounded server batches (default tunable 42 items) plus a `#`, `A`–`Z` positional jump rail and a keyboard-operable "Load more" fallback, rather than visible numbered pagination.
- **Screenshots and trailers/videos required in 1.0.** Group 3's cover/backdrop slot rules stand for editable artwork slots, and IGDB screenshots (cached locally with thumbnails and an enlarged keyboard-operable viewer) plus IGDB trailer/video associations (persisted identifiers with click-to-load inline playback) are required in 1.0 as read-only imported media collections.
- **Click-to-load external video playback exception.** Normal page and gallery rendering makes zero external requests. Explicit user-initiated click-to-load trailer/video playback is the second narrow, intentional exception alongside bounded missing-artwork recovery, with no autoplay and graceful handling when the external host is unreachable or blocked.
- **Member vs. administrator visibility.** Games that are `unavailable` or have `no discovered downloadable file` are hidden from ordinary members across catalog, search, filters, counts, `#`–`Z` navigation, and direct detail URLs, while remaining inspectable by the administrator via catalog maintenance filters and direct detail URLs. Games that are available and have downloadable files remain visible to members even when unmatched, missing artwork/media, or carrying a folder-ID discrepancy or latest refresh/artwork error.

## Provider verification notes (2026-10-15, planning research)

Evidence for the PRD FR-124 verification gates, gathered by web research on 2026-10-15. Recheck against live official documentation before implementation; nothing here is a certified terms review.

- **SteamGridDB — officially supported route.** Public documented API v2 (https://www.steamgriddb.com/api/v2) with API-key authorization; game search/lookup and artwork retrieval including grids (cover/portrait) and heroes (backdrop). No published numeric rate limit was found in the accessible official documentation, and the official Terms page (https://www.steamgriddb.com/terms) could not be read during research, so caching/usage and attribution requirements remain unverified. Do not invent a rate limit; review the terms directly before implementation.
- **Steam — no acceptable documented artwork route (blocker).** Valve documents library asset uploads for partners (https://partner.steamgames.com/doc/store/assets/libraryassets) and app-list APIs (ISteamApps is deprecated in favor of IStoreService/GetAppList), but no official artwork-retrieval API for third-party applications. The store `appdetails` endpoint (returns header/capsule images) and Steam CDN `library_600x900`/`library_hero` URL patterns are observable but undocumented for this use; the Steam Web API Terms (https://steamcommunity.com/dev/apiterms) establish call limits but no artwork rights, and the Subscriber Agreement (https://store.steampowered.com/subscriber_agreement/) licenses rather than sells content. Per FR-124: do not implement Steam artwork as an assumed-permitted fallback; a bounded maintainer decision is required (drop Steam from the 1.0 fallback chain, keep it gated pending acceptable terms, or explicitly accept the unspecified-route risk).

## Visual reference

PsySonic screenshot (music player), shared by Derek: deep navy base, layered blue panels, yellow accents for primary actions and active states, a large hero banner with a featured item (not required on the 1.0 library home view), rounded cover-art card carousels, a left navigation rail, and a right-hand panel. It is a visual-language reference only, with no copied code or assets and no music-player features.

