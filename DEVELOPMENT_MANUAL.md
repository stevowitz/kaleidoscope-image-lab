# Kaleidoscope Image Lab

## Design & Development Manual (v1)

This manual is the long-term source of truth for project standards. Every Codex milestone should begin by reading `PROJECT_NOTES.md` and this document.

## 1. Project vision

Create a professional creative exploration application centered on symmetry, kaleidoscopes, procedural image generation, and artistic discovery. Expand carefully while maintaining stability.

## 2. Development philosophy

- Preserve working functionality.
- Read `PROJECT_NOTES.md` first.
- Create or verify a backup before editing.
- Keep milestones focused: implement one milestone at a time.
- Treat milestone IDs as immutable historical identifiers. Execution priority may change, but prompts, backups, QA records, and changelogs must retain the original milestone ID.
- Use the permanent milestone ID—not its current execution-priority number—in every new prompt and implementation record.
- Preserve Undo/Redo behavior.
- Avoid duplicated logic and keep the existing state path authoritative.
- Build visual identity, geometry depth, and reusable pattern systems before investing heavily in motion, live input, or platform expansion.
- Test manually before completion.
- Update `PROJECT_NOTES.md` after every milestone.

## 3. Current architecture

The application is a self-contained browser tool with no package manager, framework, dependency manifest, build configuration, or separate source modules. `kaleidoscope-image-lab.html` is the authoritative application file. Its outer document provides a full-viewport sandboxed iframe whose escaped `srcdoc` contains the embedded HTML, scoped CSS, and JavaScript application.

### Rendering engine

- The embedded editor is scoped to `#kaleidoscope-image-lab` inside a single IIFE.
- Rendering and controls use native browser APIs.
- The preview uses a 1200 × 1200 internal canvas.
- `requestRender()` coalesces updates through `requestAnimationFrame`.
- Live Animation requests at most 30 preview renders per second through that same `requestRender()` path; it does not create a second renderer or write per-frame values into controls.
- `drawActiveRenderer()` routes both preview and export through the active compatibility layer and centralized Geometry Manager.
- Interactive rendering uses a rotated, cached source preview capped at 1400 pixels on its longest side.
- The original decoded image is retained for original-resolution export.

### Escaped `srcdoc` safeguard

The application is embedded inside an escaped iframe `srcdoc`. Quotes, selectors, template strings, HTML fragments, SVG markup, and generated JavaScript added to the embedded app must remain correctly escaped for the outer document. Inner-script syntax validation alone is insufficient: after embedded markup or JavaScript changes, validate both the outer HTML/`srcdoc` payload and every embedded script after decoding. A broken `srcdoc` can prevent the whole embedded app from initializing, making unrelated controls appear nonfunctional.

### Painting source architecture

Painting Mode adds an internal canvas source rather than a second preview renderer. Imported images are copied into the painting canvas when painting begins, while New Blank Canvas creates a transparent 1200 × 1200 source. Stroke points are mapped through the existing preview coordinate conversion and replayed into the source canvas; `drawActiveRenderer()` and the `LayerManager` remain the single preview/export rendering path. The source keeps a reusable pixel buffer and replayable stroke list so future layers, custom brushes, pressure, and masks can extend the source model without duplicating symmetry math.

### Layered Symmetry Engine

The rendering flow is organized as a reusable `LayerManager` pipeline. Layers are ordered, enabled/disabled through state, and expose a consistent interface: `id`, `type`, `enabled`, `settings`, `initialize()`, `render()`, and `dispose()`.

- Mirror Layer and Radial Layer are the initial independent layer types.
- The manager executes enabled, applicable layers sequentially and passes each layer's pixel output to the next stage.
- The existing Mode selection determines which symmetry layer applies, preserving the prior Mirror-versus-Radial visual behavior while allowing future layers to be inserted without rewriting the manager.
- When the active layer is disabled, the manager skips it and the renderer uses a centered, aspect-preserving source fallback.
- The current pipeline avoids an extra canvas copy for the one-layer path; future effect or geometry layers can reuse the same buffer-oriented contract.
- The Symmetry Layers panel is intentionally compact and exposes enable/disable, collapse/expand, and placeholder settings only. It does not redesign existing controls.

### Geometry Library

Geometry is a first-class reversible state concept above the compatibility layers. `GeometryManager` owns a registry of independent geometry generators and is the only component that performs geometry selection, cached plan generation, coordinate transformation, and source sampling.

Every geometry must expose the same contract:

