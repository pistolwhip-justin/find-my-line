# Find-My-Line — Development Plan

## 1. Project Definition

**Find-My-Line is a bikepacking route planner.**

Its purpose is simple:

> Given a starting point and destination, find rideable routes that connect them while favoring suitable unpaved riding and minimizing unnecessary pavement.

Find-My-Line is **not intended to replace navigation apps, GPS computers, mapping apps, or established route libraries.** It is a route-generation/planning tool that complements them.

The user should be able to generate a route, inspect it, and export it to the navigation system they already prefer.

The project should focus its development effort on one problem: **finding useful bikepacking lines that ordinary routing tools may not produce.**

---

## 2. Product Philosophy

### Complement, don't compete

Find-My-Line should work alongside existing tools rather than attempting to become an all-in-one navigation platform.

Examples of the intended workflow:

1. Open Find-My-Line.
2. Select a start and destination.
3. Choose riding preferences.
4. Generate several route possibilities.
5. Inspect the routes and their characteristics.
6. Export the selected route.
7. Open/use that route in the rider's preferred navigation app or GPS device.

Find-My-Line does not need to provide turn-by-turn navigation to deliver its core value.

### First-class route export

Export is a core feature, not an afterthought.

Initial target formats:

- GPX
- KML/KMZ
- GeoJSON
- TCX
- FIT where practical

The exact initial export set can be reduced for the first MVP if implementation complexity becomes excessive. GPX should be treated as the minimum required export format.

---

## 3. Core Problem

Normal point-to-point routing generally optimizes for factors such as distance, travel time, road hierarchy, or conventional bicycle suitability.

Find-My-Line should instead optimize for **bikepacking suitability**.

The planner should be able to favor:

- Gravel roads
- Forest roads
- Dirt roads
- Doubletrack
- Suitable singletrack
- Bike-legal trails
- Low-traffic roads used as connectors
- Existing bicycle routes when useful

And penalize or avoid, where practical:

- Major paved roads
- High-traffic roads
- Unsuitable trails
- Hiking-only paths
- Motorized-only routes
- Illegal bicycle access
- Known closed routes
- Excessive pavement used merely as a shortcut

The objective is not simply "maximum trail."

A route made almost entirely of technically difficult or inappropriate singletrack may be a worse bikepacking route than one using excellent gravel and forest roads. The routing model therefore needs to evaluate **surface + access + suitability + connectivity**, not just whether something is labeled a trail.

---

## 4. What Makes Find-My-Line Different

The differentiating feature is the **route-generation logic**, not ownership of a map or navigation ecosystem.

The planner should combine available geographic and trail information and search for connections that a rider might not otherwise discover.

The intended result is a route that can be described with concrete statistics such as:

- Total distance
- Elevation gain/loss
- Percentage paved
- Percentage unpaved
- Gravel percentage
- Dirt/forest-road percentage
- Singletrack percentage
- Estimated riding time
- Significant climbing
- Steep sections
- Technical sections
- Access/legality warnings
- Known closures or seasonal restrictions
- Resupply/POI information where available

Do not claim that a generated route is literally "the first" or "never ridden" unless that can actually be established. Prefer language such as **generated**, **unpublished**, **newly connected**, or **not found in the project's known route sources** when appropriate.

---

## 5. Route Generation Model

The planner should use a weighted multi-objective routing model rather than a single hard-coded preference.

Conceptually:

```text
route cost =
    distance cost
  + pavement cost
  + traffic/road-class cost
  + unsuitable-surface cost
  + access-risk cost
  + elevation cost
  + technical-difficulty cost
  + other user-selected penalties
```

The relative weights should be adjustable.

The user should eventually be able to express preferences such as:

- More unpaved
- Less pavement
- More gravel
- More trail
- Less technical
- Less climbing
- More remote
- Faster/easier
- More adventurous

These are routing preferences, not separate products.

---

## 6. Multiple Route Results

The planner should generate **multiple materially different route candidates**, rather than returning only one route.

Examples:

- Low-pavement route
- Gravel-focused route
- Trail-focused route
- Lower-climbing route
- More direct route
- More remote route
- Balanced route

