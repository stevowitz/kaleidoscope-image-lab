# Kaleidoscope Image Lab

## Permanent milestone roadmap and execution priority

**Milestone IDs are permanent historical identifiers. The Priority column reflects the current recommended execution order and may change without renumbering milestones.**

The project currently favors geometry, visual finishing, sacred geometry, mathematical/procedural Pattern Packs, and visual exploration before animation, motion, live input, and platform expansion. That strategy changes execution priority only; it never changes a milestone ID.

## Documentation rules

- A milestone ID identifies the same feature permanently across prompts, backups, implementation reports, QA records, and changelogs.
- Priority is the recommended build order for unfinished work and may be revised as architecture or product goals change.
- Never substitute a priority number for a milestone ID. For example: **Priority 1 — Milestone #11 Smart Randomizer**.
- Complete one milestone at a time and update `PROJECT_NOTES.md` after implementation and QA.

## Next active milestone

**Priority 12 — Milestone #4 Compare & Evolution Mode** is the next active implementation milestone. Milestones #20–#25 now complete the reusable Pattern Pack expansion sequence.

## Completed milestones

| Priority | Milestone ID | Milestone | Status | Short goal | Dependencies |
| --- | --- | --- | --- | --- | --- |
| Complete | #1 | Source Navigator | **Implemented** — physical desktop gesture QA remains a follow-up. | Provide precise synchronized source positioning through an aspect-ratio-preserving navigator. | Existing source-position state, preview drag, controls, and history. |
| Complete | #2 | Live Animation | **Complete** — static and localhost Chrome QA passed on 2026-08-14; cross-browser media playback, alpha, and long-duration performance remain follow-ups. | Provide bounded elapsed-time playback and simple preview recording without flooding history or changing base state. | Stable complete state, control synchronization, history isolation, shared preview renderer, and browser MediaRecorder support. |
| Complete | #8 | Layered Symmetry Engine | **Complete** — manual QA passed on 2026-07-17. | Preserve Mirror and Radial output behind a reusable ordered layer pipeline. | Existing renderers, reversible state, preview/export routing, and responsive controls. |
| Complete | #9 | Symmetry Painting | **Complete** — browser UI QA passed on 2026-07-17; physical desktop stroke QA remains. | Create artwork from imported or blank sources through the shared symmetry renderer and stroke-level history. | Milestone #8, source mapping, history, export, and responsive controls. |
| Complete | #10 | Geometry Library | **Complete** — browser QA passed on 2026-07-17; physical desktop file-picker/gesture QA remains. | Make registered geometry first-class across preview, export, painting, presets, and Randomize. | Milestone #8, complete state, preset normalization, source mapping, and export. |
| Complete | #11 | Smart Randomizer | **Complete** — browser QA passed on 2026-08-02; physical painted-stroke and saved-file inspection remain follow-ups. | Enable guided exploration through weighted strengths, scopes, locks, and bounded geometry-aware variation. | Milestone #10, current Randomize/Mutate paths, complete state, presets, and safe parameter metadata. |
| Complete | #6 | Palette & Post-Processing Foundation | **Complete** — browser QA passed on 2026-08-02; physical file-picker and saved-file inspection remain follow-ups. | Establish shared non-destructive effect architecture and neutral visual-adjustment state for preview and export. | Stable preview/export paths, centralized reversible effect state, alpha preservation, and Reset/history integration. |
| Complete | #12 | Effect Stack | **Complete** — browser QA passed on 2026-08-02; physical imported-source order comparison and saved-file inspection remain follow-ups. | Add reusable, reorderable, toggleable visual finishing with preview/export consistency. | Milestone #6, buffer-aware rendering, effect serialization, and performance safeguards. |
| Complete | #14 | Polar Engine | **Complete** — browser QA passed on 2026-08-02; physical saved-file comparison and native file-picker/drop inspection remain follow-ups. | Add a reusable polar-coordinate stage for tunnel, flower, planet, eye, and vortex forms. | Milestones #8 and #10, shared coordinate transforms, and preview/export parity. |
| Complete | #20 | Sacred Geometry Vector Mask Library | **Complete** — browser QA passed on 2026-08-03; physical saved-file and native picker inspection remain follow-ups. | Establish a reusable Pattern Pack foundation with eight precise Sacred Geometry vector patterns, Overlay/Mask/Guide modes, and preview/export parity. | Milestones #10, #14, and #13 plus shared vector/metadata conventions. |

