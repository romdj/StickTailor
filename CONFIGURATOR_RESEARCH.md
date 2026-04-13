# Configurator Research — Executive Summary

> **Purpose:** Benchmark real-time 3D product configurators to inform StickTailor's blade configurator design and technology choices.
> **Date:** 2026-04-06

---

## The Critical Distinction

Most commercial product configurators — including nearly every car configurator — use **pre-rendered image swapping**: every combination is rendered in advance, and selecting an option simply displays the matching image. This is viable when the option space is finite and known.

**StickTailor cannot use this approach.** Blade curves are continuous parametric surfaces. A user adjusting curve depth, rocker accent, or face angle produces geometry that has never been pre-rendered. The configurator must generate and render 3D geometry live, in the browser, from parametric inputs.

This narrows the relevant reference set considerably.

---

## Relevant Examples

### 1. Belforti — Custom Guitar Configurator
**URL:** https://belforti.shop/pages/our-configurator

The closest direct analogy to StickTailor. A craft manufacturer offering real-time 3D configuration of a physical product that goes into production. Key features:
- Hierarchical parameter navigation (category → sub-category)
- Instant 3D preview update on every selection
- Dynamic weight and price calculation updating live
- AI assistant for natural language configuration
- Share, save, PDF export, and add-to-cart from a single flow

**Why it matters:** Demonstrates the full loop from configurator UX to manufacturing intent on a product with real craft complexity.

---

### 2. Edenly — 3D Ring Configurator
**URL:** https://hapticmedia.com/success-stories/3d-ring-configurator-edenly/

Jewelry configurator where changing stone size, metal type, or setting style actually deforms the 3D geometry — not a component swap. This is the same class of problem as blade curves: a continuous surface that changes shape based on parameters.

- Photorealistic real-time rendering
- Dynamic pricing tied to parametric choices (carat, metal grade)
- Live engraving text preview

**Why it matters:** Proof that continuous parametric geometry deformation works in a consumer-facing web product.

---

### 3. iJewel3D — Ring Configurator
**URL:** https://docs.ijewel3d.com/ring-configurator/introduction.html

Technically interesting: components are designed once in Rhino with anchor points, then assembled and deformed dynamically at runtime. Produces hundreds of ring variations in real-time 3D without pre-rendering.

**Why it matters:** The anchor-based parametric assembly pattern is a viable architecture for StickTailor's blade-to-shaft junction.

---

### 4. Ruokangas — Guitar Creator
**URL:** https://ruokangas.com/guitar-creator/

Built on Unity WebGL compiled to WebAssembly — true procedural 3D geometry in the browser, not image swapping. Allows customers to select from actual wood inventory with realistic material rendering.

**Why it matters:** Validates WebAssembly as a viable delivery mechanism for compute-heavy geometry generation in a consumer product context.

---

### 5. BeeGraphy — Parametric Design Platform
**URL:** https://beegraphy.com

Node-based visual programming environment for parametric 3D geometry with real-time preview. Supports fabrication-ready export (DXF). Used in architecture, furniture, and manufacturing.

**Why it matters:** Closest existing tool to what StickTailor's geometry engine needs to do internally. Good reference for the parameter → geometry → export pipeline architecture.

---

### 6. Bitbybit.dev — Browser-Based Parametric CAD
**URL:** https://bitbybit.dev

Runs OpenCascade (professional CAD kernel) in the browser via WebAssembly, with a TypeScript API. Offers visual (node/block) and code-based (TypeScript) entry points.

**Why it matters:** Practical prototyping environment to validate blade geometry math in the browser before committing to a custom engine. Also surfaces OpenCascade.js / Replicad as a serious option.

---

### 7. Replicad — Wavy Vase Example
**URL:** https://replicad.xyz/docs/examples/wavy-vase

TypeScript library built on OpenCascade. The wavy vase example demonstrates how a few parametric inputs (height, radius, twist, side count) produce complex continuous 3D geometry exportable as STL/STEP.

**Why it matters:** Direct architectural reference for StickTailor's geometry engine — parametric inputs → NURBS surface → STL. TypeScript-native, no external tooling.

---

### 8. Huginen Vazy — Vase Generator
**URL:** https://www.huginen.nu/vazy/

The original inspiration. A pure-math parametric surface generator: sine/cosine functions define the profile, no CAD kernel, direct STL output. Sliders → geometry → download. Built circa 2016, still functional.

**Why it matters:** Proof that a blade-class problem (smooth parametric surface → STL) is solvable with minimal stack. Also the architectural ancestor of StickTailor's own Archive/M2MBlade attempt.

---

## UX Patterns Worth Adopting

| Pattern | Application for StickTailor |
|---|---|
| Browse first, configure second | Curve library → select curve → configure stick specs |
| Continuous auto-rotation while browsing | Helps users evaluate blade geometry from all angles passively |
| Orbit control on demand | Lock to manual when user is actively inspecting |
| Live derived values | Update estimated weight, lead time, and price as parameters change |
| Smart validation | Prevent impossible configs (e.g. flex/kick point combos that don't exist in production) |
| Save and share | Users save curve configurations to their account or share with teammates |

---

## Technology Approaches

| Approach | Examples | Fit for StickTailor |
|---|---|---|
| Pure math → mesh (no CAD kernel) | Huginen, M2MBlade JS prototype | Good for MVP; blade geometry is tractable with trigonometric functions |
| OpenCascade via WASM | Replicad, Bitbybit.dev | Higher quality surfaces, STEP/STL export, heavier payload |
| Unity WebGL / WASM | Ruokangas | High visual quality, large bundle, harder to iterate |
| SaaS configurator platforms | Zolak, Vectary | Not applicable — designed for material/color swaps on pre-modeled products |

---

## Recommendation

Before making a library or hire decision, build a **browser-based proof of concept** of the blade geometry engine:

1. Implement the core parametric blade math (curve depth, rocker, face angle, lie) as a function that outputs a triangle mesh
2. Render it in the browser at interactive framerates (target: parameter change → visible update < 100ms)
3. Export a valid STL and verify it against the manufacturing pipeline requirements

This de-risks the biggest technical unknown and produces a concrete technical brief for the creative developer hire.

**Recommended starting point:** Replicad or JSCAD for the geometry engine (TypeScript-native, STL export built-in, active communities), combined with the UI patterns from Belforti as the interaction reference.
