# StickTailor — High-Level Setup & Requirements

> **Status:** MVP Definition — Pre-Development  
> **Last updated:** 2026-04-05  
> **Legacy context:** M2M Blade (original concept, ~2016) — archive folder in repo for reference only, nothing to be reused  

---

## 1. Vision & Positioning

### 1.1 Problem Statement

The hockey stick industry has consolidated around 3 retail blade curves (P92, P88, P28) despite NHL pros using 20+ distinct patterns. Discontinued curves, pro stock patterns, and custom modifications are only accessible through fragmented secondary markets (pro stock resellers, forum trades). No player today can design, visualize, and order a stick with a truly custom blade curve through a unified digital experience.

### 1.2 Value Proposition

StickTailor is a direct-to-consumer custom hockey stick brand built on:

- A digitized library of 100+ blade curves (retail, pro stock, discontinued, community-submitted)
- A real-time 3D blade configurator enabling parametric curve exploration and customization
- EU-based carbon fiber manufacturing using expendable wax molds, enabling low-MOQ production of any curve without traditional tooling costs
- A curve digitizing service allowing players to immortalize and reorder any blade pattern

### 1.3 Brand

- **Name:** StickTailor
- **Tagline:** TBD
- **Positioning:** Performance-grade custom sticks at a fair price (€200 per stick, MOQ 3). Not a budget brand, not a luxury markup — the value is in the customization, not the carbon weave marketing.

### 1.4 Target Markets

- **Primary:** EU (hockey markets: FI, SE, CZ, SK, DE, CH, AT + Benelux/FR for expat/niche)
- **Secondary:** North America (CA, US — massive market but competitive; ship-to model)
- **Buyer persona:** Competitive adult players, former junior/college players, beer league veterans who care about their gear. The ModSquadHockey / Sports2k forum demographic.

---

## 2. Product Specification

### 2.1 Stick Configuration Options (MVP)

| Parameter | Options |
|-----------|---------|
| **Blade curve** | 100+ library curves OR custom via configurator |
| **Flex** | 65 / 75 / 85 / 95 |
| **Kick point** | Mid / Low (2 shaft constructions) |
| **Hand** | Left / Right |
| **Grip** | Yes (default, single type at launch) |
| **Length** | Standard (will need to define; cut-to-length TBD) |

### 2.2 What We Are NOT Building (MVP)

- Custom shaft shapes or profiles
- Two-piece sticks — this is one-piece sticks (OPS) only
- Team/B2B portal (future state)
- Curve blending / parametric editing (future state — configurator V1 is select + visualize, not freeform sculpt)
- AI-powered curve recommendation engine (future state — data model should support it)

### 2.3 Pricing

- **€200 per stick, MOQ 3** (= €600 per order)
- Digitizing service pricing TBD (one-time scan fee + per-reorder margin)
- Shipping: flat rate EU, calculated NA

### 2.4 Lead Time

- **Target SLA:** 8 weeks from order to delivery
- Accounts for: mold production (if new curve), layup, curing, finishing, QC, shipping

---

## 3. Curve Data Model

### 3.1 Core Philosophy

A blade curve is a 3D surface that can be decomposed into a set of measurable, parametric attributes. The data model must support:

1. **Faithful reproduction** from a digitized scan
2. **Human-readable comparison** between curves
3. **Future parametric editing** (blend, tweak, derive)
4. **Similarity search** across the library

### 3.2 Curve Attributes (Draft — Requires Deep Dive)

The following attributes define a blade's geometry. Each has both a **magnitude** (how much) and a **location** (where on the blade) component.

#### 3.2.1 Primary Geometry

| Attribute | Description | Measurement Approach |
|-----------|-------------|---------------------|
| **Curve depth** | Max perpendicular distance from a straight line heel-to-toe | Single value (mm) + location (% from heel) |
| **Curve accent** | Where the curve is most pronounced | Location along blade (heel / mid-heel / mid / mid-toe / toe) |
| **Curve profile** | Shape of the curve distribution — gradual vs concentrated | Multi-point sampling or spline representation |
| **Rocker** | Bottom-edge curvature of the blade (how it sits on ice) | Radius or multi-point profile |
| **Rocker accent** | Where the rocker inflection point is | Location (% from heel) |
| **Face angle / openness** | Twist of the blade face relative to the shaft plane | Degrees, sampled at multiple points (heel, mid, toe) |
| **Lie** | Angle between shaft and blade bottom edge when flat on ice | Single value (typically 4–6) |

#### 3.2.2 Blade Shape

| Attribute | Description |
|-----------|-------------|
| **Blade length** | Heel to toe (mm) |
| **Blade height profile** | Height at heel, mid, toe (mm) — captures max-height blades |
| **Toe shape** | Round / Square / Semi-square (categorical + geometric) |
| **Toe radius** | For round toes, the radius of curvature (mm) |

