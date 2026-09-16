# Find-My-Line — Phase Resources, Sources, and Cost Plan

This document is the procurement/checklist companion to `DEVELOPMENT_PLAN.md`. Before starting a phase, review its required resources and either have them available or explicitly approve an alternative.

## Cost conventions

- **$0** means a usable free/open-source option is available; it does not mean unlimited commercial usage.
- **Estimated** costs are planning numbers, not quotes. Re-check the linked vendor/source before committing money.
- Prefer free/open-source resources until testing demonstrates that a paid resource materially improves the product.
- Every external dataset must be checked for license, attribution, redistribution, update, and commercial-use requirements before inclusion.

---

## Phase 0 — Project foundation

### Required resources

| Resource | Purpose | Source | Expected cost | Notes |
|---|---|---|---:|---|
| GitHub public repository | Source control, issues, project automation | https://github.com/pistolwhip-justin/find-my-line | $0 | Public repositories can use standard GitHub-hosted Actions runners free of charge. |
| GitHub Actions | CI/build/test automation | https://docs.github.com/en/actions/concepts/billing-and-usage | $0 for public-repo standard runners | GitHub documents standard hosted runner use as free for public repositories. |
| Android Studio / Android SDK / NDK | Android development and native routing/AI builds | https://developer.android.com/studio | $0 | Development software; hardware is separate. |
| Kotlin/Gradle | Android application build system | https://kotlinlang.org/ | $0 | Open-source tooling. |
| Test Android phone | First real-device validation | Existing compatible Android phone | $0 incremental | Use a representative ARM64 phone; additional test devices can be added later. |

### Optional

- A second Android device for compatibility testing: **variable, approximately $50–$300+ used/new**, depending on target hardware.
- GitHub Pro/Codespaces if the project later needs additional private resources: **not required for Phase 0**.

### Phase budget target
**$0 required.**

---

## Phase 1 — First usable route generator

### Required resources

| Resource | Purpose | Source | Expected cost | Notes |
|---|---|---|---:|---|
| OpenStreetMap regional data | Base road/trail graph and attributes | https://www.geofabrik.de/data/download.html | $0 | Geofabrik provides free regional OSM extracts; OSM-derived data is ODbL 1.0 and requires attribution. |
| BRouter | Initial offline bicycle routing engine | https://github.com/abrensch/brouter | $0 | Open source, MIT licensed, Android/Java, elevation-aware and configurable. |
| Offline map renderer | Display the regional map and generated lines | https://github.com/mapsforge/mapsforge | $0 | Mapsforge is open source and supports offline OSM-based vector maps and hillshading. |
| GPX implementation | Export standard route files | Open-source GPX/XML implementation selected during Phase 1 | $0 | Prefer a permissively licensed library compatible with the app. |

### Optional alternatives to evaluate

- MapLibre Native Android: **$0 library cost**, BSD-2-Clause; useful if we decide to build around a vector-tile architecture instead of Mapsforge. Source: https://github.com/maplibre/maplibre-native
- GraphHopper: **$0 open-source engine**, but its current project documentation says official offline mobile routing is no longer supported, so it should be treated as an alternative evaluation rather than the default Phase 1 choice. Source: https://github.com/graphhopper/graphhopper

### Data/hosting cost

For the first region, keep the dataset local during development if practical: **$0**. Do not introduce a cloud bill merely to prove the routing concept.

### Phase budget target
**$0–$25** for optional testing/storage conveniences; **$0 is the intended baseline.**

### Gate before Phase 2
A real rider must be able to enter start/destination, generate useful alternatives, inspect basic statistics, export GPX, and use the GPX in an existing navigation app.

---

## Phase 2 — Better route intelligence

### Required resources

| Resource | Purpose | Source | Expected cost | Notes |
|---|---|---|---:|---|
| OSM regional data | Surface, access, road class, trail and other attributes | https://www.geofabrik.de/data/download.html | $0 | Free extracts subject to ODbL attribution/share-alike requirements for derived databases. |
| BRouter profiles/source | Experiment with route costs and bikepacking preferences | https://github.com/abrensch/brouter | $0 | Custom profiles can express surface, road class, elevation and other routing costs. |
| Elevation data | Elevation gain/loss and terrain analysis | https://www.usgs.gov/3d-elevation-program | $0 for public USGS 3DEP data | Use an appropriate resolution for the region and processing budget. |
| Government GIS datasets | Access/ownership/protected-land/road information | USFS, BLM, NPS, state/local GIS portals | Usually $0 where public/open | Each dataset gets its own license/provenance review. |

