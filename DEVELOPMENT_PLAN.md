# Find-My-Line — Phased Development Plan

## 1. Project Definition

**Find-My-Line is a bikepacking route planner and route generator.**

Its job is simple:

> Given a start, destination, and riding intent, find practical bikepacking lines that connect them while favoring suitable unpaved riding and avoiding unnecessary pavement, inappropriate terrain, and known access problems.

Find-My-Line is **not a navigation app** and is not intended to replace established navigation tools.

**Find-My-Line finds the line → the rider's preferred navigation tool follows the line.**

The project should concentrate its engineering effort on discovering and evaluating bikepacking routes that conventional routing may not produce.

---

## 2. Product Philosophy

### Complement, don't compete

1. Rider describes where they want to go and what kind of ride they want.
2. Find-My-Line translates that intent into routing preferences.
3. A routing engine generates physically connected candidate routes.
4. Find-My-Line evaluates those candidates for bikepacking suitability.
5. Rider compares alternatives.
6. Selected line is exported to the rider's preferred navigation tool.
7. Completed rides can optionally provide feedback that improves future route generation.

### Permanent architectural boundaries

- The routing engine determines physical connectivity and route geometry.
- The AI interprets human intent and evaluates returned candidates.
- The AI must **never invent route geometry**.
- GPX remains a permanent universal interoperability layer.
- Direct navigation-app integrations are optional convenience layers, never foundational dependencies.
- Every phase must produce something testable and usable before the next phase begins.

---

## 3. Core Architecture

```text
Rider intent
     |
     v
Local Find-My-Line AI
     |
     | structured routing preferences
     v
Routing engine + regional route graph
     |
     | physically connected candidate routes
     v
Find-My-Line route evaluator
     |
     | comparisons / explanations
     v
Rider chooses a line
     |
     v
GPX/FIT/etc. export
     |
     v
Existing navigation app / GPS device
```

The architecture is intentionally modular so the routing engine, map renderer, AI runtime, data providers and backend can be replaced independently.

---

## 4. On-Device AI Strategy

Find-My-Line should target a **small specialized local AI model**, not a general-purpose assistant.

The model's jobs are:

- Understand rider requests.
- Convert natural language into structured routing preferences.
- Interpret route statistics and known route attributes.
- Compare candidate routes.
- Explain tradeoffs.
- Handle revisions such as "less pavement," "less climbing," or "more remote."

The model does not need to contain the road network. Geographic knowledge belongs in the regional offline dataset and routing engine.

The production app should ultimately perform the core planning conversation without an internet connection when the required regional data and model are installed.

Model size must be **measured, not assumed**. The initial research target is roughly **0.5–2 GB for a quantized model**, with a goal of going smaller if a specialized model meets the quality requirement.

The model should produce constrained structured output rather than unrestricted routing instructions. Conceptually:

```json
{
  "surface_preference": "unpaved",
  "pavement_tolerance": 0.15,
  "traffic_tolerance": 0.10,
  "technicality": 0.35,
  "remoteness": 0.85,
  "climbing_tolerance": 0.70,
  "scenic_preference": 0.80
}
```

The exact schema is developed and benchmarked during Phase 4.

A practical Android runtime candidate is llama.cpp, whose Android documentation supports GGUF models and ARM64 Android builds. Gemma 3n is one model family worth benchmarking because it is explicitly designed for mobile/on-device use and offline operation.

---

## 5. Regional Offline Data Strategy

Find-My-Line should centralize data collection and normalization rather than forcing every phone to download raw global datasets.

```text
Central data pipeline
        |
        +-- Region A
        +-- Region B
        +-- Region C
        +-- ...
                |
                v
        Phone downloads selected regions
```

Phones download only the regions the rider needs and retain them for offline planning.

Routine updates should be **incremental/delta updates**, not full redownloads. Regional datasets should be versioned so route generation can be reproduced against a known data version.

---

## 6. Data Sources and Licensing

Potential sources include:

- OpenStreetMap for general geographic/routing data.
- USGS 3DEP for elevation.
- US Forest Service, BLM, NPS, state agencies, DOTs and local GIS where licensing permits.
- Public APIs, downloadable datasets and legally usable community/specialized sources.