```js
{
  id,
  name,
  description,
  settings,
  initialize(),
  generateSegments(),
  transformSegment(),
  dispose()
}
```

- The initial registry contains Mirror, Triangle, Hexagon, Starburst, Spiral, Flower, Crystal, and Snowflake.
- Mirror contains the preserved legacy Mirror Lab and Radial Kaleidoscope formulas, selected internally by the compatibility `renderMode`; this keeps existing presets visually stable.
- The Mirror and Radial layer definitions do not contain geometry math. Both delegate their applicable render pass to `GeometryManager.render()`.
- `generateSegments()` returns a reusable plan for the output size and cache-affecting controls. The manager retains one cached plan per geometry.
- `transformSegment()` writes normalized source coordinates into a reusable object. The manager performs one shared pixel-sampling loop for all geometry types and avoids per-pixel object allocation.
- The geometry and layer managers initialize at application startup and dispose their registered implementations on `pagehide`.
- Preview, original-resolution export, pointer-centered zoom mapping, and Symmetry Painting source mapping all use the same active geometry transform.
- Add future geometry types by implementing the contract and registering them; do not add geometry-id conditional chains to the manager, renderer, export path, or painting source.

### Future Pattern Pack architecture

Milestones #20 through #25 should share a Pattern Pack framework rather than introduce unrelated hardcoded systems. Pattern packs should be able to describe stable `id`, name, description, category, rendering type, parameter definitions, default values, safe ranges, randomization guidance, animation compatibility, mask/overlay/guide/generator compatibility, thumbnail or preview metadata, and version information.

- Prefer native browser rendering from normalized vector data, compact equations, deterministic seeds, or bounded generator rules; Wolfram may help derive and validate equations, coordinates, SVG paths, reference data, and curated ranges, but it must remain an optional development aid rather than a runtime dependency.
- Expose meaningful creative controls and deterministic recreation while enforcing safe parameter, sampling, recursion, density, resolution, and iteration limits.
- Keep generator logic, rendering style, and UI presentation separable so pattern packs can integrate consistently with presets, Smart Randomizer, animation, painting guides, source masks, overlays, export, and saved-project state.
- Extend the existing Geometry Library, Layer Manager, shared state, and preview/export paths where appropriate; do not replace them or create a separate renderer and control system for every pack.
- Leave exact interfaces and pack-specific engineering decisions to the individual milestone prompts.

### Mirror Lab

Mirror Lab is the default mirror-based renderer. Its supported modes are Alternating, Left copy, Right copy, and Pinwheel. Its renderer math is intentionally separate from radial rendering and should not be changed without an explicit requirement.

### Radial Kaleidoscope

Radial Kaleidoscope maps output pixels into angular wedges around a configurable radial origin. It supports configurable Center X/Y, optional alternating reflection, and optional circular cropping. The radial center marker and Option/Alt-drag interaction are separate from ordinary source panning.

### Shared state and controls

`getState()` and `applyState()` define the reversible rendering state and synchronize paired slider/number controls, switches, mode buttons, mode-specific visibility, and rendering. The current state includes:

```js
{
  segments,
  rotation,
  zoom,
  offsetX,
  offsetY,
  centerX,
  centerY,
  circularCrop,
  alternateMirror,
  mode,
  renderMode,
  geometryId,
  pattern: {
    enabled,
    packId,
    patternId,
    useMode,
    scale,
    rotation,
    centerX,
    centerY,
    lineWidth,
    lineColor,
    opacity,
    ...boundedPatternParameters
  },
  animation: {
    speedMultiplier,
    channels: {
      [channelId]: { enabled, amount, speed }
    },
    organic: {
      channels: {
        [targetId]: { enabled, style, amount, speed, radius }
      }
    }
  },
  effectStack: [{ id, type, enabled, settings }]
}
```

Animation configuration is reversible rendering state. Playback state, elapsed phase, recording state, recorded chunks, preview background, original-preview visibility, distraction-free layout, filename, and export settings are not part of rendering history.

Smart Randomizer strength, scope, and lock selections are also UI/session state rather than rendering state. They must not enter `getState()`, presets, export pixels, or Undo/Redo. A Randomize action itself produces a complete rendering candidate and applies it atomically through `applyState()`.

### Post-processing architecture

Milestone #12 extends the Milestone #6 finishing stage into a reusable ordered Effect Stack. `PostProcessingPipeline` runs after the active geometry layer and is called by the existing shared preview/export renderer path.

