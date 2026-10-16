---
title: "Product Brief: GameShelf"
status: final
created: 2026-10-15
updated: 2026-10-15
---

# Product Brief: GameShelf

## Executive Summary

GameShelf is a free, open-source, self-hosted library for PC game archives. It is built on one promise: it works the first time, every time. An operator creates an admin account, points it at a game folder, watches games fill in with art and details, and downloads the first one within 15 minutes, with no wiki and no troubleshooting.

Self-hosters today choose between a stalled, unreliable app, a paywalled one, and a young one that has not kept its promises. Each sends people down a troubleshooting rabbit hole they never asked for. GameShelf is for everyone who wants to share a game library without that: a child learning homelabs, a family's "cool uncle", a veteran homelabber, a first-time self-hosting retiree.

Version 1.0 is the basics done reliably: closed accounts, IGDB-matched libraries whose corrections persist, and resumable downloads for a remote household on an unstable connection. It is free forever, with no paywall, no gating, and no telemetry. It begins as a Toyota and aims to grow into a Lexus.

## The Problem

People who keep a DRM-free PC game collection on their own server have no option that is reliable, free, and simple. The most-used option misbehaves: scans create duplicate entries, a game gets rematched to the wrong title, and manually corrected metadata disappears after a browser reload. Its maintainer has said publicly that he will not have time for fixes until winter, so open issues sit unresolved. The best-known alternative gates features behind a paid tier, which many self-hosters refuse on principle. A newer free project is promising but young and less active than promised. Today a self-hoster has to pick between unreliable, paywalled, or immature.

The cost is felt by the household that depends on it. A remote member on a variable connection (Starlink) needs to sign in, browse, and download tens of gigabytes. An interrupted download or a vanished catalogue entry means the tool is not trusted.

## The Solution

GameShelf is a self-hosted, browser-based library for PC game archives. Setup is the whole pitch: in the first 15 minutes an operator creates an admin account, points GameShelf at a library folder, watches games populate with cover art and details, and downloads a first game. Anything less than that is a failure of the product.

Version 1.0 is the basics done reliably, and it is not an attempt to do everything. It has multiple libraries, folder-name IGDB matching with manual correction, and archive downloads that resume after an interruption. Admins manage accounts and create shareable invitation links from the UI. Metadata, once matched or corrected, stays put. The interface is a navy and yellow, artwork-led design that looks better than the tools it replaces.

## What Makes This Different

GameShelf does not compete on a feature list. It competes on trust. The pitch is a car that starts every morning: a Toyota first, evolving into a Lexus. Existing options each send users down a troubleshooting rabbit hole that sits well outside what they set out to do.

- **Reliability is the product.** Matched or manually corrected metadata persists. Downloads resume. The default fix for a hiccup is a container restart, and real fixes ship as updates before most users notice.
- **Nothing gated, nothing tracked.** The software is FOSS and free forever, with no paywall and no feature gating. Donations may be accepted but never unlock anything. No user data ever reaches the maintainer: there is no telemetry, analytics, crash reporting, or phoning home, and no server run by the maintainer. Users can export logs on their own initiative. Update checks, if any, run on the user's own server, and updating stays manual or user-automated. Metadata providers such as IGDB follow their own policies, which GameShelf does not control.
- **Simple enough for anyone.** Setup needs no wiki. A person new to self-hosting should be able to install it and share a library.
- **A fresh, modern stack from the ground up.** It is a new codebase, not a fork or re-skin. Anyone who opens the repo should see that it is built differently, and it is chosen to stay maintainable by one person with AI help.

The honest moat is execution and focus: a narrow scope, heavy testing, and a maintainer who runs it daily. If a competitor fixes reliability or drops a paywall, users benefit, and GameShelf's job is still to be the one people recommend because it has always worked.

## Who This Serves

GameShelf is for everyone who wants to host a game library and share it, and it is built "for everyone, by everyone", with community contributions welcome from release. Good documentation matters, but the product should not need it for the basics.

- **The first-timer:** a 12-year-old setting up a homelab to share games with friends, or a 70-year-old discovering self-hosting. Success is installing it and sharing games with no issue that a container restart can't fix.
- **The family host:** the "cool uncle" who wants relatives, near and far, to browse and download games without calling for help.
- **The experienced homelabber:** someone tired of troubleshooting software that should just work. Success is installing it and forgetting it.
- **The household member:** signs in, browses, and downloads, including over a variable connection such as Starlink.

The focus is PC games, not emulation-first tools. Emulation libraries are supported as organization and download, not launching.

## Success Criteria

Version 1.0 is a success when all of the following hold:

- **First 15 minutes.** A fresh install gets from admin account creation, to a library scan, to games populated, to a first completed download, with no wiki and no troubleshooting. The bar is that a non-technical person, or a 12-year-old, can do it.
- **Hands-off month.** The household, including the remote member on Starlink, uses it for a month. In that time the operator does not need to take it down, change code, or repair anything. A plain container restart is the only acceptable intervention.
- **Metadata persistence.** A manual match survives 20 reloads, a new session, and an application restart.
- **Resilient downloads.** An interrupted archive download resumes where it stopped, including over an unstable connection. Reliability outranks raw speed.
- **Recommendable.** The test is that a stranger asking "which one should I use?" gets "it's always worked for me" as the honest answer.
- **No surprises at release.** Nothing is known to be broken at 1.0. Bug fixes after 1.0 should come from new feature work, not from basics that never worked.

## Scope

**In for 1.0:**
- Closed accounts: admin and member roles, initial admin created through the CLI, durable sessions, and no public signup.
- Admin UI to create, delete and manage accounts, reset passwords, and generate shareable invitation links, with no email.
- IGDB identity and structured-metadata matching from the bracketed ID in the folder name (`Game Name [IGDBID]`), manual matching (title search and exact ID), per-field metadata editing and locking, and artwork from SteamGridDB, IGDB, and Steam (with an in-app SteamGridDB picker and local-first storage) whose corrections and locks persist.
- Manual library scans with visible progress and actionable errors.
- Searchable cover-art game cards with infinite scrolling and `#`–`Z` jump navigation, detail pages with backdrops, screenshot galleries, and playable trailers/videos, and a navy and yellow artwork-led interface.
- Authorized downloads of existing archives, served unchanged, with resume.
- Docker deployment, backup and restore, and clear setup documentation.
- **Multiple libraries, each with a name, folder and platform type.** This is the target for 1.0 and a core feature. If it materially jeopardizes a dependable, complete release, single-library support is the agreed first scope-reduction option; the blocker is raised and the fallback confirmed with the maintainer before it is cut. This is a contingency, not a deadline.

**Out for 1.0 (roadmap):** desktop apps, email invitations, public signups, themes, cloud saves, full mobile polish, downloadable folders or server-built archives, SSO, plugins, and emulator launching.

**Never:** paywalls, feature gating, telemetry or tracking.

## Vision

If 1.0 earns trust, GameShelf grows by building on a base that already works. The ideas below are an unordered wish list that will be prioritized closer to or after 1.0, once a running, trustworthy server is secured:

- Windows and Linux desktop apps, with a chosen download location and optional extraction and cleanup.
- A fully mobile-friendly website.
- Themes.
- Email invitations.
- User signups.
- Cloud saves.

If the project grows, it will take on community maintainers and pull requests, and may accept donations (Ko-fi, Patreon) that unlock nothing.