Do not bypass access controls, CAPTCHAs, rate limits, authentication or terms of service. Every source adapter must respect the source's legal and technical conditions.

OpenStreetMap-derived data is available through free regional extracts such as Geofabrik. OSM-derived data is under ODbL 1.0 and requires appropriate attribution and licensing treatment for derived databases.

---

## 7. Data Normalization, Provenance and Confidence

The ingestion pipeline should:

1. Collect source data.
2. Normalize schemas.
3. Geographically match corresponding features.
4. Deduplicate where appropriate.
5. Resolve conflicts using explicit rules.
6. Preserve source provenance.
7. Assign confidence and verification dates.
8. Produce a canonical regional routing dataset.

Important attributes should retain, where practical:

- Source
- Source type
- Date obtained
- Date verified
- Confidence
- Official/unofficial status
- Seasonal validity

The system must not manufacture precise facts from weak signals.

---

## 8. Route Intelligence Signals

Useful characteristics may be derived from combinations of:

- OSM road/trail classification
- Surface
- Road class
- Speed limit
- Lane count
- Traffic counts
- Population density
- Road ownership
- Protected-land status
- Terrain
- Forest cover
- Water features
- Scenic designations
- Trail difficulty
- Trail usage where legally available
- Community ride feedback
- Recent condition reports

Derived attributes such as estimated remoteness, traffic exposure, scenic value, bikepacking suitability, technical difficulty and likely hike-a-bike must retain confidence and provenance. These are decision-support signals, not guaranteed truths.

---

## 9. Legal Access Is a Routing Attribute

A mapped path is not automatically legally rideable.

The routing system must distinguish between:

1. Physical existence
2. Legal access
3. Bicycle access
4. Seasonal accessibility
5. Suitability for the selected bike/riding style

Unknown access must not silently become confirmed legal access. Official access information should generally outrank community assumptions when the two conflict.

---

## 10. Routing Engine

Do not build a routing engine from scratch unless testing demonstrates that existing technology cannot meet the requirements.

**BRouter** is the initial routing-engine candidate because it is open source, MIT licensed, OSM-based, elevation-aware, configurable and designed for bicycle routing. The routing engine must remain replaceable.

Graph model:

```text
Regional graph
    |
    +-- nodes = junctions / endpoints
    +-- edges = connected road/trail segments
    +-- attributes = surface, access, elevation, road class, etc.
    |
    v
Routing algorithm
    |
    v
Candidate lines
    |
    v
Find-My-Line evaluator
```

The routing engine solves physical connectivity. Find-My-Line controls and evaluates the bikepacking strategy.

---

## 11. Multiple Candidate Routes

The planner should generate **materially different** candidates rather than cosmetic variations.

Potential strategy profiles:

- Balanced
- Low pavement
- Gravel-focused
- Trail-focused
- Lower climbing
- More direct
- More remote
- More adventurous

The final number and names are determined through testing. Candidates should expose concrete tradeoffs rather than hiding everything behind one opaque score.

---

## 12. Route Analysis

Initial statistics:

- Distance
- Elevation gain/loss
- Pavement percentage
- Unpaved percentage
- Gravel percentage where reliable
- Dirt/forest-road percentage where reliable
- Trail/singletrack percentage where reliable
- Major-road exposure
- Estimated travel time
- Access warnings
- Known closures/restrictions

Later additions may include water, resupply, food, camping, bike shops, cell coverage, bailout points, town access and seasonal hazards when they materially improve planning.

---

## 13. Community Feedback and Intelligence Loop

Community participation should improve route planning, not turn Find-My-Line into a general social network.

After a ride, riders may optionally provide structured feedback about route sections and the overall line, including enjoyment, surface quality, technical difficulty, hike-a-bike, traffic, access problems, closures, mud/flooding/washouts and overall suitability.

Riders control whether route data is private, anonymous or public. Sensitive start/end locations should be protected by default when publishing ride data.

Repeated poor feedback should cause a segment or route characteristic to be **downweighted**, not silently deleted, because conditions change and disagreement is useful information.