#### 3.2.3 Derived / Qualitative (Future — AI-Enriched)

| Attribute | Description |
|-----------|-------------|
| **Shooting character** | Quick release / Power / Versatile |
| **Stickhandling rating** | Based on curve geometry |
| **Backhand friendliness** | Inversely correlated with face openness |
| **Comparable retail patterns** | Nearest matches from known curves |
| **Best for** | Tags: toe drags, saucer passes, slap shots, one-timers, etc. |

### 3.3 Raw Scan Data Layer

Every curve in the library must retain:

- **Source:** Pro stock scan / retail scan / community submission / parametric design
- **Provenance:** Player name (if pro stock), brand + model (if retail), user ID (if community)
- **Raw geometry:** Point cloud or mesh (STL/OBJ) from scan
- **Scan metadata:** Date, method, resolution, operator notes
- **Parametric extraction:** The computed attribute values from 3.2

### 3.4 Data Model Entity Sketch

```
Curve
├── id (uuid)
├── name (string) — e.g., "PRO21 MacKinnon" or "Custom #4481"
├── slug (string) — URL-safe identifier
├── source_type (enum: pro_stock | retail | discontinued | community | custom)
├── provenance
│   ├── player_name (string, nullable)
│   ├── brand (string, nullable)
│   ├── retail_pattern (string, nullable) — e.g., "P92", "W10"
│   └── submitted_by (user_id, nullable)
├── geometry
│   ├── raw_scan_url (string) — S3/storage reference to mesh file
│   ├── parametric_profile (CurveProfile)
│   │   ├── curve_depth_mm (float)
│   │   ├── curve_accent_pct (float) — 0.0 = heel, 1.0 = toe
│   │   ├── curve_sample_points[] — array of {position_pct, depth_mm}
│   │   ├── rocker_radius_mm (float)
│   │   ├── rocker_accent_pct (float)
│   │   ├── face_angle_heel_deg (float)
│   │   ├── face_angle_mid_deg (float)
│   │   ├── face_angle_toe_deg (float)
│   │   └── lie (float)
│   └── blade_shape (BladeShape)
│       ├── length_mm (float)
│       ├── height_heel_mm (float)
│       ├── height_mid_mm (float)
│       ├── height_toe_mm (float)
│       ├── toe_shape (enum: round | square | semi_square)
│       └── toe_radius_mm (float, nullable)
├── tags[] (string[]) — qualitative descriptors
├── similar_curves[] (curve_id[]) — computed similarity references
├── created_at (datetime)
└── updated_at (datetime)

StickOrder
├── id (uuid)
├── customer_id (uuid)
├── curve_id (uuid) — reference to Curve
├── specs
│   ├── flex (enum: 65 | 75 | 85 | 95)
│   ├── kick_point (enum: mid | low)
│   ├── hand (enum: left | right)
│   ├── grip (boolean, default true)
│   └── length_inches (float)
├── quantity (int, min 3)
├── mold_status (enum: existing | to_produce)
├── production_status (enum: queued | mold_production | layup | curing | finishing | qc | shipped)
├── ordered_at (datetime)
└── target_ship_date (datetime)

Customer
├── id (uuid)
├── email (string)
├── name (string)
├── location (country)
├── my_curves[] (curve_id[]) — saved/favorited curves
├── my_scans[] (scan_id[]) — digitized curves owned by customer
└── order_history[] (order_id[])
```

---

## 4. Platform Architecture

### 4.1 Stack Decision

| Layer | Choice | Rationale |
|-------|--------|-----------|
| **Language** | TypeScript (full stack) | Single language, type safety across FE/BE |
| **Frontend** | TBD — React/Next.js or SvelteKit | Framework choice not finalized; both provide SSR for SEO, decision pending evaluation |
| **3D Engine** | TBD (WebGL-based) | Library choice requires current-landscape evaluation — Three.js/R3F was a prior reference point, not a validated decision |
| **Backend** | Next.js API routes (MVP) → NestJS (scale) | Start simple, extract services later |
| **Database** | MongoDB | Document model fits flexible curve geometry/parametric data naturally |
| **Storage** | S3-compatible (scan files, mesh assets) | Raw scans, STL/OBJ files, rendered previews |
| **Hosting** | Serverless-first (Vercel for FE, serverless functions for API) | Low ops overhead, scale on demand |
| **E-commerce** | Shopify integration (future) — standalone checkout for MVP | Avoid Shopify lock-in during validation, plan integration path |
| **Payments** | Stripe | EU + NA support, handles VAT |

### 4.2 Key Architectural Decisions