## Core Visual Foundation — Priorities 4–5

| Priority | Milestone ID | Milestone | Status | Short goal | Dependencies |
| --- | --- | --- | --- | --- | --- |
| Complete | #13 | Crystal Mode | **Complete** — browser QA passed on 2026-08-02; physical imported-source and saved-file comparison remain follow-ups. | Add controlled faceted variation in segment scale, rotation, and offset without replacing the geometry framework. | Milestones #10 and #14, shared state/history, and export consistency. |

## Pattern and Sacred Geometry Expansion — Priorities 6–11

| Priority | Milestone ID | Milestone | Status | Short goal | Dependencies |
| --- | --- | --- | --- | --- | --- |
| Complete | #20 | Sacred Geometry Vector Mask Library | **Complete** — browser QA passed on 2026-08-03; physical saved-file and native picker inspection remain follow-ups. | Establish the Pattern Pack foundation with precise reusable vector masks, overlays, guides, and radial structures. | Milestones #10, #14, and #13 plus shared vector/metadata conventions. |
| Complete | #21 | Mathematical Curve Library | **Complete with post-completion `srcdoc` escaping hotfix** — static payload/script validation passed on 2026-08-03; physical saved-file and native picker inspection remain follow-ups. | Add normalized parameterized curves for masks, paths, guides, and outlines. | Milestone #20 plus reusable vector-path and parameter metadata. |
| Complete | #22 | Fractal Generator Pack | **Complete** — static payload/script validation passed on 2026-08-03; physical visual and saved-file QA remain follow-ups. | Add bounded recursive and iterative structures for generators, masks, guides, source art, and growth systems. | Shared Pattern Pack framework, deterministic generation, and explicit recursion/iteration limits. |
| Complete | #23 | Number-Theory Pattern Pack | **Complete** — static payload/script and bounded-generator QA passed on 2026-08-03; physical visual and saved-file QA remain follow-ups. | Add approachable deterministic generators based on modular arithmetic, sequences, prime distributions, and mathematical ordering. | Pattern Pack framework, Milestone #11, deterministic parameters/seeds, and bounded generation. |
| Complete | #24 | Tiling & Tessellation Pack | **Complete** — static payload/script and bounded-generator QA passed on 2026-08-03; physical visual and saved-file QA remain follow-ups. | Add repeating, aperiodic, and cell-based systems for image regions, masks, clipping, and guides. | Earlier Pattern Packs plus polygon, adjacency, clipping, cell-rendering, and tiling infrastructure. |
| Complete | #25 | Wave-Interference Pattern Pack | **Complete** — static bounded-generator QA and localhost browser preview/export QA passed on 2026-08-13; physical desktop gestures and native picker/Save As QA remain follow-ups. | Add wave fields and interference structures for masks, contours, overlays, and flow guides. | Earlier Pattern Packs plus bounded field sampling, reusable vector commands, and performance safeguards. |

## Exploration and Generative Creativity — Priorities 12–15

| Priority | Milestone ID | Milestone | Status | Short goal | Dependencies |
| --- | --- | --- | --- | --- | --- |
| 12 | #4 | Compare & Evolution Mode | Not started. | Present related variations side by side so users can select a direction without mutating the parent state. | Milestone #11 or reusable Mutate logic, complete state snapshots, efficient thumbnails, and history. |
| 13 | #5 | Favorites Gallery | **Implemented** — gallery interaction and narrow-layout browser QA passed on 2026-08-16; imported-source, representative-stage, delete/clear completion, and reload persistence remain environment-dependent follow-ups. | Save, name, restore, exchange, and reuse discoveries as versioned visual states with thumbnails. | Complete state serialization, source identity handling, persistence, and thumbnail strategy. |
| 14 | #15 | Evolution Lab | Not started. | Support deeper multi-generation visual breeding with parent history and guided selection. | Milestones #4 and #11, generation trails, and efficient thumbnail rendering. |
| 15 | #18 | Procedural Preset Generator | Implemented; runtime QA pending. | Create new presets automatically from weighted artistic rules with preview and save/favorite support. | Milestones #11 and #5, preset schema, Pattern Packs, and thumbnail generation. |

## Motion and Animation — Priorities 16–19

