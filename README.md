# StickTailor

**Direct-to-consumer custom hockey stick brand built on a digitized blade curve library and a real-time 3D blade configurator.**

> Status: Phase 0 — Foundation & Requirements
> Stack: TypeScript · MongoDB · TBD frontend (React/Next.js or SvelteKit) · WebGL geometry engine (TBD)

---

## The Problem

The hockey stick industry has consolidated around 3 retail blade curves (P92, P88, P28) despite NHL pros playing on 20+ distinct patterns. Discontinued curves, pro stock patterns, and custom modifications are only accessible through fragmented secondary markets. No player today can design, visualize, and order a stick with a truly custom blade curve through a unified digital experience.

## The Solution

StickTailor offers:

- **A curve library** — 100+ digitized blade patterns (retail, pro stock, discontinued, community-submitted)
- **A real-time 3D configurator** — parametric curve exploration and visualization in the browser, geometry generated live from parametric inputs (not pre-rendered)
- **Custom manufacturing** — EU-based carbon fiber production using expendable wax molds, enabling low-MOQ production of any curve without traditional tooling costs
- **A digitizing service** — players send in a physical blade; we scan it, archive the curve, and make it reorderable

**Pricing:** €200/stick, minimum order 3 sticks (€600). Target lead time: 8 weeks.

---

## Product Configuration (MVP)

| Parameter | Options |
|---|---|
| Blade curve | 100+ library curves |
| Flex | 65 / 75 / 85 / 95 |
| Kick point | Mid / Low |
| Hand | Left / Right |
| Grip | Yes (single type at launch) |
| Length | Standard |

One-piece sticks (OPS) only. No custom shaft shapes, no two-piece sticks, no parametric curve editing at launch.

---

## Manufacturing Pipeline

```
Custom engine (parametric definition)
  → Data schema (stored in MongoDB)
    → STL export (blade geometry mesh)
      → GCode generation (CNC toolpath)
        → Wax mold (expendable, 5–10 cycle life)
          → CF layup + resin + cure
            (construction method TBD: RTM / prepreg / resin infusion / wet layup)
          → Blade-to-shaft bonding (OPS)
          → Finishing (paint, graphics, grip)
        → QA (visual, flex, weight ±10g)
      → Distribution & logistics
```

Manufacturing target: Czech Republic or Slovakia (CF composites expertise, EU single market, hockey-literate workforce). No partner identified yet.

**Mold strategy:**
- Tier 1 (aluminum, permanent): 15–20 highest-demand curves
- Tier 2 (wax, on-demand): remaining 80+ library curves and all custom curves

---

## Architecture

### Key Principles

- **Configurator is the product.** The 3D blade visualizer is the core differentiator, not a nice-to-have.
- **Headless commerce.** Configurator and curve library are independent from checkout. Shopify integration is planned for Phase 4 — not the MVP.
- **Curve data is the moat.** The parametric curve database is the strategic asset. Designed to support future similarity search, AI recommendations, and community marketplace.
- **Manufacturing is decoupled.** The platform generates a work order (curve geometry + stick specs); production is handled externally.

### System Context

```
[Customer] → [StickTailor Web App]
                ├── Curve Library (browse, compare, visualize)
                ├── 3D Configurator (select curve, configure stick)
                ├── Order Flow (checkout via Stripe)
                └── My Curves (saved curves, scan submissions)

[Web App] → [MongoDB]         (curve library, orders, customers)
[Web App] → [S3-compatible]   (raw scan meshes: STL/OBJ, rendered previews)
[Web App] → [Stripe]          (payments, EU VAT)
[Web App] → [Manufacturing]   (work order export — initially manual)

[Digitizing Service] → [MongoDB]  (scan ingestion, parametric extraction)
```

### Core Data Model

**Curve** — the central entity. Stores parametric geometry (`curve_depth_mm`, `rocker_radius_mm`, `face_angle_*_deg`, `lie`, `curve_sample_points[]`, blade shape), raw scan reference (S3 URL), provenance (pro stock player, retail brand, or community user), and source type (`pro_stock | retail | discontinued | community | custom`).

**StickOrder** — links a customer to a curve + stick spec (`flex`, `kick_point`, `hand`, `grip`, `length_inches`). Minimum quantity: 3. Tracks `mold_status` and `production_status` through the pipeline.

**Customer** — account with saved curves (`my_curves[]`), digitized scans (`my_scans[]`), and order history.

### 3D Configurator

The blade geometry is generated live from parametric inputs — not pre-rendered. The specific WebGL library is TBD (candidates: JSCAD, Replicad/OpenCascade.js, custom math → mesh pipeline). Requirements:

- Parameter change → visible geometry update < 100ms
- Rotate, zoom, inspect from any angle
- Side-by-side or overlay curve comparison with delta visualization
- Desktop + tablet (mobile: view-only)

See [`CONFIGURATOR_RESEARCH.md`](./CONFIGURATOR_RESEARCH.md) for benchmarked reference examples and technology evaluation.

---

## Project Phases

| Phase | Focus | Status |
|---|---|---|
| **0 — Foundation** | Requirements, data model schema, repo setup | In progress |
| **1 — Curve Library & Configurator MVP** | Curve DB, 3D renderer, browser UI, Stripe checkout, deploy | Not started |
| **2 — Digitizing Pipeline** | Scanning R&D, scan ingestion, parametric extraction, customer portal | Not started |
| **3 — Manufacturing Integration** | CNC toolpath generation, work order system, production tracking | Not started |
| **4 — Scale & Community** | Shopify integration, curve blending editor, AI similarity engine, B2B portal | Not started |

---

## Target Markets

- **Primary:** EU — FI, SE, CZ, SK, DE, CH, AT, Benelux, FR
- **Secondary:** North America (CA, US) — ship-to model from EU manufacturing
- **Buyer persona:** Competitive adult players, former junior/college players, gear-obsessed beer leaguers (ModSquadHockey / Sports2k demographic)

---

## Documentation

| Document | Contents |
|---|---|
| [`STICKTAILOR_SETUP.md`](./STICKTAILOR_SETUP.md) | Full product spec, curve data model, platform architecture, manufacturing pipeline, open questions |
| [`CONFIGURATOR_RESEARCH.md`](./CONFIGURATOR_RESEARCH.md) | Benchmark of real-time 3D configurators, UX patterns, technology evaluation, hire recommendation |
| [`CLAUDE.md`](./CLAUDE.md) | Repo guidance for Claude Code (AI assistant) |

---

## Open Questions

Key unresolved items spanning engineering, legal, operations, and finance — tracked in [`STICKTAILOR_SETUP.md §10`](./STICKTAILOR_SETUP.md#10-open-questions--risks):

- CF construction method (RTM / prepreg / resin infusion / wet layup)
- Shaft shell construction (bladder molding vs alternatives)
- Scanning hardware and method for the digitizing service
- Number of parametric control points needed for faithful curve reproduction
- IP considerations around pro stock curve geometries
- VAT handling for EU + NA sales
- Manufacturing partner identification (CZ/SK)