- Keep the complete effect stack inside the reversible `getState()` snapshot and synchronize it through `applyState()`. Entries must contain stable ids, registered types, enabled states, normalized settings, and array order.
- Use a small registered effect contract with names, defaults, normalization, neutral checks, and apply methods. Legacy flat effect state must migrate safely; stack-less presets fall back to an empty neutral stack.
- An empty stack, disabled entries, and neutral settings must return the geometry `ImageData` unchanged so legacy output remains stable. Effects run in array order and may be repeated unless a future effect explicitly forbids repetition.
- Prefer in-place `ImageData` mutation and deterministic, allocation-light pixel loops. Avoid per-effect canvas copies and duplicate preview/export implementations.
- Preserve source pixels and alpha. JPEG background compositing remains an export-only encoding step after the shared rendered result.
- Keep expensive spatial blur, masking, and blend modes out of this focused stack milestone; future effects must retain explicit performance safeguards.

### Polar transformation architecture

Milestone #14 adds `PolarEngine` as one optional, reusable coordinate/pixel stage after the active geometry layer and before `PostProcessingPipeline`.

- `LayerManager.render()` owns the single handoff: Geometry output → Polar Engine → Effect Stack → target context. `drawActiveRenderer()` uses that same route for preview and export, including the disabled-layer source fallback.
- Polar accepts `ImageData` and returns the transformed buffer. It uses a reusable `Uint8ClampedArray` sample buffer, bounded settings, nearest-neighbor sampling, and no per-pixel object allocation or temporary canvases. The Off mode returns immediately.
- The compact API is intentionally mode-focused: `apply(imageData, settings)`, `applyToContext(context, width, height, settings)`, and `dispose()`. The initial modes are Off, Polar Wrap, Radial Tunnel, Ring Repeat, and Vortex.
- Polar controls are kept in the existing Symmetry workspace. Mode-specific visibility is UI-only; all render settings live in the nested reversible `polar` state object and are synchronized through `getState()`/`applyState()`.
- Presets, Smart Randomizer, and Mutate preserve the current Polar state during this foundation milestone. Old state/preset data without a Polar object defaults safely to Off.
- Source pixels remain unchanged. Alpha is copied from the sampled geometry output, so transparent painted/PNG pixels remain transparent and JPEG compositing stays in the existing export-only path.

### Crystal transformation architecture

Milestone #13 adds `CrystalModeStage` as one optional, reusable faceted pixel stage after `PolarEngine` and before `PostProcessingPipeline`.

- `LayerManager.render()` owns the complete handoff: Geometry output → Polar Engine → Crystal Mode → Effect Stack → target context. `drawActiveRenderer()` uses the same route for preview, fallback rendering, Painting Mode, and export.
- Crystal accepts `ImageData` and returns the transformed buffer. Its five mode contracts are Off, Facet, Prism, Shard, and Crystal Bloom; each keeps its math separate while sharing bounded settings, clamped nearest-neighbor sampling, one reusable `Uint8ClampedArray`, and `dispose()`.
- Off returns the input immediately. Active modes avoid temporary canvases and per-pixel object allocation, clamp samples to the frame to avoid blank gaps, and copy alpha from the sampled pixel without mutating source data.
- Crystal controls remain in the existing Symmetry workspace and mode-specific visibility is UI-only. Settings live in the nested reversible `crystal` state object, with old state/preset data defaulting safely to Off.
- Presets, Smart Randomizer, and Mutate preserve the current Crystal state during this focused milestone; Crystal Mode is a controlled treatment, not a replacement geometry, 3D lighting system, or refraction simulation.

### Pattern Pack architecture

Milestones #20 through #22 establish the reusable Pattern Pack contract through Sacred Geometry, Mathematical Curves, and Fractal Generators. A pack declares stable metadata (`id`, `name`, `description`, `category`, `version`, `renderingType`, supported use modes, and pattern definitions). Each pattern declares defaults, safe ranges, parameter definitions, supported parameter keys, and a deterministic `build(settings)` function that returns compact normalized vector commands. Parameter definitions may additionally supply a bounded `options` list for a compact categorical selector; this remains ordinary `pattern` state and is backward-compatible with range controls.