| Priority | Milestone ID | Milestone | Status | Short goal | Dependencies |
| --- | --- | --- | --- | --- | --- |
| Complete | #2 | Live Animation | **Complete** — eight bounded sine-wave channels, Play/Pause/Stop, reversible settings, and 30 fps preview recording implemented. | Provide basic elapsed-time playback of supported visual parameters without flooding history or breaking still export. | Stable complete state, control synchronization, history isolation, and preview/export consistency. |
| 17 | #7 | Organic Motion Engine | **Implemented** — static `srcdoc`/script/id QA passed on 2026-08-14; runtime browser QA is pending because the restricted environment blocks localhost and Playwright-launched Chrome aborts before navigation. | Add smooth procedural drift and natural motion as reusable infrastructure. | Milestone #2 timing, bounded interpolation/noise, state synchronization, and long-running performance. |
| Complete | #3 | Discovery Mode | **Complete** — static payload/script/id QA passed on 2026-08-17; user-reported desktop/browser QA passed on 2026-08-18. | Generate a small bounded batch of flavor-based candidate compositions, preview them, and apply one through the normal state/history path. | Milestones #11 and #18, complete state snapshots, shared thumbnails, Favorites, and reversible state transitions. |
| 19 | #17 | Cinematic Animation | Not started. | Add an advanced timeline with keyframes, easing, looping, camera motion, and motion-graphics-style control. | Milestones #2, #7, and #3, serializable timelines, and export planning. |

## Live Input and Platform Features — Priorities 20–21

| Priority | Milestone ID | Milestone | Status | Short goal | Dependencies |
| --- | --- | --- | --- | --- | --- |
| 20 | #16 | Live Camera Studio | Not started. | Add efficient real-time camera input, pause, snapshot, and live-source experimentation. | Stable source abstraction, continuous rendering, permissions, performance controls, and compatible visual systems. |
| Complete | #19 | Workspace System / Workspace and Project System Expansion | **Complete** — version-1 JSON Save/Open/New actions are in the Library workspace, with bounded imported/painted source restoration and history-safe state loading. Desktop/browser QA passed on 2026-08-18. | Add save/load, persistent workspace state, project management, and long-term versioned usability. | Stable serialization for presets, favorites, effects, Pattern Packs, animation timelines, sources, and preferences. |

## Completed milestone records

### Milestone #1 — Source Navigator — implemented

### Milestone #19 — Workspace / Project System Expansion — complete

The existing Library workspace now contains a compact Project section with New Project, Open Project, and Save Project. Save writes version-1 JSON using the authoritative `getState()` composition snapshot plus a bounded embedded source snapshot. Imported images and painted sources use the existing Favorites canvas/data-URL restoration path, avoiding external file references.

Open validates the project identity, version, required state, and source before changing the active composition. It restores the source first, applies the normalized state through `applyState()`, resets history to one post-load entry, and restores animation/Organic Motion settings without autoplay or recording. New Project uses the existing blank-canvas and Reset/default paths with a compact discard confirmation. Unknown project fields are ignored; project status/dirty tracking is intentionally deferred.

Static validation passed for the escaped outer `srcdoc`, all decoded inline scripts, unique DOM ids, and `git diff --check`. Per user report, desktop/browser QA also passed for native pickers, downloaded JSON inspection, representative imported/painted/combined-stage restoration, history behavior, no-autoplay behavior, responsive Library layout, basic neighboring workspace behavior, and console review.

The navigator shows the oriented source without stretching, supports synchronized pointer dragging and source X/Y controls, remains usable in narrow layouts, and commits at most one Undo state per completed gesture. Physical desktop confirmation remains recommended for file-picker and gesture behavior.

### Milestone #2 — Live Animation — complete

The Animation workspace now provides Play, Pause, Stop, global speed, and eight opt-in sine-wave channels for Global Rotation, Zoom/Scale, Center X/Y, Pattern Rotation/Scale, Polar Rotation, and Crystal Strength. Reversible animation configuration lives in `getState()` while playback phase and frame updates remain session-only; the live renderer derives a bounded temporary snapshot without changing base controls, source pixels, or history.

Preview recording uses the existing 1200 × 1200 canvas through `captureStream(30)` and `MediaRecorder`, detects a supported WebM/MP4 MIME type, offers 6/12 Mbps quality, downloads the result, and reports unsupported or non-origin-clean recording clearly. Static `srcdoc`/script/id checks and localhost Chrome QA passed, including exact Stop restoration, Pause, base editing and Undo, a nonempty WebM, unsupported-capability messaging, a nonempty PNG still export, all six Pattern Pack families, all eight geometries, workspaces, 480 px layout, and clean console/page-error logs. Physical gestures, native picker/drop and Save As, cross-browser media playback/alpha, MP4, and long-duration performance remain follow-ups.