### Useful sources

- USGS 3DEP: **$0** for publicly available datasets.
- USFS/BLM/NPS GIS: generally **$0** for public downloadable data, subject to each dataset's terms.
- OpenTopography: free registered access exists for several datasets/API uses, but non-academic API limits apply; commercial integration may require an Enterprise key. Do not make it a core dependency without reviewing terms. Source: https://opentopography.org/developers

### Phase budget target
**$0–$50/month** during development. The target remains $0 using public datasets and local processing.

### Gate before Phase 3
Real-world tests show that route quality has improved because of better surface, access, elevation, road-class, and suitability intelligence—not merely because the route scorer changed numbers.

---

## Phase 3 — Centralized regional data system

### Required resources

| Resource | Purpose | Source | Expected cost | Notes |
|---|---|---|---:|---|
| OSM daily regional extracts | Central ingestion pipeline | https://download.geofabrik.de/ | $0 | Geofabrik's free extracts are normally updated daily. |
| Government/open GIS feeds | Enrich routing attributes | USGS/USFS/BLM/NPS/state/local portals | Usually $0 | License review required source-by-source. |
| Processing compute | Normalize, match, dedupe, build regional packages | GitHub Actions or self-hosted compute | $0 initially | Public GitHub Actions standard runners are free; heavy processing may require paid compute later. |
| Object storage | Host regional packages and deltas | Cloudflare R2 or Backblaze B2 | $0 initially at small scale | R2 includes 10 GB-month free standard storage and free egress; B2 advertises first 10 GB free and $6.95/TB-month thereafter. |
| Dataset manifest/version system | Incremental updates and reproducibility | Git + signed/versioned manifests | $0 | Store checksums, source versions, dates, schema versions and delta relationships. |

### Storage options

**Cloudflare R2:** 10 GB-month standard storage, 1M Class A operations and 10M Class B operations are included monthly; standard storage is $0.015/GB-month after that and egress is free. Source: https://developers.cloudflare.com/r2/pricing/

**Backblaze B2:** first 10 GB storage is free; published pay-as-you-go storage starts at $6.95/TB/month, with free egress up to 3× average stored data. Source: https://www.backblaze.com/cloud-storage/pricing

### Phase budget target
**$0–$20/month initially.** Expect this to rise only as the number/size of regional packages and downloads grows.

### Gate before Phase 4
A selected region can be installed on an Android device, used offline, and updated by downloading only changed data rather than replacing the entire region.

---

## Phase 4 — Offline on-device AI

### Required resources

| Resource | Purpose | Source | Expected cost | Notes |
|---|---|---|---:|---|
| Small open/on-device model candidates | Natural-language preference translation and route explanation | Google Gemma 3n or other compatible open models | $0 model license/download where terms permit | Gemma 3n is explicitly designed for local phone/tablet/laptop use and offline operation. |
| llama.cpp Android runtime | Run GGUF models locally | https://github.com/ggml-org/llama.cpp | $0 | Supports Android, GGUF model loading and ARM64 builds. |
| Model evaluation dataset | Benchmark preference extraction and route explanation | Project-created test corpus | $0 | This is a project asset and becomes a regression suite. |
| Representative Android devices | Memory/latency/battery testing | Existing phone + additional representative devices as needed | $0 initially | Model must be tested beyond one flagship phone before declaring broad compatibility. |

### Model candidates

Start with multiple candidates rather than committing to one model. Gemma 3n is one candidate because Google describes it as mobile-first, memory-efficient and offline-ready. Source: https://deepmind.google/models/gemma/gemma-3n/

llama.cpp is a practical runtime candidate because its Android documentation supports loading GGUF models from app-private storage and building for Android ARM64. Source: https://github.com/ggml-org/llama.cpp/blob/master/docs/android.md

### Important cost rule

**Do not pay for an inference API for the production offline planner.** Cloud inference can be used temporarily for benchmarking if useful, but the production architecture should not depend on it.

### Phase budget target
**$0 required for the production architecture.** Optional model-testing compute: **$0–$50/month** initially if cloud benchmarking becomes useful.

### Gate before Phase 5
The smallest tested model that meets the agreed quality threshold runs locally, with no network connection, translates rider language into valid structured constraints, and accurately explains route candidates without inventing route facts.

---

## Phase 5 — Community route intelligence

### Required resources