Popularity must not be treated as quality. Keep separate signals for enjoyment, suitability, usage, confidence, novelty, remoteness and recent conditions.

---

# 14. Phased Development Strategy

Every phase must produce something **testable and usable** before the next phase begins.

A phase is complete only when its acceptance tests pass. Later phases add capability; they do not require the entire future product to be finished before anything is usable.

Each phase also has a **Resource Readiness** gate. Required resources, sources, license notes and planning costs are documented in `PHASE_RESOURCES.md`.

---

## Phase 0 — Project foundation

**Goal:** Establish repository structure, architecture decisions, test strategy and the first geographic test region.

**Capabilities:**
- Android project foundation.
- Module boundaries.
- Test architecture.
- First geographic test region.
- Source/version conventions.
- Initial representative route cases.
- CI/build/test workflow.

**Required resources:** GitHub public repository, GitHub Actions, Android Studio/SDK/NDK, Kotlin/Gradle and at least one representative Android device. Baseline expected cost: **$0**.

**Acceptance test:** The project can acquire selected source data, identify its version and run initial data-processing/build tests.

**Resource gate:** All required development tools are available; source licenses are recorded; no paid service is required to proceed.

---

## Phase 1 — First usable route generator

**Goal:** Prove the central route-discovery idea without AI or community complexity.

**Capabilities:**
- Start and destination.
- Basic bikepacking preferences.
- One regional dataset.
- Existing routing engine, with BRouter evaluated first.
- Bikepacking routing profile.
- Multiple candidate routes where practical.
- Basic statistics.
- Offline map/route display.
- GPX export.

**Explicitly excluded:** Turn-by-turn navigation, live rerouting, user accounts, social features, on-device AI and direct navigation APIs.

**Primary resources:** Geofabrik/OSM regional data ($0), BRouter ($0), offline map renderer such as Mapsforge ($0), GPX library ($0), Android development toolchain ($0).

**Expected phase cost:** **$0 baseline; approximately $0–$25 optional testing/storage.**

**Acceptance test:** A rider can enter two locations, generate useful bikepacking candidates, compare them, export one as GPX and open/use it in an existing navigation application.

**Resource gate:** One test region is locally available; routing engine builds/runs; map rendering works offline; GPX export is validated against at least two existing applications.

**Phase 1 is the first genuinely usable product.**

---

## Phase 2 — Better route intelligence

**Goal:** Make generated lines substantially better than basic bicycle routing.

**Capabilities:**
- Better surface classification.
- Better road/trail suitability.
- Access and legality handling.
- Better elevation analysis.
- Traffic/road-class penalties.
- More meaningful alternative generation.
- Route tradeoff explanations.
- More detailed statistics.

**Primary resources:** OSM/Geofabrik ($0), BRouter profiles ($0), USGS 3DEP elevation ($0 for public datasets), public USFS/BLM/NPS/state GIS where permitted (generally $0), local processing ($0 initially).

**Expected phase cost:** **$0–$50/month during development**, with $0 the target.

**Acceptance test:** The same request produces materially different candidates with understandable tradeoffs, and real-world testing explains why candidates succeed or fail for a stated riding style.

**Resource gate:** Elevation and at least the most important access/surface datasets for the test region have been acquired, versioned and license-checked.

---

## Phase 3 — Centralized regional data system

**Goal:** Build the data foundation for scalable offline planning.

**Capabilities:**
- Source-specific ingestion adapters.
- OSM normalization.
- Licensed government/public datasets.
- Legally usable community/specialized data.
- Geographic matching and deduplication.
- Provenance and confidence.
- Versioned regional datasets.
- Regional packaging.
- Incremental/delta updates.

**Primary resources:** Geofabrik daily regional extracts ($0), government/open GIS ($0 where available), GitHub Actions ($0 for standard runners in public repositories), object storage such as Cloudflare R2 or Backblaze B2.

**Storage planning:** R2 includes 10 GB-month standard storage with free egress; published standard storage is $0.015/GB-month after the free tier. Backblaze B2 advertises the first 10 GB free and published pay-as-you-go storage starting at $6.95/TB/month.

**Expected phase cost:** **$0–$20/month initially.**