### Milestone #7 — Organic Motion Engine — implemented; runtime QA pending

The existing Live Animation system now includes a compact Organic Motion layer with Drift, Orbit, Breathe / Pulse, Wobble, and bounded deterministic Wander. Organic targets cover paired Source Position and Center Position plus Global Rotation, Zoom / Scale, Pattern Rotation, Polar Rotation, and Crystal Strength. Orbit uses a bounded radius control and paired X/Y offsets for genuinely circular or elliptical source/center movement.

Organic settings are nested in reversible animation state and use the existing `getState()` / `applyState()` / Undo / Redo path. Playback phase remains session-only; Stop and Reset Organic Motion restore the neutral live layer and unchanged base composition. Preview recording continues to capture the same authoritative canvas, so enabled Organic Motion is included without a second renderer or recording-specific path.

Static QA passed for one intact outer `srcdoc`, three parsed decoded inline scripts, and unique DOM IDs. Runtime browser QA was attempted but could not complete because Python Playwright is unavailable, bundled Chromium is absent, localhost binding is restricted, and Playwright-launched installed Chrome aborts before navigation; imported/painted source behavior, console cleanliness, recording artifact capture, physical gestures, and responsive visual inspection remain pending.

### Post-completion Live Animation Source Motion — 2026-08-14

The completed Milestone #2 animation system received a focused non-milestone enhancement adding Source X and Source Y channels. They reuse the existing temporary render-state modulation, source-position sampling controls, history isolation, Stop restoration, and preview recording path. Milestone #7 now extends that same path with bounded Organic Motion; advanced timeline and discovery systems remain separate future work.

### Milestone #8 — Layered Symmetry Engine — complete

`LayerManager` provides a stable ordered layer contract and sequential pixel handoff. Mirror and Radial remain independent compatibility layers, state-backed enablement participates in Undo/Redo, and disabling the active layer produces a centered source fallback rather than a blank result.

Completed QA included mode switching, layer enable/disable, Undo/Redo, all presets, Randomize, Mutate, PNG export, responsive layouts, and browser-console review. No new console errors were found.

### Milestone #9 — Symmetry Painting — complete

Painting Mode supports imported and blank canvas sources, Pencil, Marker, Soft Brush, Eraser, size, opacity, color, shared symmetry rendering, and stroke-level history. Browser UI and source-path QA passed without new console errors; real desktop confirmation remains required for continuous strokes, eraser behavior, and per-drag history grouping.

### Milestone #10 — Geometry Library — complete

`GeometryManager` validates and registers Mirror, Triangle, Hexagon, Starburst, Spiral, Flower, Crystal, and Snowflake through one lifecycle and transform contract. Cached plans and shared allocation-conscious sampling serve preview, export, wheel/source mapping, and Symmetry Painting. Geometry participates in state, history, Reset, presets, and weighted Randomize.

QA covered all eight geometries across landscape, portrait, and square sources; geometry Undo/Redo; legacy presets; bounded Randomize; layer fallback; PNG preview/export parity; Reset; responsive layouts; and console review. Browser QA passed, with physical file-picker and gesture checks still recommended.

### Milestone #11 — Smart Randomizer — complete

Smart Randomizer adds Subtle, Balanced, and Wild strengths; Everything, Geometry Only, Transform Only, and Unlocked Values scopes; and compact session-only locks for Geometry, Segments, Rotation, Zoom, Source Position, and Center Position. Every registered geometry supplies compact profile metadata for weighted selection, preferred segment counts, zoom bands, density limits, and optional center reach. Shared helpers apply simple correlations without changing the `GeometryManager` contract.

Each action builds and validates one complete candidate, preserves locks, applies through `applyState()`, schedules one render, and commits one history entry. Browser QA covered 10 runs per strength, all eight geometries, all scopes and locks, one-step Undo/Redo, 24 deterministic presets, Mutate, Reset, PNG export invocation, imported aspect ratios, imported-image Painting Mode, responsive layouts, and a clean console. Physical painted-stroke source safety and independent saved-file inspection remain desktop-browser follow-ups.

### Milestone #5 — Favorites Gallery — implemented; runtime follow-ups pending