| Resource | Purpose | Source | Expected cost | Notes |
|---|---|---|---:|---|
| GPX/FIT import pipeline | Validate completed rides against planned lines | Open standard formats + selected open libraries | $0 | Start with GPX; add FIT when practical. |
| Database/backend | Store optional accounts, rides, reviews, segment feedback and provenance | Supabase or equivalent | $0 initially | Supabase Free currently includes 500 MB database, 1 GB file storage and 5 GB egress; projects pause after 1 week of inactivity. |
| Object storage | User-submitted tracks/attachments | Cloudflare R2 or Backblaze B2 | $0 initially at small scale | Keep raw tracks separate from normalized route intelligence. |
| Privacy model | Private/anonymous/public controls | Project design | $0 | Treat precise start/end locations as sensitive by default. |

### Backend option
Supabase Free is currently $0/month with 500 MB database, 1 GB file storage and 5 GB egress; Pro starts at $25/month. Source: https://supabase.com/pricing

### Phase budget target
**$0–$25/month** during early community testing.

### Gate before Phase 6
Real completed rides can be imported/submitted, section-level feedback is captured, privacy choices work, and repeated evidence changes route evaluation without pretending uncertain reports are facts.

---

## Phase 6 — Expanded exports and interoperability

### Required resources

| Resource | Purpose | Source | Expected cost | Notes |
|---|---|---|---:|---|
| GPX | Universal baseline export/import | Open standard | $0 | Permanent core interoperability layer. |
| KML/KMZ | Additional map/GIS compatibility | Open standard | $0 | Implement only when user demand justifies it. |
| GeoJSON | GIS/web interoperability | RFC/open standard | $0 | Useful for advanced users and data exchange. |
| TCX/FIT | GPS-device interoperability | Garmin/open ecosystem specifications and compatible libraries | $0 initially | Licensing/specification review required before implementation. |
| Android intents/share | Low-friction handoff | Android platform | $0 | Implement only for applications with supported documented interfaces. |
| App-specific APIs | Direct integrations | Official vendor documentation | Usually $0 to access; terms vary | Do not build unofficial scraping/bypass mechanisms. |

### Phase budget target
**$0–$25/month.** Direct vendor/API costs, if any, should be evaluated individually before implementation.

### Gate before Phase 7
A rider can move a generated line into supported navigation tools with substantially less friction while retaining a portable GPX export independent of any one provider.

---

## Phase 7 — Advanced route discovery

### Required resources

| Resource | Purpose | Expected cost | Notes |
|---|---|---:|---|
| Mature regional graph/data | Identify candidate connections and anomalies | Existing Phase 3 infrastructure | $0 incremental initially | More complete data improves discovery quality. |
| Community ride history | Validate novelty and actual rideability | Existing Phase 5 infrastructure | $0 incremental initially | Requires appropriate privacy/consent. |
| Additional GIS layers | Terrain, land use, protected areas, scenic signals | Usually $0 from public sources | Source-specific licensing applies. |
| Larger processing jobs | Corridor analysis and route discovery | GitHub/self-hosted/cloud | $0–$100+/month depending on scale | Start locally or with free public-repo Actions where practical. |

### Phase budget target
**$0–$100/month initially**, with paid compute only when measured workload requires it.

### Gate before Phase 8
Field testing demonstrates that the discovery system produces genuinely useful connections that riders would not routinely find through ordinary route libraries.

---

## Phase 8 — Mature ecosystem integrations

### Required resources

| Resource | Purpose | Expected cost | Notes |
|---|---|---:|---|
| Official vendor APIs/SDKs | Navigation/GPS integrations | Usually $0 to obtain; commercial terms vary | Evaluate one integration at a time. |
| Partnership/contact channels | Agreements and technical coordination | $0 | Business/legal effort rather than infrastructure cost. |
| Production backend | Integration metadata, diagnostics, user controls | $0–$100+/month initially | Scale only with measured usage. |
| Monitoring/analytics | Integration reliability | $0–$50/month initially | Privacy-preserving analytics preferred. |

### Phase budget target
**$0–$150/month initially**, excluding legal/business expenses or vendor-specific commercial agreements.

### Gate
Integrations are additive conveniences. Find-My-Line remains useful, portable and independently operable if any one partner changes or disappears.

---

# Procurement rule for every phase

Before starting a phase, create a short **Resource Readiness Checklist**:

1. Required software installed/available.
2. Required datasets identified and license checked.
3. Required accounts created only where necessary.
4. API keys/secrets stored securely, never committed to Git.
5. Estimated monthly cost recorded.
6. A free/open-source fallback identified for every paid dependency.
7. Exit criteria and acceptance tests understood.

No paid service should be introduced simply because it is convenient. The default question is: **Can Find-My-Line prove this capability with an open/free resource first?**