- `SacredGeometryStage` is the shared Pattern Pack vector path for Overlay, Mask, and Painting Guide support. It converts normalized circles, lines, polygons, sampled paths, and bounded point buffers into native Canvas commands at the target preview or export dimensions; it does not import raster pattern assets or call a network service.
- The pipeline is Geometry / Layer Manager → Polar Engine → Crystal Mode → Sacred Geometry → Effect Stack → preview or export. `LayerManager.render()` and the disabled-layer fallback use the same stage, preserving preview/export parity.
- Overlay blends vector color over the current `ImageData`; Mask multiplies alpha by bounded vector coverage and clears fully transparent RGB channels; Painting Guide is drawn after preview rendering and remains outside the source pixels and export path.
- Pattern state is nested under `pattern` and defaults safely to disabled for older states and presets. State edits use the existing `getState()`/`applyState()`/`statesMatch()` history path. Smart Randomizer, presets, and Mutate preserve current Pattern Pack settings until a later milestone explicitly integrates them.
- Keep later Pattern Packs on this contract. Prefer normalized procedural vector data, explicit safe limits, native rendering, and small metadata-driven UI adaptations; do not create a separate renderer, export path, or marketplace loader for each pack.

### Live Animation and preview recording

- Live Animation is a temporary modulation layer over the current base `getState()` snapshot. `getRenderableState()` clones only the nested objects it modulates, applies bounded basic and Organic Motion offsets, and sends the derived snapshot through the established Layer Manager → Polar → Crystal → Pattern → Effect renderer.
- Milestone #2 plus its focused post-completion source-motion enhancement supports Global Rotation, Zoom/Scale, Source X, Source Y, Center X, Center Y, Pattern Rotation, Pattern Scale, Polar Rotation, and Crystal Strength. Source X/Y target the existing `offsetX`/`offsetY` source-sampling controls in the temporary snapshot; a channel may safely target an inactive subsystem without forcing that subsystem on.
- Milestone #7 adds compact Organic Motion settings for Source Position, Center Position, Global Rotation, Zoom/Scale, Pattern Rotation, Polar Rotation, and Crystal Strength. Drift, Orbit, Breathe / Pulse, Wobble, and bounded deterministic Wander remain lightweight signals over the same temporary snapshot; Source and Center Orbit are paired X/Y offsets rather than separate competing channels.
- Play and Pause retain elapsed phase. Stop cancels playback, resets phase to zero, and renders the unchanged base state. Playback and frames never commit history; completed animation-setting edits use the ordinary 80-entry history path.
- The loop is bounded to 30 requested renders per second and relies on synchronous renderer completion for natural back-pressure. Avoid DOM reconstruction, canvas copies, or control synchronization from live modulated values.
- Preview recording uses the authoritative 1200 × 1200 preview canvas through `canvas.captureStream(30)` and `MediaRecorder`. Detect a supported WebM/MP4 MIME type, keep bitrate choices minimal, download through an object URL, stop stream tracks after finalization, and report unsupported or non-origin-clean canvases visibly.
- Recording is a preview capture, not a second renderer or original-resolution video export. Container, codec, effective bitrate, and alpha support are browser capabilities and must be documented rather than assumed.

### Favorites Gallery persistence and thumbnails

- Milestone #5 uses the existing `getState()` / `applyState()` path for saved visual settings. A Favorite stores source transform, geometry/symmetry, Pattern Pack, Polar, Crystal, Effect Stack, animation, and Organic Motion configuration; it does not store generated geometry caches or create a parallel rendering-state model.
- Each Favorite stores a bounded source snapshot and a 180 × 180 thumbnail. The thumbnail is rendered through the shared `drawActiveRenderer()` path, with WebP preferred and PNG fallback. Source snapshots are capped at 1200px on the longest side to keep gallery storage practical.
- The gallery attempts `localStorage` persistence under a versioned key, filters malformed records, caps the collection at 24 items, and falls back to session-only behavior when the sandboxed iframe cannot access storage or a quota is exceeded. A storage failure must not discard the in-memory gallery or make other Favorites unusable.
- Restore stops playback, decodes the saved source snapshot, applies the saved state once through `applyState()`, and uses the ordinary history path for one sensible rendering-state entry. Playback phase, preview helpers, project/workspace data, and recording files remain session-only.
- The current Painting Mode serializer does not include replayable strokes or brush settings. Favorites may preserve the current painted pixels as a bounded source snapshot, but future work must not imply stroke-level restoration until the authoritative painting state model supports it.

### Image loading and source data

Supported inputs are PNG, JPEG/JPG, WebP, GIF, BMP, and AVIF. File picker, drag-and-drop, and clipboard paste share `loadFile()`. Images are decoded with browser image APIs, deliberately rotated 90 degrees left into a source canvas, cached for interaction, and used to reset controls and begin a fresh history session. The forced `importRotation = -90` behavior is deliberate current product behavior and must be rechecked against real user images before changing.