**Acceptance test:** A phone can install a selected region, plan within it without a network connection and update it incrementally without redownloading the complete region.

**Resource gate:** At least one complete regional package can be built, hosted, installed, checksummed, updated by delta and rolled back to a prior version.

---

## Phase 4 — Offline on-device AI

**Goal:** Add conversational route planning without requiring a network connection.

**Capabilities:**
- Small specialized local model.
- Natural-language ride requests.
- Natural-language preference changes.
- Structured preference output.
- Candidate-route interpretation.
- Conversational route comparison.
- Offline operation with installed regional data.

**AI boundary:** The AI translates and evaluates. It does not invent route geometry.

**Model benchmark:** Measure preference extraction accuracy, constraint consistency, ambiguity handling, resistance to hallucinated route facts, candidate comparison accuracy, model size, memory use, latency and battery impact. Test multiple small models and keep the smallest model that meets the quality threshold.

**Primary resources:** llama.cpp Android runtime ($0), candidate small models such as Gemma 3n ($0 where license/terms permit), project-created benchmark corpus ($0), representative Android devices ($0 initially).

**Expected phase cost:** **$0 required for production; approximately $0–$50/month optional model-testing compute.**

**Acceptance test:** A rider can say, for example, "Get me there with as little pavement as reasonably possible. I don't mind climbing, but I don't want busy roads." The offline model converts that into valid structured preferences, the routing engine generates candidates and the model accurately explains their tradeoffs.

**Resource gate:** A candidate model runs locally with airplane mode enabled, produces schema-valid output, meets the agreed benchmark threshold and does not become a cloud inference dependency.

---

## Phase 5 — Community route intelligence

**Goal:** Turn completed rides into useful feedback for future route generation without becoming a general social network.

**Capabilities:**
- Optional post-ride feedback.
- GPX/FIT import where available.
- Section-level feedback.
- Condition reports.
- Route reviews.
- Privacy controls.
- Anonymous/public contribution options.
- Time-decayed condition information.
- Feedback incorporated into route evaluation.

**Primary resources:** GPX/FIT processing libraries ($0), backend such as Supabase ($0 initial Free tier), object storage such as R2/B2 ($0 initially), privacy/consent architecture ($0).

**Supabase planning:** Current Free tier is $0/month with 500 MB database, 1 GB file storage and 5 GB egress; Pro starts at $25/month. Use this only if its current limits and terms remain appropriate when the phase begins.

**Expected phase cost:** **$0–$25/month during early testing.**

**Acceptance test:** A completed ride can be imported or submitted, sections can be evaluated and repeated evidence measurably influences future route selection while preserving uncertainty and provenance.

**Resource gate:** Privacy controls, data retention rules, source attribution and a safe way to delete/revoke user contributions are implemented before public collection begins.

---

## Phase 6 — Expanded exports and interoperability

**Goal:** Reduce friction between route discovery and existing navigation tools.

**Capabilities:**
- KML/KMZ.
- GeoJSON.
- TCX.
- FIT where practical.
- Android share workflows.
- Direct handoff to compatible navigation applications.
- Direct track import where supported.

**Strategic rule:** Interoperability is additive. Portable GPX/FIT-style export remains available even when direct integrations exist.

**Primary resources:** Open route formats ($0), Android share/intents ($0), official vendor APIs/SDKs where available (usually $0 to access; terms vary), documentation/testing accounts.

**Expected phase cost:** **$0–$25/month**, excluding any vendor-specific commercial agreement.

**Acceptance test:** A rider can send a generated line into supported navigation tools with substantially less manual work while Find-My-Line remains independently useful.

**Resource gate:** Each integration has been checked against the provider's current API/terms; no scraping, bypassing or unofficial authentication is required.

---

## Phase 7 — Advanced route discovery

**Goal:** Improve discovery of lines riders would not normally connect manually.

**Potential capabilities:**
- Detect promising unconnected corridors.
- Compare against known/public route networks.
- Identify likely new connections.
- Detect likely hike-a-bike sections.
- Better remoteness/scenic estimates.
- Resupply and bailout intelligence.
- Improved condition prediction.
- Route novelty analysis.
- Advanced community-derived suitability signals.