The existing Library workspace now contains a bounded Favorites Gallery with Add to Favorites, compact renderer-generated thumbnails, Restore, inline Rename, Delete, and inline-confirmed Clear All. A Favorite reuses the authoritative `getState()` payload for source transform, geometry/symmetry, Pattern Pack, Polar, Crystal, Effect Stack, animation, and Organic Motion settings. It also stores a conservative 1200px source snapshot and 180px thumbnail, prefers WebP with PNG fallback, and caps the gallery at 24 records.

Browser-local `localStorage` persistence is attempted through a versioned key with malformed-record filtering and a session-only fallback for the sandboxed iframe. Restore stops playback, decodes the source snapshot, applies the saved state through `applyState()`, refreshes the shared preview path, and creates one ordinary history entry when state changes. Painting stroke replay, original full-resolution source recovery, playback phase, and project/workspace save data remain outside this milestone.

Static QA passed for the outer escaped `srcdoc`, all eight decoded scripts, and unique DOM IDs. Localhost browser QA passed blank-source creation, Add to Favorites, thumbnail/name presence, inline rename, restore after Randomize with Undo availability, Clear All confirmation visibility without destructive confirmation, no console errors, and 480px Library reachability without horizontal overflow. The browser file chooser did not expose a usable chooser event, and the sandbox reported storage unavailable, so imported PNG/JPEG intake, representative Pattern Pack/Polar/Crystal/Effect/animation restoration, delete/clear completion, and reload persistence remain desktop/browser-configuration follow-ups.

### Milestone #18 — Procedural Preset Generator — implemented; runtime QA pending

Added eight bounded style recipes, three intensity levels, Generate Preset and Regenerate actions, and compact Pattern/Polar/Crystal/Effects include toggles inside the existing Presets and Exploration group. Recipes choose a coherent geometry and transform first, then optionally configure Pattern Pack, Polar, Crystal, and a maximum three-entry Effect Stack with category-specific palettes. Excluded stages preserve the current state; animation is left unchanged and never auto-starts. Generated candidates use the existing `applyState()` path and one history entry, and remain available to the existing Library → Favorites workflow.

Static validation passed for the escaped `srcdoc`, three decoded inline scripts, 230 unique DOM ids, and all 33 generator Pattern Pack references. A temporary headless Chrome document load succeeded, but interactive browser QA could not run in this environment because Python Playwright is unavailable, the bundled Node Playwright browser is not installed, and CDP access to a manually launched Chrome was blocked. Physical desktop-browser QA remains required before marking this milestone complete.

### Milestone #3 — Discovery Mode — complete

Discovery Mode now lives in the existing Library workspace. It provides eight compact flavors — Balanced, Minimal, Geometric, Organic, Psychedelic, Dark, Luminous, and Surprise Me — plus Discover and Regenerate Batch actions. Each batch contains four bounded candidates generated from the existing Milestone #18 procedural recipes rather than a second recipe system.

Candidate previews are lightweight 132 × 132 thumbnails rendered through the shared renderer. Candidate generation stays outside rendering history and leaves the current composition unchanged until Apply. Applying one candidate uses `applyState()` and the ordinary history boundary for one Undo entry, stops active playback so the applied still is clear, never starts animation, and leaves the existing Library → Favorites action as the save workflow. Batches are cleared when the active source changes.

Static validation passed for the escaped outer `srcdoc`, all decoded inline scripts, unique DOM ids, and whitespace errors. Per user report, desktop/browser QA passed on 2026-08-18, closing flavor generation, visual distinctness, Apply/Undo, Favorites save, responsive inspection, and console review.

Files modified for Milestone #3:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `MILESTONE_ROADMAP.md`

### Milestone #6 — Palette & Post-Processing Foundation — complete

`PostProcessingPipeline` runs after geometry output for both preview and export. It supports neutral-by-default Brightness, Contrast, Saturation, Monochrome, Vignette, Grain, lightweight Glow, two-color Gradient Map, Swap Colors, and Reset Effects controls. Effect settings are part of reversible state; completed edits, Reset Effects, full Reset, Undo/Redo, and new-image sessions preserve neutral/default behavior as expected. The pipeline mutates `ImageData` in place, leaves alpha unchanged, and uses a fast neutral path. Glow is intentionally a highlight lift rather than a full blur/bloom pass. Browser QA covered effect combinations, transparent PNG radial crop, JPEG/PNG intake and export invocation, all eight geometries, representative presets, Randomize, Mutate, Painting Mode, responsive layouts, and a clean console. Physical file-picker, drag/drop, painted-stroke, and independent saved-file comparison remain desktop-browser follow-ups.

