# Open TV Stick: Stage 0 Weekly Plan

**Status:** Working draft, October 16, 2026
**Covers:** Week 1 (Oct 19, 2026) through Gate 1 (Jun 28, 2027)
**Companion files (keep in the same folder):**

- `01_design_decisions.md`: why things are built this way ([open it](01_design_decisions.md))
- `02_roadmap.md`: the multi-year picture ([open it](02_roadmap.md))
- `open_tv_stick_business_model.xlsx`: financial model ([open it](open_tv_stick_business_model.xlsx))

---

## How to use this plan

- **Assumes full-time work** (roughly 40–50 hours a week) by the founder, who writes the code. If you're part-time, roughly double each week's duration and shift the dates; keep the order.
- **Each week** has engineering to-dos, business to-dos, and a "done when" test. The week is finished when the test passes, not when the hours are spent.
- **Holiday weeks** (Thanksgiving, Christmas, New Year) are deliberately lighter.
- **If you fall behind,** cut scope, not quality. Remote navigation and playback reliability are never what gets cut.
- **Check the boxes** as you go (`- [x]`). This file works well committed to the repository under `/docs`.

---

## Overview

### Stage 0a: Build (Weeks 1–16)

| Month | Weeks | Focus | Milestone |
|---|---|---|---|
| Oct 2026 | 1–2 | Setup, provider interface, navigable home screen | App runs on a real TV device with a remote |
| Nov 2026 | 3–6 | Jellyfin end to end, catalog database, Internet Archive starts | Watch and resume a movie from Jellyfin |
| Dec 2026 | 7–11 | Unified search, handoff, detail pages, landing page | End-to-end demo on 3 devices |
| Jan 2027 | 12–15 | Private alpha, hardening, store and launch prep | Alpha feedback fixed; stores submitted |
| Feb 2027 | 16 | Public beta v0.1 | **Gate 0** |

### Stage 0b: Traction (Weeks 17–37)

| Month | Weeks | Focus | Milestone |
|---|---|---|---|
| Feb 2027 | 17–20 | Stabilize, contributor docs, more handoff apps, start hiring | Weekly release rhythm |
| Mar 2027 | 21–24 | Trial hire (Plex), OTA provider, ODM sample evaluation | OTA live TV in the app |
| Apr 2027 | 25–28 | Plex shipped, launcher-as-home prototype on RK3539 | "This is the hardware experience" demo |
| May 2027 | 29–32 | ODM shortlist, BSP hire prep, real numbers in model | v0.5; updated spreadsheet |
| Jun 2027 | 33–37 | Reliability, translations, Gate 1 package | **Gate 1 go/no-go** |

---

## Before Week 1: shopping and setup list

| Item | Approx. cost | Why |
|---|---|---|
| Walmart Onn 4K Pro | ~$60 | Most popular budget Google TV box; your main test device (buy locally for speed) |
| Generic Android TV box (AOSP-style, not Google TV) | $30–60 | Lets you set a default launcher; closer to your own device |
| RK3539 stick (e.g., HS89 T15, 4GB/32GB) | ~$45 | Your future chip; AliExpress shipping takes 2–4 weeks, so order on day 1 |
| Spare Bluetooth remote / USB keyboard | $15–25 | Debugging and text entry |
| USB-C power meter | ~$20 | Power and thermal sanity checks |
| Existing PC or NAS | $0 | Runs a Jellyfin test server (Docker is easiest) |

**Legal test media for Jellyfin:** Blender Foundation open movies (Big Buck Bunny, Sintel, Tears of Steel; Creative Commons licensed), plus codec test files in H.264, HEVC, AV1, and HDR10. Never use pirated files, even for testing; the repository and screenshots are public.

**Stage 0a cash budget:** roughly $5–15K (devices, domain and hosting, company formation and attorney consult, trademark filing, Play developer account).

---

## Stage 0a: Build

### Week 1 (Oct 19–25): Foundations