The actual names and number of route variants should be determined during implementation and testing.

The important requirement is that alternatives should be meaningfully different, not minor variations of the same route.

---

## 7. Data Strategy

Find-My-Line should not assume that one map database contains everything needed for good bikepacking routing.

Potential data sources include:

### OpenStreetMap

Primary general-purpose geographic/routing data source.

Useful information includes:

- Roads
- Trails
- Surface
- Access
- Bicycle restrictions
- Route relations
- Track classifications
- Trail difficulty tags
- Points of interest

### Government/public datasets

Where available, incorporate authoritative datasets such as:

- USGS
- US Forest Service
- Bureau of Land Management
- National Park Service
- State land-management agencies
- County/local GIS

These can provide useful information about trails, roads, land ownership, access, closures, and other geographic features.

### Community/specialized datasets

Potential future sources include community trail databases and route collections.

Licensing and redistribution rights must be checked for every external source before incorporating its data into the project.

---

## 8. Data Provenance and Conflict Handling

Different sources will disagree.

The project should not simply merge everything together and assume every record is equally trustworthy.

Each important piece of route information should retain provenance where practical:

- Source
- Source type
- Date obtained
- Date verified
- Confidence
- Official/unofficial status

When sources conflict, the system should resolve individual attributes according to an explicit hierarchy.

For example:

- Official land-manager access information should take precedence over community assumptions about legal access.
- Elevation can be calculated consistently from a selected elevation dataset.
- OSM can provide detailed road/trail geometry where authoritative trail datasets are incomplete.

The underlying source observations should remain distinguishable even when the routing system creates a canonical representation.

---

## 9. Legal Access Is a Routing Attribute

A mapped path is not automatically a legally rideable path.

The routing system must distinguish between:

1. Physical existence
2. Legal access
3. Bicycle access
4. Seasonal accessibility
5. Suitability for the selected bike/riding style

Where access information is unknown, the system should not silently treat the segment as confirmed legal.

Warnings should be presented when the planner cannot establish appropriate access.

---

## 10. Routing Engine

Do not build a routing engine from scratch unless there is a demonstrated need.

Existing open-source routing technology should be evaluated first.

**BRouter** is an especially relevant candidate because it is an open-source OSM-based cycling router with configurable profiles, elevation awareness, alternative routing, and GPX/KML/GeoJSON output capabilities. Its profile system can be adapted to change how surfaces, road classes, elevation, and other characteristics affect routing.

The project should evaluate BRouter as a routing-engine foundation rather than assuming a custom graph engine is necessary from day one.

Potential architecture:

```text
Find-My-Line
     |
     +-- Data collection
     |
     +-- Data normalization
     |
     +-- Access / suitability intelligence
     |
     +-- Bikepacking routing profile
     |
     +-- Alternative-route generation
     |
     +-- Route analysis
     |
     +-- Export
     |
     +-- Existing navigation app / GPS device
```

The routing engine should remain replaceable if a better implementation is discovered later.

---

## 11. Route Analysis

Every generated route should be analyzed after routing.

At minimum:

- Distance
- Elevation gain
- Elevation loss
- Surface breakdown
- Pavement percentage
- Unpaved percentage
- Trail percentage
- Major-road exposure
- Estimated travel time
- Access warnings

Future analysis may include:

- Water availability
- Resupply opportunities
- Camping
- Bike shops
- Food
- Cell coverage
- Bailout points
- Town access
- Seasonal hazards

These should be added only when they materially improve route planning.

---

## 12. User Interface — MVP

The MVP should stay deliberately small.

### Start/destination

The user selects:

- Start
- Destination

Optional later support:

- Waypoints
- Avoid areas
- Required areas

### Riding preferences

The user controls the characteristics they care about.

Initial controls should be simple enough to understand without knowing routing-engine terminology.

Potential controls:

- Pavement ↔ unpaved
- Gravel ↔ trail
- Easy ↔ technical
- Low climbing ↔ climbing accepted
- Direct ↔ exploratory

### Results

Display multiple generated routes with clear statistics.

The user can select a route and inspect it before export.

### Export

Provide a straightforward export/share action that lets the rider send the route to another application or save the file.