### Source Navigator

The navigator is collapsed by default and appears after an image is loaded. It preserves the oriented source aspect ratio with letterboxing, marks source position with a teal circle, and marks radial origin with an orange diamond when Radial Kaleidoscope is active. Navigator, main-preview, and control interactions use the existing state as the single source of truth; completed gestures commit at most one history state.

### Presets, Randomize, and Mutate

The preset library is centralized in one validated array with stable ids, names, categories, descriptions, and complete supported settings. There are 24 presets across Mirror, Radial, and Experimental categories. `applyPreset()` applies a normalized state through the shared state path without replacing the loaded source image. Legacy presets migrate to `geometryId: mirror`, while normalization accepts any registered geometry for future presets.

Smart Randomizer supports Subtle, Balanced, and Wild strengths plus Everything, Geometry Only, Transform Only, and Unlocked Values scopes. Six session-only locks cover Geometry, Segments, Rotation, Zoom, Source Position, and Center Position. Locks are preserved after candidate generation and before control clamping; the validated state then uses one `applyState()` call, one scheduled render, and at most one history commit.

Geometry-specific guidance belongs in the registered geometry's compact `settings.randomizer` metadata rather than conditionals in `GeometryManager` or the shared renderer. The current convention supports strength weights, preferred segment sets, zoom ranges, a density threshold, and optional center reach. Shared helpers own centered weighting and simple correlations such as dense-geometry zoom limits and reduced source reach when center offsets are already strong. Mutate remains the small related-change action and preserves the current geometry.

The Procedural Preset Generator is a higher-level recipe layer, not a replacement for Smart Randomizer. Its eight style profiles choose a plausible geometry, transform band, optional Pattern/Polar/Crystal stages, bounded Effect Stack, and palette before filling concrete values. Three intensity levels adjust activation and safe ranges without making every field equally random. Pattern, Polar, Crystal, and Effects include toggles are session-only UI state; when a stage is excluded, its current rendering state is carried into the generated candidate. Generated animation settings are left unchanged and playback never starts automatically. The generator applies one complete candidate through `applyState()` and commits one ordinary history entry, so generated results remain compatible with existing presets and Favorites.

### History

History stores up to 80 entries containing the rendering `getState()` plus a painting stroke index when applicable. Duplicate entries are not stored, a new action after Undo discards the Redo branch, and grouped gestures create one undoable action. Slider and numeric edits commit on change; preview, navigator, and radial-origin drags commit on release; wheel zoom is grouped after 220 ms without a new wheel event; one completed painting drag commits one complete stroke. Presets, Smart Randomizer, Mutate, and Procedural Preset Generator each apply one complete candidate and commit at most one rendering-history entry. Loading a new image resets controls and starts a new history session.

### Export

Export renders through the active renderer into a temporary export canvas. Imported images use the original-resolution source path; Painting Mode uses the active painting canvas so the export follows the painted preview source. PNG and JPEG, automatic/original-derived, fixed, and custom sizes, aspect-ratio locking, JPEG quality/background settings, sanitized filenames, `canvas.toBlob()`, native Save As when available, and an object-URL download fallback remain supported. PNG transparency is preserved, including radial circular-crop transparency.

### Workspace UI architecture

The UI uses the dark “Optical Instrument” design system and a workflow-based three-region shell:

- A persistent global toolbar owns Open Image, filename/status, Undo, Redo, Randomize, Reset, and Export Image.
- The supplied traced SVG emblem is embedded as the primary app-bar logo; keep its accessible name and exact source artwork intact.
- An icon-and-label navigation rail owns Source, Symmetry, Paint, Animation, Effects, and Library workspace selection.
- The canvas is the flexible center and visual priority. Its preview-only helpers are Show Original, preview background, and Focus Canvas.
- A spectral aperture ring around the canvas visualizes segment interval and rotation without becoming rendering or history state.
- The right panel displays one workspace panel at a time. Each setting keeps its single existing DOM control and remains bound to the authoritative rendering state.
- Native `details` elements provide the reusable collapsible-group component. Core task groups open by default; guidance, optional render stages, advanced systems, and unimplemented saved-work areas default closed. Closing a group never disables or clears its contents.
- In Symmetry, preserve the core-first order: presets/exploration, geometry, symmetry transform, mode-specific radial origin, and reflection/crop. Present Pattern Pack, Polar, Crystal, and Advanced Layers afterward as optional progressive-disclosure stages.
- Pattern Pack should read as Family → Pattern → Variant when relevant → Use As, followed by nested Placement and Appearance & Parameters controls. Keep active-only generated numeric parameters inside the existing metadata-driven container and retain Reset Pattern in the same section.
- The Animation workspace uses the shared disclosure-group pattern for playback, motion channels, Organic Motion, and recording. Library Favorites use the existing disclosure-group pattern and shared renderer/state paths; Effects uses the Milestone #12 ordered stack.
- Outfit supplies interface hierarchy and IBM Plex Mono supplies measurement and output typography. Prism mint marks active creative controls; spectral amber marks optical/source indicators.

