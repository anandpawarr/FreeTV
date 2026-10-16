# Open TV Stick: Design Decisions and Rationale

**Status:** Working draft, October 16, 2026
**Owner:** Founder
**Companion files (keep in the same folder):**

- `open_tv_stick_business_model.xlsx`: the financial model ([open it](open_tv_stick_business_model.xlsx))
- `02_roadmap.md`: the multi-year timeline ([open it](02_roadmap.md))
- `03_stage0_weekly_plan.md`: week-by-week plan for Stage 0 ([open it](03_stage0_weekly_plan.md))

This document records what has been decided, what else was considered, and why. Each decision has a "revisit when" line so you know what new information should reopen it. When a decision changes, edit the entry and add a dated note rather than deleting the history.

---

## Contents

1. Product vision and positioning
2. Content and legal stance
3. Hardware decisions
4. Software decisions
5. Licensing and royalties
6. Business model and pricing
7. Funding strategy
8. Team and hiring
9. Risks and open questions
10. Glossary
11. Sources

---

## 1. Product vision and positioning

**The product:** an open-source TV stick and remote, sold once with no subscription and no ads on the home screen. Its core feature is a unified launcher (the "umbrella") that brings free and legal content into one interface: FAST channels, library-card streaming, public-domain films, over-the-air TV, and the user's own media server.

**The pitch:** no subscriptions required, no home-screen ads, no tracking, fully open hardware and software, and updates promised in writing.

**Target buyers:** people underserved by Amazon, Roku, and Google:

- Jellyfin and Plex home-media users
- Privacy-conscious households
- Cord-cutters who mostly watch free content
- People who want sideloading and control back
- Makers and tinkerers
- Institutions such as libraries, schools, and hotels

**Not the target:** mass-market buyers who wait for $20 Prime Day deals. We can't win on price against companies that subsidize hardware with ad revenue, so we don't try.

### Competitive landscape (October 2026)

Full table with sources: **Competitors** tab in the spreadsheet.

- Every major player raised prices in mid-2026, citing component costs (mainly memory). Current list prices include the Fire TV Stick 4K Max at $84.99, the Roku Streaming Stick 4K at $80, the Google TV Streamer at $149.99, and the Apple TV 4K at $199. Walmart's Onn 4K Pro (about $60) is the value leader.
- Sale prices are deep. Amazon has sold the Fire TV Stick 4K Select for $17–20.
- Amazon's new Vega OS sticks only allow apps from the Amazon Appstore, so sideloading is gone. This is a major opening with enthusiasts.
- Incumbents sell hardware near cost and earn on ads, subscription commissions, and data. Our model deliberately rejects home-screen ads and tracking, so our hardware must carry real margin.

---

## 2. Content and legal stance

### D-01: Legal content only; no piracy tooling

- **Decision:** The product and its software only integrate legal, licensed, or public-domain sources. No torrent-streaming of copyrighted content, no "fully loaded" builds, and no add-ons that pull infringing streams.
- **Alternatives considered:** Selling hardware plus an open-source torrent-streaming UI, on the theory that "what users do is their choice."
- **Why:** There's no legal loophole for this. Under *MGM v. Grokster* (2005), a company that designs and distributes a product with intent to foster infringement is liable, and product design and internal documents count as evidence of intent. Precedents are consistent: Popcorn Time and its forks were repeatedly shut down; TickBox TV, which used the same "users choose" argument, ended with a reported $25M judgment; UK sellers of "fully loaded" boxes have been imprisoned. Beyond legal risk, payment processors, marketplaces, and suppliers drop vendors quickly once rights holders complain. Open-source code doesn't shield the company; it makes intent easier to show.
- **Revisit when:** Never, for the piracy question itself.

### D-02: Official APIs and deep links only; no scraping

- **Decision:** Content providers are integrated through official APIs, public data, or deep links into their own apps. Stream URLs from services we haven't partnered with are never scraped or embedded.
- **Alternatives considered:** Extracting streams from free services so everything plays in our own player.
- **Why:** Scraping breaks terms of service, breaks whenever providers change their backends, and turns a legitimate product into a legal target. The code is public, so anything questionable would be visible to everyone.
- **Revisit when:** A provider signs a distribution agreement. Then that provider moves to native playback.

### Content sources (planned)