### Milestone #12 — Effect Stack — complete

The fixed Milestone #6 controls are now a reusable ordered stack with stable instance ids, registered effect definitions, normalized settings, enabled states, Move Up, Move Down, Duplicate, Remove, Reset Stack, and neutral empty-stack defaults. Preview, fallback rendering, PNG export, and JPEG export all share the same ordered, in-place, alpha-preserving pipeline. Legacy flat effect state migrates safely, while stack-less presets use a neutral stack.

Browser QA covered Add Effect, setting expansion, enable/disable, reorder, duplicate, remove, setting edit, Reset Stack, full Reset, Undo/Redo, reload initialization, PNG export invocation, JPEG export invocation, responsive Effects workspace reachability, and a clean console. The in-app browser did not expose the native file picker or saved artifact, so physical nonblank order-sensitive comparison and independent file inspection remain follow-ups.

### Milestone #14 — Polar Engine — complete

`PolarEngine` is a reusable optional stage between Geometry/Layer Manager output and the ordered Effect Stack. It supports Off, Polar Wrap, Radial Tunnel, Ring Repeat, and Vortex with bounded center, rotation, strength, scale, repeat-count, and direction controls inside the existing Symmetry workspace. Preview, fallback, PNG export, and JPEG export share the same stage through `drawActiveRenderer()` and `LayerManager`.

Polar settings are nested in reversible application state. Mode changes, completed slider edits, Reset Polar, full Reset, Undo, and Redo use the existing history path; states and presets without Polar data safely use Off. The stage preserves alpha, does not mutate source pixels, reuses one sample buffer when active, and skips its work entirely when Off.

Browser QA covered all five modes, mode-specific controls, center/rotation/strength/scale/repeat/direction interactions, all eight geometries, imported PNG, Effect Stack compatibility, presets, Smart Randomizer, Mutate, Reset, Undo/Redo, PNG/JPEG export invocation, responsive layouts, and console review. Physical saved-file comparison and native picker/drop inspection remain follow-ups.

### Milestone #13 — Crystal Mode — complete

`CrystalModeStage` is a reusable optional stage after Polar Engine and before the ordered Effect Stack. It supports Off, Facet, Prism, Shard, and Crystal Bloom with bounded facet size/density, rotation, strength, spread/displacement, direction, reflection, and Reset Crystal controls in the existing Symmetry workspace. The stage uses one shared ImageData path for preview, disabled-layer fallback, Painting Mode, and original-resolution export.

Crystal settings are nested in reversible application state. Mode changes, completed control edits, Reset Crystal, full Reset, Undo, and Redo restore the complete Crystal configuration; old states and presets without Crystal data safely use Off. Active modes reuse one sample buffer, avoid temporary canvases and per-pixel object allocation, clamp samples to avoid blank gaps, and preserve alpha.

Browser QA covered all five modes and mode-specific controls, safe extremes, all eight geometries, all five Polar modes, Effect Stack compatibility, Painting Mode transparency, presets, Smart Randomizer, Mutate, Reset, Undo/Redo, PNG/JPEG export invocation, narrow layout reachability, and console review. The in-app browser did not expose native file-picker or saved-artifact inspection, so physical imported-source and preview/export pixel comparison remain follow-ups.

### Milestone #21 — Mathematical Curve Library — complete with post-completion `srcdoc` escaping hotfix

The shared Pattern Pack stage now includes a Mathematical Curves pack with Rose Curve, Lissajous Figure, Hypotrochoid, Epitrochoid, Archimedean Spiral, Logarithmic Spiral, Lemniscate, and Superformula Shape. Each curve declares stable metadata, bounded creative parameters, defaults, safe ranges, and a deterministic normalized sampled path. The existing Overlay, Mask, and Painting Guide modes, state/history integration, cache, preview, fallback, and export route are reused without a separate curve renderer. No Wolfram runtime or development call was required. A post-completion unescaped quote in the outer iframe `srcdoc` initially prevented embedded-app initialization, so image loading and workspace navigation appeared nonfunctional; the selector quote was escaped and the outer payload plus all decoded embedded scripts now validate. Physical desktop visual and saved-file comparisons remain follow-ups because browser automation could not initialize in this environment.

### Milestone #22 — Fractal Generator Pack — complete