`activeWorkspace` is UI-only state. `setActiveWorkspace()` changes tab/tabpanel ARIA attributes, roving tab focus, and panel visibility only. It must never call `getState()`, `applyState()`, render-state reset, source loading, painting reset, history commit, export, or animation playback APIs. Workspace changes must not become Undo entries or interrupt playback/recording.

Control ownership follows user workflow:

- **Source:** compact source-intake guidance, one primary Source Transform group for Zoom/source X/Y/centering, and Source Navigator.
- **Symmetry:** presets/exploration, compatibility mode, geometry, segments, rotation, radial origin, reflection/crop, then optional Pattern Pack, Polar, Crystal, and advanced layers.
- **Paint:** source-canvas creation/enabling and all brush controls.
- **Animation:** Play, Pause, Stop, global speed, opt-in motion channels, compact Organic Motion controls, and preview Recording.
- **Effects:** addable ordered post-processing entries with enable/disable, expand/collapse, settings, reorder, duplicate, remove, and Reset Stack.
- **Library:** export settings and the Favorites Gallery share the workspace while the quick Export action remains global; deeper project/state management stays in a collapsed Saved Work group reserved for Milestone #19.

At 900 px and wider, the application is a full-height rail/canvas/panel grid and the panel scrolls independently. Below 900 px, the canvas and active panel stack and the six workspace destinations move into a fixed bottom dock. Below 700 px, the persistent global actions form one compact six-action row. Focus Canvas hides the rail and panel without removing them from state; Show Controls or Escape restores the same active workspace.

## 4. Roadmap

Milestone identity and execution priority are separate documentation concepts:

- **Milestone ID:** the permanent historical identifier for a feature. It never changes after assignment.
- **Execution priority:** the current recommended order for unfinished work. It may change without renumbering any milestone.

The geometry-first strategy remains active: build visual finishing, coordinate transforms, sacred geometry, and mathematical/procedural Pattern Packs before motion, live input, and platform expansion.

### Completed milestone IDs

- Milestone #1 — Source Navigator — implemented
- Milestone #2 — Live Animation — complete
- Milestone #7 — Organic Motion Engine — implemented; runtime browser QA pending
- Milestone #8 — Layered Symmetry Engine — complete
- Milestone #9 — Symmetry Painting — complete for the current single-source brush scope
- Milestone #10 — Geometry Library — complete
- Milestone #11 — Smart Randomizer — complete
- Milestone #12 — Effect Stack — complete
- Milestone #14 — Polar Engine — complete
- Milestone #13 — Crystal Mode — complete
- Milestone #20 — Sacred Geometry Vector Mask Library — complete
- Milestone #21 — Mathematical Curve Library — complete
- Milestone #22 — Fractal Generator Pack — complete

### Core Visual Foundation — Priorities 2–5

- Priority 2 — Milestone #6 Palette & Post-Processing Foundation — complete
- Priority 3 — Milestone #12 Effect Stack — complete
- Priority 4 — Milestone #13 Crystal Mode — complete

### Pattern and Sacred Geometry Expansion — Priorities 6–11

- Priority 6 — Milestone #20 Sacred Geometry Vector Mask Library — complete
- Priority 7 — Milestone #21 Mathematical Curve Library — complete
- Priority 8 — Milestone #22 Fractal Generator Pack — complete
- Priority 9 — Milestone #23 Number-Theory Pattern Pack
- Priority 10 — Milestone #24 Tiling & Tessellation Pack
- Priority 11 — Milestone #25 Wave-Interference Pattern Pack

These milestones validate and extend one shared Pattern Pack architecture. Milestone #14 Polar Engine and Milestone #13 Crystal Mode precede them in priority so coordinate-transformation and faceted-geometry foundations are available first.

### Exploration and Generative Creativity — Priorities 12–15