| Source | Type | Playback mode | Status |
|---|---|---|---|
| Jellyfin servers | User's own media | Native | Stage 0 MVP |
| Internet Archive | Public domain films | Native | Stage 0 MVP |
| Tubi, Kanopy, Hoopla, Pluto TV | Free / library streaming | Handoff (deep link) | Stage 0 MVP (1–2), more later |
| HDHomeRun network tuners | Over-the-air TV | Native / live | Stage 0 traction phase |
| Plex servers | User's own media | Native | Stage 0 traction phase |
| FAST channels via partnerships | Free ad-supported linear TV | Native / live | After distribution deals |
| Own USB ATSC 3.0 tuner | Over-the-air TV | Native / live | Stage 2 hardware |

---

## 3. Hardware decisions

### D-03: Form factor: stick, plus a power dongle that carries the ports

- **Decision:** A small HDMI stick powered over USB-C. The power adapter includes USB-A and Ethernet, with USB data carried over the same USB-C cable (similar to Chromecast's Ethernet adapter). An HDMI extender ships in the box.
- **Alternatives considered:** Ports on the stick itself (too large and hot); a puck/box form factor (better thermals and RF, less "stick").
- **Why:** Keeps the stick small and cool while giving power users wired networking and a port for tuners or drives. The extender moves the Wi-Fi antenna away from the TV's metal and HDMI noise.
- **Revisit when:** Thermal testing fails. A puck is the fallback and is also planned as the "Pro" product.

### D-04: SoC: Rockchip RK3539

- **Decision:** Rockchip RK3539 (quad Cortex-A55 at about 1.9 GHz, Mali-G310 GPU, 4Kp60 AV1/H.265/H.264 decode, HDR10).
- **Alternatives considered:**
  - *Amlogic S905X4 / S905X5M:* the most mature TV chips, with strong video, a well-worn path to Netflix/DRM certification, and CoreELEC (Kodi Linux) support. But documentation is under NDA and Linux runs on old vendor kernels.
  - *Rockchip RK3576 / RK3588:* better mainline Linux progress, but more expensive and power-hungry than a stick can cool, and AV1 still isn't in mainline Linux.
  - *Allwinner H618:* cheapest, but its video pipeline is weaker.
- **Why:** The RK3539 combines Rockchip's more open stance (its datasheet is public) with what a TV stick needs: 4K AV1 decode at budget pricing. It already ships in retail sticks (about $34–45 on AliExpress), which proves the ODM ecosystem exists.
- **Trade-offs accepted:** The chip is brand new. Today its video engine only works through Rockchip's Android BSP, not mainline Linux, so we start on Android (see D-12). The DRM certification path is weaker than Amlogic's.
- **Revisit when:** We can't get the RK3539 SDK from an ODM, the BSP proves unstable, or premium DRM apps (Netflix etc.) become a priority. Amlogic S905X5M is the fallback, and the SoC-agnostic launcher (D-14) keeps switching cheap.

### D-05: Memory and storage: 4GB LPDDR4, 32GB eMMC

- **Decision:** Ship 4GB RAM and 32GB storage.
- **Alternatives considered:** 2GB/16GB (cheaper).
- **Why:** A unified launcher with artwork from many sources is memory-hungry. 2GB causes stutter and app reloads. 32GB leaves room for offline downloads of legal content.
- **Revisit when:** DRAM prices spike further. Memory is the most volatile BOM line in 2026, so get quotes early and often.

### D-06: Wireless: pre-certified Wi-Fi 6 + Bluetooth 5.x module

- **Decision:** Use a pre-certified combo module rather than discrete radio design.
- **Why:** Modular FCC approval avoids certifying the radio ourselves, and Bluetooth qualification can reference the module's existing design.

### D-07: HDMI connector, not a custom or alternative connector

- **Decision:** Standard HDMI with CEC.
- **Alternatives considered:** USB-C/DisplayPort with an adapter, to avoid HDMI licensing.
- **Why rejected:** TVs essentially only accept video over HDMI, so an adapter would still be an HDMI product. DisplayPort has its own patent pool. A converter chip costs $2–5, far more than HDMI's $0.04–0.15 royalty. And adapters rarely pass CEC, which would break "one remote controls everything."
- **Revisit when:** Never, realistically.

### D-08: Remote: Nordic BLE MCU, 2×AAA, IR plus CEC

- **Decision:** nRF52833 or nRF54L15 microcontroller with Zephyr firmware; BLE HID-over-GATT for standard keys plus a custom GATT service for launcher-specific keys; IR LED for TVs without CEC; silicone keypad; 2×AAA batteries; "find my remote" buzzer; optional push-to-talk microphone.
- **Alternatives considered:** ESP32-C3 (higher power use while connected); rechargeable lithium battery (needs a charging circuit, safety certification, and shipping restrictions for little benefit).
- **Why:** Nordic parts have the best battery life for devices that sleep most of the time, and Zephyr lets hobbyists remap buttons.
- **Revisit when:** Customer feedback strongly favors rechargeable.

### D-09: Build path: white-label ODM first, custom board later

- **Decision:** Stage 1 uses an ODM's existing RK3539 stick design with our firmware, remote, and packaging. A custom board comes in Stage 2, after traction and a seed round.
- **Alternatives considered:** Designing our own board from the start.
- **Why:** Eliminates most engineering and tooling costs (NRE), uses an already-proven design, and may let us reuse the ODM's certifications (ask about this explicitly). Shipping sooner with less capital matters more than a custom PCB in year one.
- **Revisit when:** The ODM can't provide SDK access, firmware freedom, or acceptable quality.

### D-10: What "DIY" means

- **Decision:** The stick is sold assembled, with an open, flashable OS and printable case files. The remote is offered as a true DIY kit (hand-solderable, mechanical switches, 3D-printed case, open firmware).
- **Why:** Customers can't solder BGA chips or route LPDDR4 memory. The remote is where DIY genuinely works.

### D-11: Open hardware publication

- **Decision:** Publish our design files (schematics, layout, enclosure, firmware) under CERN-OHL and pursue OSHWA certification for our own designs. Be upfront that the SoC internals, GPU drivers, and Wi-Fi firmware are vendor binaries.
- **Why:** Honest openness is the brand. OSHWA certification covers our design, not chip internals.

---

## 4. Software decisions

### D-12: Operating system: AOSP Android 14 on Rockchip's BSP (for v1)

- **Decision:** Ship AOSP (Android 14, kernel 6.1 class) from Rockchip's SDK, without Google services. Maintain a Linux + Kodi image as a community track for later.
- **Alternatives considered:**
  - *Google TV / Android TV with Play Store:* requires a Google license that isn't realistic for a startup at launch.
  - *Linux + Kodi (LibreELEC) on mainline:* most open, but mainline Linux has no RK3539 video decode support, and even AV1 for the older RK3576 isn't upstream yet.
- **Why:** The BSP is currently the only stack where the RK3539's hardware video decode works.
- **Consequences accepted:** No Play Store (apps come from F-Droid, vendor APKs, or our own curated store); no Widevine DRM (see D-19); some apps that require Google Play Services won't run, so test every app we promise.
- **Revisit when:** Mainline Linux gains RK3539 video support, or a Google partnership becomes realistic.

### D-13: Boot, security, and updates

- **Decision:** U-Boot → OP-TEE → Android Verified Boot; A/B partitions with Android's `update_engine`; signed OTA updates from our own server with staged rollouts (1% → 10% → everyone). **Bootloader ships locked but is unlockable** with a documented procedure.
- **Why:** A/B updates roll back automatically if a release fails to boot, which protects against bricking devices in bulk. An unlockable bootloader is central to the "it's yours" promise and allows community firmware.
- **Kernel maintenance:** We maintain our own kernel fork and backport security fixes, because vendor BSPs tend to stop updating within a few years. The kernel is GPL, so we publish its source.

### D-14: Launcher architecture: provider plugins with three playback modes

- **Decision:** The launcher is an Android HOME app (on our hardware) built around a provider plugin interface. Each provider implements browse, search, metadata, and "play." A provider plays content in one of three ways:
  - **Native:** a legitimately obtained stream plays in our Media3 player (Jellyfin, Internet Archive, OTA, partnered FAST channels).
  - **Handoff:** a deep link opens the exact title in the provider's own app (Tubi, Kanopy, Hoopla, Pluto).
  - **Live:** linear channels through the Android TV Input Framework, so OTA and FAST share one channel guide.
- **Supporting decisions:**
  - A local catalog database merges all providers and matches the same title across sources ("Free on Tubi with ads · Free with your library card on Kanopy").
  - Unified search across every provider is the headline feature.
  - Everything is local-first: no account required, no telemetry by default.
- **Why:** This is how the umbrella works without content deals on day one, while leaving room for native playback as partnerships arrive. The launcher is chip-agnostic, so it protects us if we change SoCs.
- **Known limitation:** Continue-watching only works for native playback, plus handoff apps that publish progress (many won't outside Google TV). Another reason to pursue native partnerships.

### D-15: Metadata sources

- **Decision:** Wikidata (CC0, free for any use) as the base, plus TMDB for rich metadata.
- **Why / caution:** TMDB's API is free for non-commercial use with attribution. The free Stage 0 app likely qualifies, but a commercial license is needed once we sell hardware. TMDB's "where to watch" data comes from JustWatch and requires attribution.
- **Revisit when:** Before Stage 1 hardware sales. Budget for a TMDB commercial license or rely more on Wikidata and provider metadata.

### D-16: Technology stack

| Area | Choice | Alternatives | Why |
|---|---|---|---|
| Language | Kotlin | Java | Modern Android standard |
| TV UI | Jetpack Compose for TV (`androidx.tv`) | Leanback library, React Native | Google's current TV approach; ports directly to our device |
| Playback | Media3 (ExoPlayer) | libVLC, mpv | Uses MediaCodec, so hardware decode; HDR passthrough; frame-rate matching |
| Database | Room | SQLDelight | Standard, well documented |
| Async | Kotlin coroutines + Flow | RxJava | Standard, simpler |
| Networking | OkHttp + kotlinx.serialization | Retrofit + Moshi | Lightweight, no reflection |
| Images | Coil (with aggressive downsampling) | Glide | Compose-native; downsampling protects low-RAM devices |
| Dependency injection | Koin | Hilt | Simpler for a solo developer |

**Hard rule: no Google Play Services dependencies anywhere** (no Firebase, Google sign-in, or Play-based analytics or crash reporting). Our device runs AOSP without Google, and F-Droid rejects apps that need it.

### D-17: Project structure

```
:app                    UI, navigation, home screen
:core:provider-api      plugin interface (published separately, Apache-2.0)
:core:catalog           Room database, cross-source title matching
:core:player            Media3 wrapper, continue-watching
:providers:jellyfin
:providers:archive      Internet Archive
:providers:handoff      deep links into Tubi, Kanopy, etc.
```

- **Rule:** Each provider depends only on `provider-api`, never on the app or another provider.
- **Why:** Clear boundaries make it possible to hand a new developer a self-contained task ("build the Plex provider") and let the community write providers.

### D-18: Software licensing (proposed; confirm before accepting outside contributions)

- **Proposal:** Apache-2.0 for the `provider-api` SDK; GPLv3 for the app; a contributor license agreement (CLA) so the company can offer commercial licenses to B2B customers.
- **Alternatives considered:** Apache-2.0 for everything (friendliest to B2B, but competitors could ship closed forks); GPLv3 for everything (strongest protection, but may deter B2B partners).
- **Why:** Anyone can write providers freely, while the app itself stays open. The CLA preserves the B2B licensing revenue stream.
- **Revisit when:** Before the first external contribution. Relicensing later requires permission from every contributor.

### D-19: No DRM (Widevine) in v1

- **Decision:** Launch without Widevine. Netflix, Disney+, and similar premium apps are out of scope for v1, and we say so clearly at launch.
- **Why:** Widevine requires a Google license and certified hardware. Most of our sources (own media, Internet Archive, OTA, many FAST streams) don't need it. Skipping DRM also lets us skip HDCP (see section 5).
- **Revisit when:** Customer demand makes premium apps a priority. This would likely require an Amlogic-based certified platform.

### D-20: Stage 0 ships as a regular app on existing hardware

- **Decision:** Release the launcher first as a free, open-source app for Onn, Shield, and generic Android TV boxes, before building any hardware.
- **Why:** It proves demand, builds a community and waitlist, and develops the provider integrations for about $50–150K instead of $1M+.
- **Limitation:** On stock Google TV devices our app can't easily replace the home screen, and Amazon's Vega OS sticks can't run Android apps. So Stage 0 is an app users open, not a launcher. Generic AOSP boxes (including RK3539 sticks) do allow choosing a default launcher, which lets us prototype the full Stage 1 experience early.

### D-21: Voice search

- **Decision:** Stage 0 uses the device's standard Android speech input where available. On our hardware, an on-device speech model (Vosk-class) processes voice locally.
- **Revisit when:** Latency tests on the RK3539's A55 cores are done.

### D-22: Privacy

- **Decision:** No telemetry by default. Any analytics are opt-in, with exactly what's collected stated in plain language. Crash reports go through user-initiated reports, not silent upload.

---

## 5. Licensing and royalties

Royalties are generally owed by whoever sells the finished product (us), not the chip vendor. **Ask the ODM for license-inclusive pricing and written indemnity** for anything they say is covered. Verify all terms below directly; pools change terms often. Not legal advice.

| Licensor | Covers | Needed? | Cost (as researched, Oct 2026) |
|---|---|---|---|
| HDMI Licensing Administrator | HDMI port, CEC, spec access | Yes | Historically: $10K/yr + $0.15/unit ($0.05 with logo, $0.04 with logo + HDCP); or under 10K units/yr: $5K/yr + $1/unit admin + royalty |
| DCP LLC (HDCP) | Copy protection over HDMI | Only with DRM; skip in v1 | Annual fee + tiered key fees ($500/yr for up to 100 keys … $5,000/yr up to 100K); no per-unit royalty |
| Via LA (H.264/AVC) | H.264 decode | Yes | Per unit; historically a free tier for the first units each year |
| Access Advance + Via LA (HEVC) | H.265 decode | If HEVC enabled | Tens of cents to about $1/unit; Access Advance discounts devices up to $80 |
| Sisvel (AV1, VP9) | AV1/VP9 decode | Disputed (AOMedia says royalty-free) | Non-display devices: AV1 €0.11 (€0.08 compliant), VP9 €0.08 |
| Via LA (AAC) | AAC audio | Yes, in practice | Per unit. Note: Android's FDK-AAC decoder has a copyright license but no patent license |
| Dolby / Xperi (DTS) | Decoding Dolby/DTS audio | Only if decoding on-device | Avoid by passing audio through to TV/soundbar |
| Bluetooth SIG | Qualification per design | Yes (stick and remote) | March 2026 schedule: $12,000 per design as free Adopter; $8,000 first qualification/yr as Contributing Adopter ($3,500/yr dues) |
| Avanci Broadcast | ATSC 3.0 tuner | Only with tuner add-on | Per unit; fees waived on first 75,000 units for dongle makers who sign within 6 months of first sale |
| Google | Play Store, Widevine | Not for v1 | n/a |

**Cost-saving decisions:**

- Start on the HDMI low-volume tier; the standard tier becomes cheaper above about 5,000 units/yr (the spreadsheet picks the cheaper tier automatically).
- Use the HDMI logo (cuts the royalty by two-thirds; requires compliance testing).
- Skip HDCP in v1.
- Pass Dolby and DTS audio through rather than decoding.
- Consider an optional paid codec unlock (Raspberry Pi once sold MPEG-2/VC-1 licenses this way). Check with pools first, since some count hardware capability rather than enabled features.
- Sell the OTA tuner as a separate product and sign with Avanci early to use the waiver.

**Estimated totals:** fixed costs in year one of roughly $30–50K (HDMI, Bluetooth for two designs, legal review), plus per-unit royalties of roughly $1–3 depending on codecs enabled.

---

## 6. Business model and pricing

### D-23: Pay once, no subscriptions, no home-screen ads

- **Decision:** Customers buy the hardware once and own it. No subscriptions, no ads on the home screen, no features removed by updates.
- **Alternatives considered:** The Roku/Amazon model (subsidized hardware, ad-funded platform); subscription for premium features.
- **Why:** It's the brand promise and the clearest differentiator. It requires hardware to carry real margin and multiple supporting revenue streams.
- **The hard part:** "Forever" means years of security updates paid for by one-time revenue. Plex's lifetime pass tripling in price (to $750) shows what happens when lifetime promises are underpriced.

### Pricing (base case in spreadsheet: Products tab)

| Product | Price | Notes |
|---|---|---|
| Stick + remote + power dongle | $89 | $79 loses about $1/unit at retail; $89 nets about $30 direct, $5 retail |
| Pro puck | $119 | More RAM, gigabit Ethernet; best retail candidate |
| DIY remote kit | $29 | Direct only |
| OTA tuner add-on | $49 | Separate product |
| Accessories (avg) | $19 | Cases, cables, remotes |
| Codec unlock | $4.99 | Optional, one-time |

### D-24: Sell direct first; retail later and mainly for the Pro

- **Why:** Retailers take about 25–35%, which nearly erases the stick's margin. Retail is a marketing channel for this product, not a profit center.

### Supporting revenue streams (compatible with pay-once)

- **Hardware family and accessories:** stick, Pro puck, tuner, remotes, cases.
- **DIY kits:** high margin, best marketing with makers.
- **FAST revenue share:** ads are already inside channels we carry natively through partnerships; our home screen stays ad-free.
- **Disclosed referral fees:** when users subscribe to paid services through the launcher. Always disclosed; never used to rank content.
- **B2B licensing:** hotels, libraries, schools, ISPs, other hardware makers.
- **Optional pay-once extras:** for example, the codec unlock.
- **Community funding:** Open Collective / sponsorships for the open-source OS.

### Commitments (put these in writing)

1. Security updates for at least 5 years from the last sale date.
2. End-of-life pledge: if the company shuts down or ends support, release final firmware with an unlocked bootloader and all source.
3. No home-screen ads and no default telemetry, as policy.
4. No feature removals by update.

### What the model shows (base case at time of writing)

From the **Projection** tab (assumptions: lean team growing from 3 to 6, $160K loaded cost per person, $600K starting cash):

- First profitable year: 2029 (year 3 of sales)
- Lowest cash: about –$355K in 2028, so roughly $1M starting funding is needed, plus inventory deposits not modeled
- Break-even: about 24–29K sticks per year at planned volumes of everything else
- Maintenance reserve ($10/device) fully funds the software team from 2030

These figures move with the inputs. In particular, switching "Loaded cost per staff member" to India-based rates (see section 8) substantially reduces the funding need. Re-read the **Key outputs** block on the Projection tab after any change.

**Note on years:** the model's first sales year is labeled 2027. On the current roadmap, meaningful sales begin around 2028, so read the model's columns as "sales year 1, 2, 3…" (see `02_roadmap.md`).

---

## 7. Funding strategy

### D-25: Stage the company; avoid traditional VC

- **Decision:** Fund each stage with proof from the previous one:
  - **Stage 0, software first:** about $50–150K (largely the founder's time).
  - **Stage 1, white-label hardware:** about $200–400K, mostly from pre-orders.
  - **Stage 2, custom hardware and team:** a $500K–1M seed round on traction.
- **Sources, in order of fit:** pre-orders and crowdfunding (Crowd Supply suits open hardware; Kickstarter has more reach); angel investors from open-source, privacy, and home-media communities; open-source grants for the software (for example NLnet's NGI Zero; check current calls); inventory and revenue-based financing once there's sales history; hardware accelerators.
- **Why not traditional VC:** A pay-once, ~30% margin, privacy-first hardware business is built to be sustainable, not to grow 10× a year. VC pressure would push toward ads or subscriptions, the exact things the brand rejects.
- **Revisit when:** A specific investor's terms and values are aligned.

**Inventory cash warning:** ODMs often require 30–50% deposits before production. A first run of 6,000 sticks at about $42 ties up about $250K before any sale. Pre-orders are the cheapest way to fund this.

---

## 8. Team and hiring

### D-26: Founder writes the Stage 0 code; hire from India as needed

- **Decision:** The founder builds Stage 0. Hire India-based talent as the work outgrows one person.
- **Hiring order:**
  1. Senior Android developer with Compose (ideally TV) experience, around the Stage 0 public release.
  2. AOSP/BSP engineer for Stage 1 (Rockchip SDK bring-up, kernel, OTA). Look for set-top box, automotive, or chipset BSP experience; Bangalore and Hyderabad have deep pools.
  3. Part-time QA tester with a device shelf, once there are real users.
- **Costs (October 2026 guides):** senior Android developers (7–9 years) earn about ₹18–35 LPA; consumer-tech startups pay about ₹28–40 LPA. Through a hiring platform that handles payroll and compliance, seniors cost about $3,500–6,000/month all-in, so roughly $40–70K/yr versus $160K/yr in the US-based model.
- **How to hire:**
  - Use a paid trial project ("build the Plex provider against our interface").
  - Hire experience over cheapness; for a tiny team, one excellent engineer beats two average ones.
  - Use a contract with an IP assignment clause, plus the project CLA. An employer-of-record service (Deel, Remote, etc.) handles Indian employment law properly.
  - Texas–India is about 10.5–11.5 hours apart: work asynchronously with written specs, a project board, recorded demos, and one daily overlap call.
- **Productivity:** AI coding tools such as Claude Code help with scaffolding, provider integrations against documented APIs, and tests.

---

## 9. Risks and open questions

| Risk | Impact | Mitigation |
|---|---|---|
| Can't obtain RK3539 SDK from an ODM | Blocks Stage 1 | Contact 3+ ODMs early; Amlogic S905X5M fallback; launcher is SoC-agnostic |
| DRAM price volatility | Margin squeeze | Quote early and often; consider price adjustments |
| ODM quality or reliability | Returns, reputation | Sample evaluation, small first run, staged quality checks |
| Deep links break or don't open exact titles | Poor handoff experience | Defensive implementation with fallback to the app's search; monitoring |
| Apps refuse to run without Google services | Fewer available apps | Test every app before promising it; publish a compatibility list |
| Sisvel AV1/VP9 claims | Unbudgeted royalties | Legal review; budget the rate as a contingency |
| ATSC 3.0 broadcast encryption | Tuner can't play some channels | Verify before marketing local channels heavily |
| TMDB commercial license cost | Metadata cost at Stage 1 | Budget it; lean on Wikidata |
| Vendor BSP stops receiving updates | Security debt | Own kernel fork; budget maintenance staff |
| Demand lower than hoped | Wasted hardware spend | Gate 1 metrics before any hardware commitment |
| Crowdfunded hardware ships late | Reputation | Buffer in timeline; honest updates |

**Open questions to resolve:**

- Company name and trademark
- Software license (D-18) confirmation
- Legal entity: Texas LLC versus Delaware C-corp (angels often prefer C-corps; consult an attorney)
- Whether the ODM's FCC/CE certifications can be reused for the white-label stick
- TMDB commercial terms and cost
- Which FAST partners to approach first

---

## 10. Glossary

- **AOSP:** Android Open Source Project, Android without Google's apps and services.
- **BSP:** Board support package, the vendor's kernel, drivers, and tools for a chip.
- **CEC:** Consumer Electronics Control, which lets HDMI devices control each other (TV power, volume).
- **CLA:** Contributor license agreement.
- **DTC:** Direct to consumer.
- **FAST:** Free ad-supported streaming TV (Pluto, Tubi channels, etc.).
- **NRE:** Non-recurring engineering, one-time design and tooling costs.
- **ODM:** Original design manufacturer, a factory that designs and builds products others brand.
- **OTA:** Over-the-air (either broadcast TV or software updates, depending on context).
- **SoC:** System on a chip, the main processor.
- **Widevine:** Google's DRM system, required by most premium streaming apps.

---

## 11. Sources (researched October 2026)

- Competitor prices: Ars Technica (Roku price increases, Jul 24, 2026); Android Authority (Google TV Streamer price increase, Sep 1, 2026; Fire TV Stick 4K Select deal); Digital Trends (Fire TV 4K Max and Apple TV increases); Slickdeals (Amazon post-increase list prices, Aug 2026; Vega OS app restrictions); AmericanTV (Oct 6, 2026 prices); AFTVnews (Onn price increases, Plex lifetime pass)
- RK3539: cnx-software.com (Jun 18, 2026); androidtvbox.eu; androidpctv.com
- Rockchip mainline video status: cnx-software.com (Collabora H.264/H.265 for RK3588/RK3576); linux-rockchip mailing list (Jun 2026)
- Amlogic: cnx-software.com (S905X5 and S905X5M); coreelec.org; CoreELEC forum device guide (May 2026)
- Bluetooth SIG fee schedule effective March 1, 2026: bluetooth.com/fee-schedule
- HDCP fees: digital-cp.com/faqs
- HDMI adopter terms: historically published terms (verify with HDMI Licensing Administrator)
- HEVC: accessadvance.com program overviews; StreamTV Insider (low-price device discount)
- AV1/VP9: sisvel.com AV1 licensing program; Streaming Media
- ATSC 3.0: avanci.com/broadcast; ATSC news (75,000-unit waiver)
- India salaries: weekday.works, recrew.ai, secondtalent.com, cutshort.io (2026 guides)