**Engineering**
- [ ] Install the latest stable Android Studio; create an Android TV emulator (Android 14, TV image)
- [ ] Create GitHub organization and repository; add README (vision paragraph), `.gitignore`, placeholder LICENSE
- [ ] Set up Gradle version catalog (`libs.versions.toml`) and create empty modules: `:app`, `:core:provider-api`, `:core:catalog`, `:core:player`, `:providers:jellyfin`, `:providers:archive`, `:providers:handoff`
- [ ] Add Compose for TV dependencies; build a "hello" screen that responds to D-pad focus
- [ ] Enable developer options and network ADB on the Onn; deploy the app to it
- [ ] Install Jellyfin (Docker) on your PC; load the legal test media library

**Business**
- [ ] Order all test devices (RK3539 stick first, given shipping time)
- [ ] Brainstorm 10–15 name candidates; run a USPTO trademark search and domain check on each

**Done when:** The app runs on both the emulator and the Onn, the D-pad moves focus, and the repo is pushed.

---

### Week 2 (Oct 26 – Nov 1): Provider interface and home screen

**Engineering**
- [ ] Write `provider-api` v0 (see `01_design_decisions.md`, D-14):
  - Data models: media item (id, provider id, title, type, year, artwork, external IDs such as TMDB/IMDb/Wikidata), row, details
  - Playback options as a sealed type: **Native** (stream URL, headers, MIME type), **Handoff** (deep link, package name, fallback search), **Live** (channel)
  - Provider interface: id, display name, home rows, search, details, resolve playback, capability flags
- [ ] Build a fake provider with dummy data
- [ ] Set up Koin and Compose navigation (home → details → player)
- [ ] Home screen with lazy rows; focus rules: sensible initial focus, focus restored when returning from details, no "lost" focus at row edges
- [ ] Coil with size-limited image loading

**Business**
- [ ] Narrow names to 3; check social handles and domains; register the domain for the favorite
- [ ] Book an attorney consult on entity type (Texas LLC vs Delaware C-corp; angels often prefer C-corps)

**Done when:** You can navigate fake rows with a physical remote on the Onn, open a detail page, press back, and land exactly where you were.

---

### Week 3 (Nov 2–8): Jellyfin provider, part 1

**Engineering**
- [ ] Server address entry that works with a D-pad (stretch: QR code login from a phone)
- [ ] Username/password authentication; store the access token securely
- [ ] Browse user libraries with paging
- [ ] Use Jellyfin's image API with a max-width parameter so the server resizes artwork (big memory saver on cheap devices)
- [ ] Detail screen: synopsis, artwork, year, runtime, seasons/episodes for series

**Business**
- [ ] Attorney consult; decide entity type; file formation documents
- [ ] Pick final name (pending trademark check)

**Done when:** You can log in to your Jellyfin server and browse movies and shows with real artwork.

---

### Week 4 (Nov 9–15): Jellyfin provider, part 2: playback

**Engineering**
- [ ] Media3 player in `:core:player`; play Jellyfin direct-play URLs
- [ ] Codec check (MediaCodecList): direct-play if the device supports the format, otherwise request a server-side HLS transcode
- [ ] Remote-friendly player controls: play/pause, seek ±10s, audio and subtitle track selection
- [ ] Report playback progress to Jellyfin so resume works across devices
- [ ] Create `docs/device-matrix.md`; test H.264, HEVC, AV1, and HDR10 files on all available devices and record results

**Business**
- [ ] Obtain EIN once the entity is formed; open a business bank account
- [ ] Request a D-U-N-S number (free, but can take up to 30 days; required for a Google Play **organization** account)

**Done when:** You can watch Big Buck Bunny from Jellyfin to the end, then resume it from the middle on another device.

---

### Week 5 (Nov 16–22): Catalog database and performance