---

## 13. Navigation Is Intentionally Out of Scope

The following are **not part of the core Find-My-Line product**:

- Turn-by-turn navigation
- Voice navigation
- Live rerouting
- Traffic navigation
- Full GPS computer replacement
- Continuous background navigation
- Building a competing navigation app
- Maintaining a proprietary global basemap
- Recreating features already handled well by established navigation apps

A future integration may make sending a generated route directly to another service easier, but Find-My-Line should remain useful without owning the navigation experience.

---

## 14. MVP Scope

The first working version should prove only the central concept.

### MVP must:

1. Accept a start point.
2. Accept a destination.
3. Allow basic bikepacking routing preferences.
4. Generate more than one route where possible.
5. Prefer appropriate unpaved riding according to those preferences.
6. Avoid known inappropriate/illegal segments where data allows.
7. Calculate basic route statistics.
8. Display the route candidates.
9. Export at least GPX.
10. Work as a useful planning tool without requiring Find-My-Line to perform navigation.

### MVP should not require:

- User accounts
- Social networking
- Route publishing
- Turn-by-turn navigation
- Live tracking
- Native integrations with every navigation provider
- A massive proprietary trail database
- A custom routing engine

---

## 15. Post-MVP Development

After the core route-generation concept works:

### Phase 2 — Better routing

- More routing parameters
- Better alternative generation
- Better surface classification
- Improved trail suitability
- Better access handling
- More reliable elevation analysis
- Route comparison tools

### Phase 3 — Better data

- Additional authoritative datasets
- More regions/countries
- Closure and seasonal information
- Better POI data
- Automated data refresh
- Improved provenance/confidence system

### Phase 4 — Export/integration

Make it increasingly easy to hand a route to existing tools.

Potential targets can include navigation apps, GPS computers, and route-management services where technically and legally practical.

The goal is interoperability, not platform lock-in.

### Phase 5 — Route intelligence

Potential advanced capabilities:

- Identify gaps between existing route networks
- Detect promising unconnected trail/road corridors
- Compare generated routes against known published routes
- Automatically identify difficult sections
- Detect likely hike-a-bike segments
- Suggest resupply/camping opportunities
- Improve route generation from rider feedback

---

## 16. Technical Principles

### Open source first

Prefer open-source components with permissive licenses and active development.

### Modular architecture

Keep data ingestion, normalization, routing, analysis, and export separate.

### Replaceable routing engine

Do not couple the entire application to one routing engine.

### Reproducible routes

Given the same source data, routing configuration, and inputs, the system should produce reproducible results where practical.

### Explainable routing

The system should eventually be able to explain why a route was selected:

> "This route is 82% unpaved, avoids the highway, and adds 11 miles compared with the more direct option."

This is much more useful than presenting an unexplained route score.

### Don't hide uncertainty

If access, surface, trail condition, or legality is uncertain, show that uncertainty rather than presenting it as fact.

---

## 17. Success Criteria

The project succeeds if a rider can enter two places where they want to travel and discover a practical bikepacking route that they might not have thought to connect manually.

The route does not need to replace the rider's existing navigation application.

In fact, the intended workflow is:

**Find-My-Line finds the line → the rider's existing navigation tool follows the line.**

That separation is a feature, not a limitation.

---

## 18. Initial Development Order

1. Define the route data model.
2. Select the first geographic region for development/testing.
3. Obtain and normalize OSM routing data.
4. Evaluate BRouter integration.
5. Build the first bikepacking routing profile.
6. Implement start/destination routing.
7. Implement alternative-route generation.
8. Calculate route statistics.
9. Build the simplest usable results UI.
10. Add GPX export.
11. Test generated routes against real-world riding conditions.
12. Improve routing weights based on actual failures.
13. Add additional authoritative datasets.
14. Expand export formats and integrations.

Real-world route testing should drive routing improvements. A route that looks excellent numerically but is miserable, illegal, closed, or impossible to ride is a routing failure.

---

## 19. Guiding Principle

> **Find-My-Line should do one thing exceptionally well: find bikepacking routes worth riding, then get out of the rider's way and let their preferred navigation tool handle the ride.**