- Priority 12 — Milestone #4 Compare & Evolution Mode
- Priority 13 — Milestone #5 Favorites Gallery
- Priority 14 — Milestone #15 Evolution Lab
- Priority 15 — Milestone #18 Procedural Preset Generator

Milestone #11 supplies guided variation before side-by-side and multi-generation exploration. Milestone #5 establishes reusable bounded saved states before Milestone #18 relies on save/favorite support.

### Motion and Animation — Priorities 16–19

- Priority 16 — Milestone #2 Live Animation — complete
- Priority 17 — Milestone #7 Organic Motion Engine
- Priority 18 — Milestone #3 Discovery Mode
- Priority 19 — Milestone #17 Cinematic Animation

Milestone #2 establishes playback before Milestone #7 adds natural drift. Milestone #3 depends on both foundations, and Milestone #17 follows as the advanced timeline and keyframe system.

### Live Input and Platform Features — Priorities 20–21

- Priority 20 — Milestone #16 Live Camera Studio
- Priority 21 — Milestone #19 Workspace System / Workspace and Project System Expansion

Live input follows the visual and motion systems it will exercise. Milestone #19 remains a late-stage persistence and project-management milestone distinct from the completed workspace UI reorganization.

`MILESTONE_ROADMAP.md` is authoritative for permanent IDs, current priorities, dependencies, status, architecture notes, and acceptance guidance. Existing DOCX prompt packs already use the permanent original milestone IDs.

## 5. Standard development workflow

1. Read `PROJECT_NOTES.md`.
2. Read this manual.
3. Create or verify a backup of the project.
4. Implement one milestone only.
5. Manually test the feature and the regression checklist.
6. Update `PROJECT_NOTES.md` with implementation details, tests, and limitations.
7. Report changes, tests, and remaining issues.

## 6. Milestone template

Each milestone should define:

- Goal
- Requirements
- Codex prompt
- Files expected to change
- Manual testing
- Known risks
- Completion criteria
- Recommended next milestone

## 7. QA requirements

The baseline manual QA checklist is:

- Load JPG.
- Load PNG.
- Drag an image onto the app.
- Use wheel zoom.
- Apply every preset.
- Use Randomize.
- Use Mutate.
- Switch through every registered geometry for landscape, portrait, and square sources.
- Confirm geometry switching participates in Undo/Redo and Reset.
- Exercise the Milestone #12 Effect Stack: add, enable/disable, expand/collapse, tune, reorder, duplicate, remove, Reset Stack, full Reset, Undo/Redo, legacy migration, preview/export parity, alpha preservation, and browser console cleanliness.
- Test Undo.
- Test Redo.
- Use Reset Controls.
- Export PNG.
- Create a blank painting canvas, enable/disable Painting Mode, select each brush, adjust size/opacity, and complete a physical desktop stroke.
- Verify one-drag/one-history-entry behavior, eraser replay, imported-image painting, and painted export matching the preview.
- Switch through Source, Symmetry, Paint, Animation, Effects, and Library; confirm exactly one panel is visible and artwork state does not change.
- Use arrow keys through the workspace rail and verify visible focus plus selected/current semantics.
- Collapse and reopen important and advanced groups; confirm settings and enabled state are unchanged.
- Switch workspaces during long-running or future animation operations and confirm the operation is not interrupted.
- For Live Animation and Organic Motion, verify Play/Pause/Stop, each supported target and style, unchanged base controls, one-step history for setting edits, no frame history, exact base restoration after Stop, and continued base editing while playing.
- Where supported, record the animated preview, stop cleanly, inspect the downloaded media, and verify visible failure messaging when `captureStream()` or `MediaRecorder` is unavailable or the canvas is not origin-clean.
- Resize the browser.
- Check the browser console.

For milestone-specific acceptance tests and the regression checklist required after every future milestone, consult `MILESTONE_ROADMAP.md` and the detailed prompt pack in `docs/`.

## 8. Performance guidance

- Avoid unnecessary renders.
- Cache reusable objects.
- Keep the UI responsive.
- Optimize only after correctness is established.
- Preserve rendering quality.
- Avoid unnecessary canvas copies; pass reusable pixel buffers between layers where practical.
- Skip disabled or inapplicable layers before allocating output buffers.
- Cache geometry plans by dimensions and relevant settings, and reuse the manager's mapping object inside per-pixel loops.
- Remember that all registered geometry types loop over output pixels on the main thread; large previews and exports can be CPU- and memory-intensive.

## 9. UI principles