**Primary resources:** Mature regional graph/data from Phase 3, ride history from Phase 5, public GIS layers and additional processing compute.

**Expected phase cost:** **$0–$100/month initially**, with paid compute only when measured workload requires it.

**Acceptance test:** Real riders can identify useful route connections not obvious from existing published route collections, and field testing validates a meaningful proportion of discoveries.

**Resource gate:** Sufficient route/ride history exists to distinguish genuine discoveries from merely unfamiliar existing roads.

---

## Phase 8 — Mature ecosystem integrations

**Goal:** Make Find-My-Line a route-discovery layer that works naturally with the broader navigation ecosystem.

Potential integrations may include navigation apps, GPS computers, route-management services and other tools where APIs and agreements permit.

**Primary resources:** Official vendor APIs/SDKs, partnership/contact channels, production backend and monitoring.

**Expected phase cost:** **$0–$150/month initially**, excluding legal/business expenses or vendor-specific commercial agreements.

**Acceptance test:** Supported integrations reduce friction without making Find-My-Line dependent on any single navigation provider.

**Resource gate:** Every integration is independently replaceable and GPX remains available as the universal fallback.

---

# 15. Resource Readiness Policy

Before starting any phase, create a short checklist:

1. Required software installed/available.
2. Required datasets identified and license checked.
3. Required accounts created only where necessary.
4. API keys/secrets stored securely and never committed to Git.
5. Estimated monthly cost recorded.
6. A free/open-source fallback identified for every paid dependency.
7. Data provenance and attribution requirements recorded.
8. Acceptance tests understood.
9. Rollback/recovery plan exists for any destructive data migration.

No paid service should be introduced simply because it is convenient. The default question is:

> **Can Find-My-Line prove this capability with an open/free resource first?**

The detailed source links, pricing notes and procurement checklist for every phase live in **`PHASE_RESOURCES.md`**.

---

# 16. MVP Scope

The first MVP is **Phase 1**, not the entire future product.

It must:

1. Accept a start point.
2. Accept a destination.
3. Allow basic bikepacking preferences.
4. Generate multiple useful candidates where possible.
5. Favor suitable unpaved riding.
6. Avoid known inappropriate/illegal segments where data permits.
7. Calculate basic statistics.
8. Display candidates.
9. Export GPX.
10. Work without becoming a navigation app.

Everything else is earned through later phases and testing.

---

# 17. Testing Philosophy

Development should proceed from real-world tests rather than feature accumulation.

Each phase should have unit tests where appropriate, data validation tests, representative route cases, failure cases, acceptance tests and real-world riding validation whenever physical route behavior is involved.

A route that looks excellent numerically but is miserable, illegal, closed, impassable or inappropriate for the selected bike is a routing failure.

Maintain a growing regression suite of known-good and known-bad routes so improvements do not silently break previously solved problems.

---

# 18. Technical Principles

### Open source first
Prefer open-source components with active development and compatible licenses.

### Modular architecture
Keep data ingestion, normalization, routing, AI interpretation, route evaluation, community intelligence and export separate.

### Replaceable routing engine
Do not couple the application permanently to BRouter or any other single engine.

### Small local AI
Use the smallest model that reliably performs the specialized Find-My-Line language/evaluation tasks.

### Explainable routing
Prefer concrete explanations such as:

> "This option is 82% unpaved, avoids the highway, and adds 11 miles compared with the direct option."

### Don't hide uncertainty
Unknown access, surface, condition or legality should be represented as uncertainty rather than false certainty.

### Portable routes
GPX and other standard formats preserve the rider's freedom to choose navigation software.

### Incremental delivery
Every phase should leave behind a working capability that can be tested in the real world.

---

# 19. Success Criteria

The project succeeds if a rider can enter two places and discover a practical bikepacking line they might not have connected manually.

The long-term loop is:

**Rider intent → local AI → routing engine → candidate lines → route intelligence → rider chooses → existing navigation tool follows → optional ride data improves Find-My-Line.**

---

# 20. Guiding Principle

> **Find-My-Line should do one thing exceptionally well: find bikepacking routes worth riding, then get out of the rider's way and let the rider's preferred navigation tool handle the ride.**