**Engineering**
- [ ] `:core:catalog` with Room: title entity, source entity (title ↔ provider item), external IDs
- [ ] Background sync from providers (WorkManager is part of AndroidX and doesn't require Google services)
- [ ] "Continue watching" row from local progress plus Jellyfin progress
- [ ] Profile on your cheapest device: memory, frame drops while scrolling; tune Coil cache limits

**Business: ODM outreach**
- [ ] Contact 3–5 RK3539 stick makers (Alibaba, AliExpress stores, CNX Software listings). Ask:
  - Is the Android 14 BSP/SDK source available to customers?
  - Can the bootloader be unlocked and custom firmware loaded?
  - White-label minimum order quantity; unit price at 1,000 and 5,000 units
  - Support for a custom remote and packaging
  - Certifications held (FCC ID, CE) and whether they can be reused
  - Lead times
- [ ] Order 1–2 samples from the most responsive

**Done when:** Continue-watching works; scrolling stays smooth on the slowest device; ODM emails are sent.

---

### Week 6 (Nov 23–29, Thanksgiving): Internet Archive, part 1 (short week)

**Engineering**
- [ ] Internet Archive provider: Advanced Search API for feature-film collections; metadata API for file lists
- [ ] Choose the best playable file per item (MP4/H.264 derivatives are most compatible)
- [ ] **Only show public-domain or openly licensed items.** Not everything on the Archive is public domain; filter by license/rights fields and stick to curated public-domain collections

**Business**
- [ ] Outline devlog #1 ("why we're building an open, pay-once TV stick")

**Done when:** Archive films appear in rows with correct titles and artwork.

---

### Week 7 (Nov 30 – Dec 6): Internet Archive, part 2, plus unified search v1

**Engineering**
- [ ] Archive playback through Media3; artwork fallbacks for items without posters
- [ ] Curated rows (e.g., film noir, classic sci-fi, silent era)
- [ ] Unified search: query all providers in parallel with a per-provider timeout (~3s), merge, de-duplicate by external IDs, rank
- [ ] Search UI that works with the on-screen keyboard; voice input via the platform speech recognizer where available

**Done when:** One search returns merged results from Jellyfin and the Internet Archive.

---

### Week 8 (Dec 7–13): Handoff provider

**Engineering**
- [ ] Get a TMDB API key (non-commercial terms; read them, since a commercial license will be needed before hardware sales)
- [ ] Use TMDB's watch-provider data for the US region (requires JustWatch attribution)
- [ ] Test deep links for Tubi and Kanopy on the Onn: which link formats open the exact title?
- [ ] Declare package visibility (`<queries>`) so the app can detect which streaming apps are installed (required on Android 11+)
- [ ] Fallback when a deep link fails: open the app's search for the title, or the app's home screen
- [ ] Add an About screen with TMDB and JustWatch attribution

**Done when:** Pressing play on a Tubi title in your app opens that title in the Tubi app.

---

### Week 9 (Dec 14–20): Detail pages, settings, onboarding

**Engineering**
- [ ] "Where to watch" on detail pages: every source for a title with labels (Your server, Free, Free with ads, Library card)
- [ ] Settings: add/remove providers, manage Jellyfin servers, region
- [ ] First-run onboarding: choose sources, connect Jellyfin
- [ ] Empty states, error states, offline handling

**Business**
- [ ] Landing page with waitlist signup (static site; privacy-respecting email tool such as Buttondown or self-hosted Listmonk)
- [ ] Draft privacy policy (short, plain language, matching the no-telemetry stance)

**Done when:** A new user can install, onboard, and watch something without help.

---

### Week 10 (Dec 21–27, Christmas): Bug bash (light week)

- [ ] Bug bash on every device; log issues in GitHub
- [ ] Record a 2-minute demo video
- [ ] Publish devlog #1 and the landing page

**Done when:** Landing page is live with the demo video and waitlist.

---

### Week 11 (Dec 28 – Jan 3, New Year): Plan and tooling (light week)

- [ ] Read F-Droid's inclusion policy and build requirements (fully open dependencies; reproducible builds preferred)
- [ ] Set up CI with GitHub Actions: build, unit tests, lint on every push
- [ ] Recruit ~20 alpha testers: friends, Jellyfin forum volunteers, Android TV enthusiasts
- [ ] If the RK3539 stick has arrived: install the app, set it as the default launcher, and note what breaks
- [ ] Review this plan; adjust January

**Done when:** CI is green and you have a list of 20 alpha testers.

---

### Week 12 (Jan 4–10): Private alpha

**Engineering**
- [ ] Crash handling without Google services: local crash log plus a "send report" button that opens a pre-filled GitHub issue or email
- [ ] Check memory and responsiveness on the 2GB device; fix any "app not responding" cases
- [ ] Accessibility basics: text size, contrast, focus visibility
- [ ] Move all UI text into string resources for future translation
- [ ] Ship alpha APK to testers with a short feedback form

**Business**
- [ ] Google Play: if you're registering a **personal** developer account, new personal accounts must run a closed test (currently 12+ testers for 14 days) before publishing; start it now. An **organization** account (needs the D-U-N-S number) avoids this. Check the current Play Console rules.

**Done when:** Alpha is in testers' hands and feedback is arriving.

---

### Week 13 (Jan 11–17): Alpha fixes and quality checklist

**Engineering**
- [ ] Fix alpha feedback; focus and navigation bugs come first
- [ ] Android TV quality checklist: Leanback launcher intent, 320×180 banner, touchscreen declared not required, everything works by D-pad
- [ ] App icon, TV banner, screenshots

**Business**
- [ ] File the trademark (name final)
- [ ] Finalize licenses per D-18; add CONTRIBUTING.md, code of conduct, and CLA tooling (e.g., CLA Assistant)

**Done when:** Testers report no navigation dead ends; licensing files are final.

---

### Week 14 (Jan 18–24): Store submissions

**Engineering**
- [ ] Release signing set up; CI builds signed release APKs
- [ ] Google Play listing (Android TV category); submit for review
- [ ] F-Droid: open a merge request with app metadata (review can take weeks, so submit early)

**Business**
- [ ] Publish privacy policy on the website
- [ ] Set up community space: Matrix room or Discord, plus GitHub Discussions; issue templates

**Done when:** Both store submissions are in review.

---

### Week 15 (Jan 25–31): Launch prep

- [ ] Final demo video; devlog #2 ("what v0.1 can do, and what's next")
- [ ] Draft launch posts tailored to each community: r/jellyfin, r/AndroidTV, r/cordcutters, r/fossdroid, Hacker News ("Show HN"), Jellyfin forum
- [ ] Publish a public roadmap (a short version of `02_roadmap.md`)
- [ ] Prepare a hotfix process: branch, build, release in under 2 hours

**Done when:** Everything for launch week is drafted and scheduled.

---

### Week 16 (Feb 1–7): Public beta v0.1

- [ ] Tag v0.1.0; GitHub release with signed APK and changelog
- [ ] Stagger launch posts over the week (not all on one day)
- [ ] Triage issues daily; ship v0.1.1 hotfix as needed
- [ ] Start the metrics tracker (below)
- [ ] **Gate 0 review** (criteria in `02_roadmap.md`, section 4): write down pass/fail and date

**Done when:** v0.1 is public and the Gate 0 decision is recorded.

---

## Stage 0b: Traction (two-week sprints)

### Weeks 17–18 (Feb 8–21): Stabilize

- [ ] Weekly release rhythm (same day each week)
- [ ] Label "good first issue" tasks for contributors
- [ ] Write the provider SDK guide ("how to write a provider") with an example provider

### Weeks 19–20 (Feb 22 – Mar 7): More sources, start hiring

- [ ] Add handoff support for Hoopla and Pluto TV
- [ ] Write the senior Android developer job post (Compose, ideally TV; Media3 a plus)
- [ ] Source candidates: LinkedIn, Wellfound, Cutshort, referrals, contributors from India in your community
- [ ] Choose a hiring route: employer-of-record service vs contractor (include IP assignment either way)

### Weeks 21–22 (Mar 8–21): Paid trial and OTA

- [ ] Start a 2–3 week paid trial with the top candidate: build the Plex provider against `provider-api`
- [ ] Build the HDHomeRun provider: discover tuners on the local network, read the channel lineup, play live streams. Treat program-guide data as a separate problem to evaluate later
- [ ] Optional: submit an open-source grant application (check current NLnet/NGI calls)

### Weeks 23–24 (Mar 22 – Apr 4): Release v0.3 and evaluate ODM samples

- [ ] Ship v0.3 with OTA live TV
- [ ] Evaluate ODM samples: boot time, UI smoothness, 2-hour 4K playback thermal test, Wi-Fi performance with the HDMI extender
- [ ] Request SDK access formally from the best one or two

### Weeks 25–26 (Apr 5–18): Plex and the hire decision

- [ ] Review and merge the Plex provider; decide on the hire and make an offer if the trial went well
- [ ] Build the two or three features users request most (from the issue tracker)
- [ ] Contact Crowd Supply about a future campaign (they vet projects in advance)

### Weeks 27–28 (Apr 19 – May 2): Launcher-as-home prototype

- [ ] On the generic RK3539 stick, make the app a default HOME launcher: boot behavior, home-button handling, input switching
- [ ] Record a demo: "this is what our hardware will feel like"
- [ ] Devlog about the hardware plan; push the waitlist

### Weeks 29–30 (May 3–16): ODM shortlist and BSP role

- [ ] Shortlist one or two ODMs; draft agreement terms (SDK, firmware freedom, certification reuse, license-inclusive pricing, quality checks)
- [ ] Write the AOSP/BSP engineer job description; begin sourcing (set-top box, automotive, chipset BSP backgrounds)
- [ ] Begin informal angel conversations

### Weeks 31–32 (May 17–30): Real numbers

- [ ] Update the spreadsheet: India loaded cost on Assumptions, actual ODM quotes on Products, realistic volumes
- [ ] Pricing survey to the waitlist (what they'd pay; stick vs Pro interest)
- [ ] Release v0.5

### Weeks 33–34 (May 31 – Jun 13, includes Memorial Day): Reliability

- [ ] Performance and reliability push on the slowest devices
- [ ] Open community translations (Weblate's hosted service is free for open-source projects)
- [ ] Assemble Gate 1 data

### Weeks 35–36 (Jun 14–27): Gate 1 package

- [ ] Metrics report against Gate 1 criteria
- [ ] Written go/no-go memo
- [ ] If "go": detailed Q3 plan (BSP hire offer, ODM contract, licensing sign-ups, campaign prep)

### Week 37 (Jun 28): Gate 1 review

- [ ] Decide and record the outcome with the date

---

## Metrics tracker (fill in monthly)

Measure without breaking the privacy pledge: use store statistics (Google Play Console reports installs and active devices without any code in your app), GitHub release downloads, waitlist counts, and opt-in surveys. If you ever add an in-app counter, make it opt-in and say exactly what it sends.

| Metric | Gate 1 target | Feb | Mar | Apr | May | Jun |
|---|---|---|---|---|---|---|
| Active users (Play active devices + estimate for F-Droid/GitHub) | ≥ 5,000 | | | | | |
| Total installs / downloads | n/a | | | | | |
| Hardware waitlist signups | ≥ 2,000 | | | | | |
| Outside contributors (merged PRs) | ≥ 5 | | | | | |
| GitHub stars | n/a | | | | | |
| Crash-free sessions (Play vitals) | ≥ 99% | | | | | |
| ODM samples evaluated / SDK confirmed | ≥ 1 / yes | | | | | |

---

## Weekly check-in template

Copy this into a dated note every Friday:

```
Week __ (dates):
Done:
Not done (and why):
Blockers:
Metrics change:
Next week's top 3:
```