- Preserve the Optical Instrument palette and optical-equipment visual vocabulary.
- Preserve the traced emblem as the primary brand mark; presentation-level sizing and contrast may adapt responsively without altering the vector artwork.
- Consistent spacing.
- Use Outfit for hierarchy and IBM Plex Mono only for measurements, filenames, and output values.
- Minimal dialogs.
- Responsive layout.
- Stable control locations.
- Keep the canvas visually dominant.
- Keep the spectral aperture ring truthful to segment count and rotation; it must not create or mutate renderer state.
- Keep global actions in the persistent toolbar and avoid duplicating them in workspace panels.
- Organize controls by user workflow: Source, Symmetry, Paint, Animation, Effects, or Library.
- Do not create a new top-level panel until the feature has been evaluated against the six existing workspaces.
- Use the shared workspace tab, panel, disclosure-group, and empty-state patterns instead of duplicating navigation or section markup.
- Keep advanced and experimental groups collapsed by default unless frequent use justifies otherwise.
- Keep Symmetry Layers compact, discoverable, and consistent with the existing dark creative-tool interface.
- Keep the Geometry selector and description compact; reserve the existing placeholder for future generator-specific settings.
- Preserve the Optical Instrument design and responsive workspace hierarchy unless a new requirement supersedes them.

## 10. Feature backlog

- Discovery Mode
- Evolution Tree
- Favorites Gallery — implemented in Milestone #5; bounded browser-local/session-only snapshots remain distinct from project save/load
- Live Morph
- Organic Motion — implemented in Milestone #7; browser QA and future motion extensions remain separate work
- Symmetry Painting — complete in Milestone #9
- Blend Modes
- Recursive and iterative pattern systems — see Milestone #22, Fractal Generator Pack
- Polar Engine — complete in Milestone #14
- Crystal Mode — complete in Milestone #13
- Live Webcam
- Video Support
- Plugin API

## 11. Changelog guidance

For every milestone, record its permanent milestone ID, date, summary, files changed, tests performed, and known issues. Never record the current execution-priority number as though it were the milestone ID. Keep the record in `PROJECT_NOTES.md` so the current project state remains easy to audit. Milestone #20 confirms that a metadata-first vector Pattern Pack can share one bounded renderer across overlays, alpha masks, non-destructive painting guides, preview, and export without becoming a runtime plugin system.

## 12. Lessons learned

Record prompting improvements, Luna-versus-Terra observations, architectural decisions, and recurring pitfalls. In particular, note decisions that protect state synchronization, history grouping, renderer separation, export fidelity, and real-browser QA coverage. Milestone #8 confirms that separating layer lifecycle and applicability from the existing Mirror/Radial formulas creates a safer foundation without requiring visual redesign or premature multi-layer composition. Milestone #10 confirms that geometry selection, plan generation, coordinate transforms, and pixel sampling can be separated without duplicating preview, export, layer, or painting paths. Milestone #11 confirms that small geometry-owned profiles plus shared weighting/correlation helpers can guide useful variation while preserving one authoritative state application, one render, one history entry, deterministic presets, and source safety. Milestone #12 confirms that a small definition registry plus one shared in-place pipeline can provide ordered, repeatable, alpha-preserving effects without duplicating preview/export paths; stack operations remain ordinary serializable state so history can restore the complete chain. Milestone #14 confirms that a bounded, reusable post-geometry pixel stage can add distinct circular transformations while preserving the existing Layer Manager, Effect Stack, preview/export parity, source immutability, and fast neutral behavior. Milestone #13 confirms that a second bounded pixel stage can remain independently mode-focused while sharing the same ImageData handoff, reusable buffer strategy, alpha rules, and preview/export route. Milestone #2 confirms that reversible motion settings can remain inside authoritative state while elapsed phase and playback stay session-only, allowing one shared renderer to serve still preview, animated preview, export, and canvas recording without frame-by-frame history.

## 13. Git workflow

After a development task is complete and validated:

1. Inspect `git status` and review the diff.
2. Do not stage temporary files or timestamped backups.
3. Update the relevant project documentation.
4. Commit the completed logical change with a concise descriptive message.
5. Push the commit to `origin/main`.

Do not make intermediate commits while a requested change is broken or incomplete unless explicitly asked. Do not automatically rewrite, squash, force-push, or otherwise alter published history.

## Appendix: recommended development cycle

Backup → Read `PROJECT_NOTES.md` → Read the Manual → Select one milestone → Implement → Manual QA → Update `PROJECT_NOTES.md` → Archive/Commit → Select the next milestone