- **Configurator is the product.** The 3D blade visualizer is not a nice-to-have — it's the core differentiator. Invest here first.
- **Headless commerce.** Even when Shopify integration comes, the configurator and curve library live independently. Shopify handles cart/checkout/fulfillment, not product definition.
- **Curve data is the moat.** The parametric curve database is the strategic asset. Design the API and data model to support future applications (similarity engine, AI recommendations, community marketplace).
- **Manufacturing is decoupled.** The platform generates a manufacturing spec (curve geometry + stick specs) that feeds into a production pipeline. The platform does NOT manage CNC/mold/layup — it produces a work order.

### 4.3 System Context (C4 Level 1)

```
[Customer] → [StickTailor Web App]
                ├── Curve Library (browse, compare, visualize)
                ├── 3D Configurator (select curve, configure stick, preview)
                ├── Order Flow (checkout, payment, tracking)
                └── My Curves (saved curves, scan submissions)

[StickTailor Web App] → [Curve Database] (PostgreSQL + S3)
[StickTailor Web App] → [Stripe] (payments)
[StickTailor Web App] → [Shopify] (future: cart/checkout delegation)
[StickTailor Web App] → [Manufacturing Pipeline] (work order export — initially manual/email)

[Digitizing Service] → [Curve Database] (scan ingestion, parametric extraction)
```

---

## 5. Configurator Requirements

### 5.1 MVP Configurator Flow

1. **Browse** the curve library — search, filter by attributes, view 2D/3D previews
2. **Select** a curve — view detailed parametric breakdown, overlay comparison with other curves
3. **Configure** the stick — flex, kick point, hand, grip, length
4. **Preview** in 3D — real-time blade rendering, rotate/zoom
5. **Add to cart** → checkout

### 5.2 3D Visualization Requirements

- Real-time WebGL rendering of the blade from parametric data
- Ability to rotate, zoom, and inspect the blade from any angle
- Side-by-side or overlay comparison of 2+ curves
- Highlight differences between curves (color-coded delta visualization)
- Responsive — must work on desktop and tablet (mobile is view-only, not configure)

### 5.3 Future Configurator Features (Post-MVP)

- **Parametric editing:** Adjust individual curve attributes via sliders (e.g., "add 2mm toe hook")
- **Curve blending:** Select 2 curves, blend at a configurable ratio
- **Community curves:** Browse and fork other users' custom curves
- **AR preview:** "View in your space" on mobile (à la ProStockHockeySticks but for any curve)

---

## 6. Digitizing Service

### 6.1 Concept

Players send in a physical blade (or a full stick). StickTailor scans it, extracts the parametric profile, archives it in the customer's account, and returns the stick. The customer can then reorder that exact curve forever.

### 6.2 Ingestion Pipeline (TBD — Requires R&D)

```
Physical blade arrives
  → Clean and prep
  → Scan (method TBD: photogrammetry / structured light / laser)
  → Generate point cloud / mesh
  → Parametric extraction (automated + manual QC)
  → Store raw scan + parametric profile in Curve Database
  → Associate with customer account
  → Return blade to customer
```

### 6.3 Open Questions

- Scanning method and hardware selection — cost vs fidelity tradeoff
- Automated parametric extraction accuracy — tolerance thresholds
- Turnaround time target for scan service
- Pricing model (one-time fee? included with first order?)

---

## 7. Manufacturing Pipeline

### 7.1 Process Overview

```
Custom engine (parametric definition)
  → Custom data schema (stored in DB)
    → STL export (blade geometry mesh)
      → Check if mold exists for this curve
         → YES: Retrieve wax mold
         → NO:  Generate GCode (CNC toolpath from STL)
                → CNC wax mold (expendable, 5-10 cycle life)
      → CF layup on blade mold
      → Resin application + cure
          (method TBD — RTM / prepreg / resin infusion / wet layup;
           shaft shell construction method also TBD — bladder common in industry)
      → Blade-to-shaft bonding (OPS construction)
      → Finishing (paint, graphics, grip application)
      → QA
          → Visual inspection against reference
          → Flex verification, weight check (target ±10g), cosmetic
          → Future: dimensional scan vs. parametric spec, automated pass/fail
      → Distribution & logistics
```

### 7.2 Manufacturing Location

- **Target:** Czech Republic or Slovakia
- **Rationale:** Existing CF composites knowledge, hockey-literate workforce, EU single market, competitive labor costs
- **Status:** Exploratory — no manufacturing partner identified yet

### 7.3 Wax Mold Economics

- Wax mold cost per unit: TBD (target: €50-100 per mold)
- Mold lifecycle: 5-10 blade pulls per mold
- Per-blade mold amortization: ~€6-20
- Break-even vs aluminum mold: aluminum only justified for core curves with consistent demand

