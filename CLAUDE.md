# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Project Status

**StickTailor is in Phase 0 — Foundation (pre-development).** No application code exists yet. The repository contains only requirements documentation. See `STICKTAILOR_SETUP.md` for the full product specification.

Current phase work:
- Finalize curve data model schema (DB migrations)
- Scaffold repository with project structure
- Begin Phase 1: curve database + 3D configurator MVP

The `Archive/M2MBlade/` folder contains the legacy 2016 concept — it is for historical reference only and nothing should be reused from it.

---

## Planned Tech Stack

| Layer | Technology |
|-------|-----------|
| Language | TypeScript (full stack) |
| Frontend | TBD — React/Next.js or SvelteKit (not yet decided) |
| 3D Engine | TBD — WebGL-based blade visualization (library choice not yet validated) |
| Backend | TBD — tied to frontend framework choice (API routes for Next.js / SvelteKit endpoints or standalone NestJS at scale) |
| Database | MongoDB |
| Storage | S3-compatible (blade mesh files: STL/OBJ, rendered previews) |
| Hosting | Vercel (serverless-first) |
| Payments | Stripe (EU VAT + NA) |
| E-commerce | Standalone checkout for MVP → Shopify integration (Phase 4) |

---

## Commands

Commands will be added here when the project is scaffolded. Expected standard Next.js scripts:

```bash
npm run dev          # Local development server
npm run build        # Production build
npm run test         # Run test suite
npm run test:watch   # Watch mode for TDD
npm run lint         # ESLint
npm run type-check   # TypeScript checks
```

---

## Architecture

### Key Architectural Principles

1. **Configurator is the product.** The 3D blade visualizer is the core differentiator — invest here first.
2. **Headless commerce.** The configurator and curve library are independent from the checkout layer. Shopify (when integrated) handles cart/checkout/fulfillment, not product definition.
3. **Curve data is the moat.** The parametric curve database is the strategic asset. Design APIs and data models to support future similarity search, AI recommendations, and community marketplace.
4. **Manufacturing is decoupled.** The platform generates a manufacturing work order (curve geometry + stick specs); actual production is handled externally by a CZ/SK manufacturing partner.

### System Context

```
[Customer] → [StickTailor Web App]
                ├── Curve Library (browse, compare, visualize)
                ├── 3D Configurator (select curve, configure stick, preview)
                ├── Order Flow (checkout, payment, tracking)
                └── My Curves (saved curves, scan submissions)

[StickTailor Web App] → [Curve Database] (PostgreSQL + S3)
[StickTailor Web App] → [Stripe] (payments)
[StickTailor Web App] → [Manufacturing Pipeline] (work order — initially manual/email)

[Digitizing Service] → [Curve Database] (scan ingestion, parametric extraction)
```

### Core Data Model

Three entities are central to everything:

**Curve** — A blade pattern definition. Contains:
- `source_type`: `pro_stock | retail | discontinued | community | custom`
- `geometry.parametric_profile`: `curve_depth_mm`, `curve_accent_pct`, `curve_sample_points[]`, `rocker_radius_mm`, `face_angle_*_deg`, `lie`
- `geometry.blade_shape`: `length_mm`, `height_*_mm`, `toe_shape` (`round | square | semi_square`), `toe_radius_mm`
- `geometry.raw_scan_url`: S3 reference to the source mesh (STL/OBJ)
- `provenance`: player name (pro stock), brand/retail pattern, or `submitted_by` user

**StickOrder** — A customer order. Spec fields: `flex` (65/75/85/95), `kick_point` (mid/low), `hand`, `grip`, `length_inches`. Minimum `quantity: 3`. Tracks `mold_status` and `production_status` through the manufacturing pipeline.

**Customer** — User account with `my_curves[]` (saved/favorited), `my_scans[]` (digitized curves), and `order_history[]`.

### 3D Visualization

The configurator renders blade geometry from parametric data in real-time (WebGL). The specific library is TBD and needs current-landscape evaluation. Requirements:
- Rotate, zoom, and inspect from any angle
- Side-by-side or overlay comparison of 2+ curves
- Delta visualization (color-coded differences between curves)
- Desktop + tablet (mobile is view-only)

### MVP Scope Boundaries

**In MVP:**
- Curve library with 100+ digitized patterns
- 3D blade configurator (select + visualize, not freeform sculpt)
- Stick spec configuration (flex, kick point, hand, grip, length)
- Standalone Stripe checkout
- One-piece sticks (OPS) only

**Not in MVP:** Custom shaft shapes, two-piece sticks, curve blending/parametric editing, AI recommendations, B2B portal, Shopify integration, community curve marketplace.