The shared Pattern Pack stage now includes Fractal Generators: Koch Snowflake, Sierpinski Triangle, Dragon Curve, Hilbert Curve, Recursive Tree, Barnsley-style Fern, Apollonian Circle Pack, and Levy C Curve. Each generator declares stable metadata, normalized reusable geometry, safe parameter ranges, deterministic seed behavior where variation is useful, and explicit complexity guidance. The existing Overlay, alpha-preserving Mask, and non-exporting Painting Guide modes, cache, state/history, preview, fallback, and export route are reused without a second renderer. Complexity is capped at 3,072 Koch segments, 729 Sierpinski triangles, 8,192 Dragon/Levy segments, 4,095 Hilbert segments, 2,047 tree branches, 9,000 fern points, and 364 circles. No Wolfram runtime or development call was required. Static QA decoded the outer `srcdoc`, parsed all eight embedded scripts, and confirmed 195 unique DOM IDs; physical desktop visual, imported-file, and saved PNG/JPEG comparisons remain follow-ups because the in-app browser blocks local-file navigation.

### Milestone #23 — Number-Theory Pattern Pack — complete

The existing Pattern Pack framework now includes Number Theory: Ulam Spiral, Prime Spiral, Modular Multiplication Circle, Fibonacci Bloom, Golden-Angle Phyllotaxis, Farey Sunburst, Recamán Web, and Pascal Triangle Modulo Pattern. All eight use metadata-backed, deterministic, bounded native JavaScript geometry and retain the shared Overlay, alpha-preserving Mask, non-exporting Painting Guide, cached geometry, reversible `pattern` state, Undo/Redo, Reset, preview/export, Polar, Crystal, and Effect Stack route. No Wolfram use or runtime dependency was required. Static QA decoded the outer `srcdoc`, parsed all 8 embedded scripts, confirmed 195 unique DOM IDs, and ran every builder twice at out-of-range settings to verify deterministic clamping, finite geometry, and caps. Physical visual, imported-file, painting, responsive-layout, console, and saved PNG/JPEG checks remain desktop-browser follow-ups because local file navigation is blocked in this environment.

### Milestone #25 — Wave-Interference Pattern Pack — complete

The shared Pattern Pack registry now includes Concentric Waves, Dual-Source Ripple Interference, Standing Wave Grid, Radial Interference Field, Moire Line Field, Circular Moire Rings, Harmonic Stripe Field, and a practical Chladni-Inspired Contour Pattern. All builders produce bounded normalized line commands; dual-source and Chladni patterns share a capped sampled-field contour helper. The established metadata controls, cache, reversible `pattern` parameters, one-entry history behavior, Reset, Overlay, alpha-preserving Mask, non-exporting Painting Guide, prior-pack compatibility, and shared preview/export route remain unchanged.

Static QA passed for the outer `srcdoc`, all 8 decoded scripts, 193 unique DOM IDs, deterministic unsafe-input clamping, finite output, and generator caps. Localhost Chromium QA with an actual PNG source and decoded PNG downloads passed all eight patterns at default/minimum/maximum settings, one-step Undo/Redo, Reset plus complete-state restoration, earlier-pack switching, Overlay preview/export parity, Mask transparency, Painting Guide export exclusion, a 480px layout, and console review. Physical native picker/drop, painting and pan/wheel gestures, platform Save As, JPEG compositing, and independent desktop visual review remain follow-ups.

### Post-completion Pattern Pack Expansion — 2026-08-03

Completed Milestones #20 and #22 received a focused, non-milestone expansion. Sacred Geometry gained Fruit of Life, Merkaba / Star Tetrahedron Projection, Sri Yantra–Inspired Grid, Platonic Solid Projection Set, Concentric Hexagram Lattice, and Vesica Chain Mandala. Fractal Generators gained Pythagoras Tree, Minkowski Sausage, Gosper Curve, Vicsek Fractal, Hexaflake, and Cantor Dust. The shared contract, renderer, state/history route, and Overlay/Mask/Painting Guide behavior were preserved; only a bounded metadata-driven categorical selector was added for Platonic projection choice. Milestone #23 remains the next planned milestone.

## Planned architecture guidance

### Visual foundation — Priorities 1–5

- Milestone #11 Smart Randomizer is complete with the deliberately focused Subtle, Balanced, and Wild strengths, geometry-aware profiles, scopes, locks, and one authoritative state path. Additional artistic profile families remain possible future exploration work rather than part of this milestone.
- Milestone #6 and Milestone #12 establish neutral-by-default, alpha-preserving, reversible post-processing shared by preview and export, with the stack serialized as ordinary rendering state.
- Milestone #14 is complete as the modular coordinate-transform foundation, and Milestone #13 is complete as the bounded faceted treatment stage. Both precede the shared Pattern Pack architecture.