### 7.4 Mold Inventory Strategy

- **Tier 1 — Permanent molds (aluminum):** 15-20 highest-demand curves (P92, P28, P88, P90TM, top pro stock patterns)
- **Tier 2 — On-demand molds (wax):** Remaining 80+ library curves, custom curves
- Decision to promote Tier 2 → Tier 1 based on order volume data

---

## 8. QC & Validation

### 8.1 MVP QC

- Visual inspection of blade geometry against reference
- Flex verification (manual flex test)
- Weight check (target tolerance: ±10g)
- Cosmetic inspection

### 8.2 Future QC

- Dimensional scanning of produced blade vs digital spec (tolerance TBD)
- Automated pass/fail against parametric profile
- Production batch tracking and traceability

---

## 9. Project Phases

### Phase 0 — Foundation (Current)
- [x] Market validation research (NHL curve usage, community demand)
- [ ] Finalize this requirements document
- [ ] Set up repository and project structure
- [ ] Define curve data model schema (Prisma / DB migrations)

### Phase 1 — Curve Library & Configurator MVP
- [ ] Build curve database and seed with initial data (manual entry for known curves)
- [ ] Build 3D blade renderer (Three.js / React Three Fiber)
- [ ] Build curve browser UI (search, filter, compare)
- [ ] Build stick configurator flow (curve → specs → preview)
- [ ] Standalone checkout (Stripe integration)
- [ ] Deploy (Vercel or equivalent)

### Phase 2 — Digitizing Pipeline
- [ ] R&D scanning method and hardware
- [ ] Build scan ingestion pipeline
- [ ] Automated parametric extraction from mesh
- [ ] Customer portal: "My Curves" with scan management

### Phase 3 — Manufacturing Integration
- [ ] Identify and onboard CZ/SK manufacturing partner
- [ ] CNC toolpath generation from curve parametric data
- [ ] Work order management system
- [ ] Production status tracking (customer-facing)

### Phase 4 — Scale & Community
- [ ] Shopify integration (headless checkout)
- [ ] Community curve sharing and forking
- [ ] Curve blending / parametric editor
- [ ] AI similarity engine and recommendation system
- [ ] B2B portal (teams, junior programs)

---

## 10. Open Questions & Risks

| # | Question | Owner | Status |
|---|----------|-------|--------|
| 1 | Wax mold material selection and lifecycle validation | Romain | Exploratory |
| 2 | CZ/SK manufacturing partner identification | Romain | Not started |
| 3 | Scanning hardware and method selection | Romain | TBD |
| 4 | Number of control points needed for faithful curve reproduction | Engineering | Requires R&D |
| 5 | Blade flex/energy transfer quality at target price point | Engineering | Requires prototyping |
| 5b | CF construction method selection: RTM vs prepreg vs resin infusion vs wet layup — cost, quality, MOQ tradeoffs | Engineering | Requires R&D |
| 5c | Shaft shell construction: bladder molding vs other methods — industry standard but needs validation for OPS | Engineering | Requires R&D |
| 6 | Automated parametric extraction accuracy from 3D scans | Engineering | Requires R&D |
| 7 | IP considerations — can pro stock curve geometries be freely reproduced? | Legal | Not started |
| 8 | VAT handling for EU + NA sales | Finance | Not started |
| 9 | Shipping logistics for NA from EU manufacturing base | Operations | Not started |
| 10 | Minimum viable curve library size to launch | Product | 100 target, validate |

---

## 11. Reference & Context

### 11.1 Competitive Landscape

| Player | Model | Strengths | Gaps |
|--------|-------|-----------|------|
| **ProStockHockeySticks** | 50+ pro curves, custom builds, 3D visualizer | Best curve variety, ships to EU from Finland | No true parametric customization, no digitizing service |
| **HockeyStickMan** | Pro stock resale + Pro Blackout line with extinct curves | Huge inventory, strong community presence | Resale model, no custom manufacturing |
| **All Black Hockey Sticks** | Budget composite sticks | Low price point | Limited curve selection, lower build quality |
| **Chinese OEM (Hyoung, etc.)** | White-label manufacturing | Very low cost, flexible MOQ | No consumer brand, no curve expertise, quality variance |
| **Bauer / CCM / Warrior** | Mass retail | Brand trust, distribution, R&D | Shrinking curve selection, no customization |

### 11.2 Key Community Sources

- ModSquadHockey forums (modsquadhockey.com)
- Sports2k Pro Stock Hockey forums (sports2k.com)
- ProStockHockeySticks 3D Curve Visualizer
- HockeyStickMan Blade Chart & Pattern Database
- GearGeek.com player equipment database