### Shared Pattern Pack architecture — Milestones #20–#25

Milestones #20 through #25 should extend one shared Pattern Pack framework, not create six unrelated hardcoded systems. A pack should be able to declare consistent metadata for `id`, name, description, category, rendering type, parameter definitions, defaults, safe ranges, randomization guidance, animation compatibility, mask/overlay/guide/generator compatibility, thumbnail or preview metadata, and version information.

The framework should support pattern discovery, presets, Smart Randomizer integration, painting guides, source masks, overlays, export, saved-project compatibility, and later animation. Prefer normalized vector data, compact equations, deterministic seeds or parameters, and browser-native rendering. Wolfram may assist research, equation/coordinate derivation, SVG or reference-data generation, parameter curation, and validation, but must not become a required application runtime or network dependency.

#### Milestone #20 — Sacred Geometry Vector Mask Library

Candidate families include Flower of Life, Seed of Life, Vesica Piscis systems, Metatron-style lattices, nested and hexagonal circle grids, concentric radial mandalas, star polygons, radial polygons, and interlocking-circle arrangements.

#### Milestone #21 — Mathematical Curve Library

Candidate families include Lissajous and rose curves, hypotrochoids, epitrochoids, Archimedean/logarithmic/Fermat spirals, lemniscates, butterfly curves, and superformula-derived forms.

#### Milestone #22 — Fractal Generator Pack

Candidate families include Koch and Sierpiński structures, dragon and Hilbert curves, recursive trees, Barnsley-style ferns, Apollonian systems, and selected iterated-function systems. Separate generator rules from style and enforce conservative recursion/iteration limits.

#### Milestone #23 — Number-Theory Pattern Pack

Candidate families include modular multiplication circles, prime or Ulam spirals, Fibonacci blooms, golden-angle distributions, Farey sunbursts, Recamán webs, Pascal-triangle modulo patterns, and selected rational structures.

#### Milestone #24 — Tiling & Tessellation Pack

Candidate families include Truchet tiles, Islamic geometry, triangular/hexagonal tilings, Voronoi and Delaunay structures, Penrose-inspired layouts, Archimedean tilings, and selected procedural or aperiodic systems.

#### Milestone #25 — Wave-Interference Pattern Pack

Candidate families include moiré systems, standing waves, harmonic grids, radial interference, circular wave collisions, Chladni- and cymatics-inspired contours, and layered sine-wave fields. Distinguish lightweight contour/vector modes from expensive per-pixel fields and enforce resolution/complexity limits.

### Exploration, motion, and platform guidance

- Milestone #4 should provide side-by-side selection; Milestone #15 should extend that foundation into multi-generation breeding rather than duplicate it.
- Milestone #5 establishes durable bounded saved visual states before Milestone #18 depends on save/favorite support; deeper project/workspace persistence is implemented under Milestone #19.
- Milestone #2 establishes basic playback and Milestone #7 adds natural drift. Milestone #3 should build on both, and Milestone #17 should follow as the advanced timeline layer.
- Milestone #16 should reuse the established source and visual pipelines rather than precede them.
- Milestone #19 is a late-stage persistence and project-management milestone, not a rename of the already completed workspace UI reorganization.

## Regression test after every implementation milestone

- Load JPG, PNG, portrait, landscape, and square images.
- Switch between Mirror Lab and Radial Kaleidoscope.
- Apply presets from every category; use Randomize and Mutate.
- Exercise the active geometry, layer, painting, effect, pattern, or motion path introduced by the milestone.
- Drag the main preview and use pointer-centered wheel zoom.
- Undo and Redo multiple mixed actions; Reset controls.
- Export PNG and inspect the exported file.
- Resize from wide desktop to narrow layout.
- Reload where persistence is involved.
- Check the browser console for warnings and errors.

## Completion rule

A milestone is complete only after its feature works, milestone acceptance tests and the regression checklist pass, no new console errors are present, and `PROJECT_NOTES.md` accurately records implementation, testing, limitations, and the next milestone.

## Prompt sources

The existing DOCX prompt packs already use the permanent original milestone IDs and remain authoritative for their milestone-specific implementation content. New prompts must use the permanent milestone ID from this roadmap, never the current priority number.

## Current recommendation

Proceed next with **Priority 12 — Milestone #4 Compare & Evolution Mode**.
