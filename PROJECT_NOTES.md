# Kaleidoscope Image Lab - Current Project Notes

Last updated: 2026-08-18

## 1. Purpose

Kaleidoscope Image Lab is a self-contained browser image editor for transforming a loaded image into reusable mirror, tessellation, radial, organic, and crystalline compositions. It is a focused creative tool centered on one square preview, editable source positioning, reversible history, and PNG/JPEG export.

The application has two rendering modes:

- **Mirror Lab:** the original mirror-based renderer with alternating, left-copy, right-copy, and pinwheel behavior.
- **Radial Kaleidoscope:** a true angular-wedge renderer that repeats image detail around a configurable radial origin, with optional alternating reflection and circular cropping.

## 2. File structure

The project intentionally has no package manager, framework, dependency manifest, build configuration, or external runtime dependency.

```text
/Users/stephendavis/Documents/Prism LAb - Kaleidoscope Image Lab/
  kaleidoscope-image-lab.html                         Browser-ready application
  PROJECT_NOTES.md                                    This document
  DEVELOPMENT_MANUAL.md                               Long-term design and development standards
  MILESTONE_ROADMAP.md                                 Milestone order, status, and QA summary
  kaleidoscope-image-lab.html.backup-2026-07-17       Preset-library backup
  kaleidoscope-image-lab.html.backup-2026-07-17-radial-interactions
                                                        Radial-interaction backup
  kaleidoscope-image-lab.html.backup-20260717-214058-layered-symmetry
                                                        Milestone #8 pre-edit backup
  kaleidoscope-image-lab.html.backup-20260717-223247-milestone9-pre-painting
                                                        Milestone #9 pre-edit backup
  kaleidoscope-image-lab.html.backup-20260717-230534-milestone10-pre-geometry
                                                        Milestone #10 pre-edit backup
  kaleidoscope-image-lab.html.backup-20260718-000621-pre-workspace-ui
                                                        Workspace UI pre-edit backup
  kaleidoscope-image-lab.html.backup-20260802-014914-milestone11-pre-smart-randomizer
                                                        Milestone #11 pre-edit backup
  kaleidoscope-image-lab.html.backup-20260803-000000-milestone13-pre-crystal-mode
                                                        Milestone #13 pre-edit backup
```

`kaleidoscope-image-lab.html` is the authoritative application file. It is a standalone wrapper whose outer document contains an iframe with an escaped `srcdoc` application. The wrapper uses a full-viewport iframe (`width: 100%`, `max-width: none`, `height: 100dvh`) and `sandbox="allow-scripts allow-downloads"`.

The embedded application contains its own HTML, scoped CSS, and JavaScript. There are no separate source modules to synchronize.

## 3. Rendering and state architecture

### Application structure

- The embedded app is a single IIFE scoped to `#kaleidoscope-image-lab`.
- All DOM access is scoped to the application root.
- Rendering, controls, image decoding, history, and export are implemented with native browser APIs.
- `requestRender()` coalesces updates through `requestAnimationFrame`.
- The preview renderer works on a 1200 × 1200 internal canvas. CSS scales it to the available workspace.

### Rendering state

`getState()` captures the complete reversible rendering state:

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
  mirrorLayerEnabled,
  radialLayerEnabled,
  mode,
  renderMode,
  geometryId,
  crystal: {
    mode,
    facetSize,
    rotation,
    strength,
    spread,
    direction,
    reflection
  },
  animation: {
    speedMultiplier,
    channels: {
      [channelId]: { enabled, amount, speed }
    }
  },
  effectStack: [
    { id, type, enabled, settings }
  ]
}
```

- `renderMode` is `mirror` or `radial`.
- `mode` is `alternating`, `left`, `right`, or `pinwheel`; it is primarily used by Mirror Lab.
- `offsetX` and `offsetY` control the source-image position.
- `centerX` and `centerY` control the radial origin in Radial Kaleidoscope.
- `mirrorLayerEnabled` and `radialLayerEnabled` control whether the corresponding symmetry layer may render.
- `geometryId` is the active registered geometry and defaults to `mirror` for legacy state.
- `effectStack` is the ordered, neutral-by-default, reversible post-processing state. Each entry has a stable instance id, registered effect type, enabled state, and normalized serializable settings. Legacy flat `effects` snapshots migrate into equivalent active entries; presets without a stack fall back to an empty neutral stack.
- `crystal` is the nested, neutral-by-default Crystal Mode state. Legacy states and presets without Crystal data safely use Off and the default settings.
- `animation` stores reversible channel enablement, amount, speed, the global speed multiplier, and the nested Organic Motion settings. Playback state and elapsed phase remain session-only and are not written to history. Basic channels and Organic Motion both temporarily modulate the existing render-state targets without changing source pixels or base controls.
- Preview-only values such as preview background, original-preview visibility, distraction-free layout, filename, export settings, animation playback phase, and recording status are not part of rendering history.

`applyState()` synchronizes all paired slider/number controls, checkboxes, mode buttons, mode-specific visibility, and schedules a render.

### Source data and performance

- Supported image types are PNG, JPEG/JPG, WebP, GIF, BMP, and AVIF.
- The original decoded image is retained for export.
- Interactive rendering uses a rotated cached preview source capped at `previewSourceLimit = 1400` pixels on its longest side.
- The hard-coded `importRotation = -90` rotates decoded images 90 degrees left while creating the source canvas. This is deliberate product behavior and must be rechecked against real user images before changing.
- Mirror and radial rendering both loop over output pixels on the main thread. Large previews and exports can be CPU-intensive.

### Renderer routing

- `GeometryManager` owns geometry registration, lifecycle, active selection, cached segment plans, coordinate transforms, and the shared source-sampling loop.
- The legacy Mirror and Radial formulas now live inside the registered `mirror` geometry so old modes and presets retain their prior output.
- `LayerManager` executes the ordered layer pipeline for both preview rendering and export.
- Mirror and Radial remain independent compatibility layers. Each delegates its render operation to `GeometryManager` rather than owning geometry math.
- The current Mode supplies layer applicability, so Mirror and Radial remain available without stacking their formulas and changing existing visuals. Future layers can be inserted without rewriting `drawActiveRenderer()`.
- Layer output is passed as pixel data to the next applicable enabled layer. With the current one-layer active path, only one output buffer is created before the final `putImageData()` call.
- The shared post-geometry route is Geometry / Layer Manager → Polar Engine → Crystal Mode → Effect Stack → target context. Preview, fallback rendering, and original-resolution export all use this route.
- Live Animation derives a temporary render-state snapshot from the current base state immediately before the shared renderer. It does not write modulated values back to controls or source pixels, and the 30 fps-bounded playback loop reuses `requestRender()` rather than adding a second renderer. Source X/Y motion feeds the existing `offsetX`/`offsetY` sampling path and stops at the unchanged base source position. Organic Motion extends this same snapshot with bounded Drift, Orbit, Breathe / Pulse, Wobble, and deterministic layered-sine Wander behaviors.

### Geometry Library

- Eight registered geometries are available: Mirror, Triangle, Hexagon, Starburst, Spiral, Flower, Crystal, and Snowflake.
- Every geometry exposes `id`, `name`, `description`, `settings`, `initialize()`, `generateSegments()`, `transformSegment()`, and `dispose()`.
- `GeometryManager.register()` validates this contract, and new geometry types can be added by registering another independent implementation.
- `generateSegments()` creates a reusable render plan for the current dimensions and relevant controls. One plan per geometry is cached and reused until a cache-affecting control changes.
- `transformSegment()` writes normalized source coordinates into a reusable object, avoiding per-pixel object allocation.
- Geometry and layer lifecycle initialization happens at startup, and both managers dispose registered implementations on `pagehide`.
- Preview rendering, original-resolution export, wheel-centered source mapping, and Symmetry Painting all use the active Geometry Manager mapping.
- The Geometry UI provides the active selector, live description, and a stable placeholder for future geometry-specific settings.

### Palette & post-processing pipeline

- `PostProcessingPipeline` runs the ordered `effectStack` after the active geometry layer and before the final `putImageData()` for both preview and export.
- Brightness, contrast, saturation, monochrome, vignette, lightweight glow, grain, and two-color gradient mapping use registered effect definitions with defaults, normalization, neutral checks, and pipeline apply methods. The shared buffer is mutated in place, so disabled or neutral entries are skipped without per-effect canvas copies.
- An empty stack, disabled entries, and neutral settings preserve the pre-effects output as closely as possible; order is the visible top-to-bottom processing order.
- RGB values are adjusted without changing alpha. Transparent pixels remain transparent and their RGB channels are cleared rather than colorized.
- The optional glow is deliberately conservative: it is a highlight lift, not a full-resolution blur/bloom pass, to protect interactive responsiveness.

### Polar Engine

- `PolarEngine` is an optional reusable pixel stage between the active Geometry/Layer Manager output and `PostProcessingPipeline`.
- Preview and original-resolution export call the same `drawActiveRenderer()` and Layer Manager path, so Polar settings are applied consistently at both sizes; fallback rendering also passes through Polar before effects.
- Supported modes are Off, Polar Wrap, Radial Tunnel, Ring Repeat, and Vortex. Off returns the geometry buffer without allocating or iterating a Polar buffer.
- The engine uses one reusable `Uint8ClampedArray` sample buffer per canvas size, bounded center/scale/strength/repeat values, and nearest-neighbor sampling. It preserves source alpha and does not mutate source pixels.
- Polar state is stored under `polar` in `getState()` with mode, center, rotation, strength, scale, repeat count, and direction. Legacy states and presets without Polar data safely use Off and neutral defaults.
- Mode changes, completed Polar slider/number edits, Reset Polar, full Reset, and Undo/Redo use the existing reversible history path. Presets, Smart Randomizer, and Mutate intentionally carry the current Polar state without changing it during this milestone.

### Crystal Mode

- `CrystalModeStage` is a reusable optional pixel stage after `PolarEngine` and before `PostProcessingPipeline`. It supports Off, Facet, Prism, Shard, and Crystal Bloom through bounded, nearest-neighbor ImageData transforms.
- Facet, Prism, Shard, and Crystal Bloom use separate mode-specific math for angular facets, crisp refracted bands, deterministic displaced shards, and radial organic facets. Off returns immediately without allocating or iterating a Crystal buffer.
- The stage reuses one `Uint8ClampedArray`, avoids per-pixel object allocation and temporary canvases, clamps samples to the frame to avoid blank gaps, and copies sampled alpha without changing source pixels.
- The compact Crystal Mode disclosure lives in the existing Symmetry workspace. It exposes mode, facet size/density, rotation, strength, spread/displacement, relevant direction/reflection controls, and Reset Crystal.
- Crystal settings participate in `getState()`/`applyState()`, `statesMatch()`, presets, full Reset, Undo, and Redo. Mode changes and completed control edits create one history entry; Smart Randomizer and Mutate preserve the current Crystal state, while older presets without Crystal data remain Off.

### Layered Symmetry Engine

- A compact Symmetry Layers panel displays Mirror Layer and Radial Layer entries.
- Each entry supports Enable/Disable, Collapse/Expand, and a placeholder settings area for future layer-specific controls.
- Layer enablement is part of `getState()` and therefore participates in Undo/Redo, presets through the existing state path, Reset, and fresh image sessions.
- Disabled active layers are skipped efficiently and fall back to a centered, aspect-preserving source image instead of producing a blank preview/export.
- Layer initialization is centralized through `layerManager.initialize()`; disposal is available for future resources even though the current layers do not allocate external resources.

## 4. Current implemented features

### Image workflow

- Open Image control in the top bar.
- Empty-state Open Image button using the same native file input.
- Supported-image drag-and-drop onto the preview workspace, with valid-drop feedback and an in-app error for unsupported file drops.
- Clipboard image paste through Ctrl+V/Cmd+V. Only supported image clipboard items are intercepted; plain-text pastes remain available, including in numeric inputs.
- File picker, drop, and paste all use the shared `loadFile()` pipeline.
- Current filename display with truncation.
- New image loads clear unfinished drag/wheel state, reset rendering controls, clear stale preset selection, and start a fresh history session.

### Shared controls

- Segments: 2–32, step 1, slider and numeric input.
- Rotation: -180–180 degrees, step 45, slider and numeric input.
- Rotate left 45 deg and Rotate right 45 deg buttons.
- Zoom: 0.35–3.00, step 0.01, slider and numeric input.
- Horizontal source and Vertical source: -100–100, slider and numeric input.
- Center Source button.

### Symmetry Painting

- Painting Mode owns an internal canvas source and routes it through the same `drawActiveRenderer()` and `LayerManager` path as imported images.
- Imported images remain available as the base source while painting; New Blank Canvas creates a transparent 1200 × 1200 source.
- Controls provide Pencil, Marker, Soft Brush, Eraser, adjustable size, opacity, and color.
- The painting canvas stores replayable stroke records and rebuilds from its base image plus the active stroke prefix for Undo/Redo.
- Pointer coordinates reuse the existing preview-to-source mapping, including radial-origin mapping, so painting follows the active symmetry engine rather than a separate drawing renderer.

### Mirror Lab

Mirror Lab is the default renderer. Its mode buttons are:

- Alternating
- Left copy
- Right copy
- Pinwheel

These controls are hidden when Radial Kaleidoscope is active.

### Radial Kaleidoscope

- Center X and Center Y: -100–100 radial-origin controls.
- Alternate Mirror: reflects adjacent wedges when enabled; repeats them directly when disabled.
- Circular Crop: makes pixels outside the centered circular crop transparent.
- Center Radial Origin button.
- Visible radial center marker when an image is loaded.
- Interaction guidance in the Radial Settings section.

Current radial interactions are intentionally separated:

- Normal preview drag pans the source image by changing Horizontal/Vertical Source.
- Dragging the visible center marker moves the radial origin by changing Center X/Y.
- Holding Option/Alt while dragging the preview moves the radial origin instead of panning the source. Meta-key detection is also supported for platform compatibility.
- Wheel zoom remains pointer-centered and does not change Center X/Y; it may adjust source offsets to keep the pointer position stable.

Mirror Lab keeps its existing behavior: normal preview dragging pans the source image.

### Source Navigator

- Collapsed by default and hidden until an image is loaded.
- Uses the oriented cached preview source and preserves its aspect ratio with letterboxing.
- Teal circle marks Horizontal/Vertical Source.
- Orange diamond marks Center X/Y in Radial Kaleidoscope.
- Pointer and touch dragging update the mapped values and commit at most one history state per gesture.

### Preview helpers

- Show Original press-and-hold preview, including Space-hold when the preview is focused.
- Checkerboard, black, white, and neutral-gray preview backgrounds. These are CSS/preview-only and do not affect exports.
- Hide Controls / Show Controls distraction-free toggle.
- Escape restores controls from distraction-free mode while preserving editor state and sidebar scroll position.

### Presets and creative tools

The Presets and Exploration group contains the preset menu, Smart Randomizer controls, compact locks, and Mutate. The global Randomize action remains in the app toolbar.

The preset library is centralized in one validated `presets` array. Every preset has:

- Stable unique `id`.
- User-facing `name`.
- Category: `mirror`, `radial`, or `experimental`.
- Short `description`.
- Complete supported rendering state under `settings`.

There are 24 presets, eight per category:

- **Mirror:** Bilateral Portrait, Vertical Totem, Horizontal Reflection, Four-Way Cross, Eight-Way Mirror, Rorschach, Mechanical Mask, Symmetry Fold.
- **Radial:** Sixfold Mandala, Eightfold Glass, Twelvefold Burst, Mechanical Bloom, Crystal Flower, Pinwheel Tunnel, Insect Eye, Sun Wheel.
- **Experimental:** Alien Iris, Black Hole, Stained Glass, Fractured Halo, Kaleido Tunnel, Orbital Ring, Bio-Mechanical Flower, Cosmic Compass.

`validatePresetLibrary()` checks metadata, unique ids, allowed categories, and required state keys. `normalizePresetSettings()` clamps values to existing control ranges. `applyPreset()` applies the state through the shared `applyState()` path without changing the loaded source image.

Smart Randomizer provides three strengths: Subtle, Balanced (default), and Wild. Subtle usually preserves geometry and makes small changes around the current state; Balanced favors clearly different but controlled compositions; Wild uses broader geometry, repetition, transform, and center ranges while remaining inside supported control limits.

The four scopes are Everything, Geometry Only, Transform Only, and Unlocked Values. Everything may also vary compatible Mirror mode or Radial reflection/crop options. Unlocked Values is restricted to the six lockable groups. Geometry Only changes geometry and repetition; Transform Only preserves geometry and repetition while changing rotation, zoom, source position, and center.

Six session-only locks protect Geometry, Segments, Rotation, Zoom, Source Position, and Center Position. Lock state is deliberately excluded from `getState()`, presets, rendering history, and export. It survives source changes and Reset while the app remains open, and changing a lock never renders or creates an Undo entry.

Each registered geometry carries a compact `settings.randomizer` profile containing strength weights, preferred segment sets, zoom bounds, density thresholds, and optional center reach. Shared randomizer utilities apply centered weighting, geometry-change probability, dense-geometry zoom limits, and reduced source reach when center movement is already strong. Mirror favors common even counts; Triangle, Hexagon, Starburst, Spiral, Flower, Crystal, and Snowflake use their milestone-specific readable ranges.

Randomize builds one complete candidate, preserves locks, clamps through the existing control metadata, validates the geometry id, and calls the shared `applyState()` path once. The action schedules one render and commits at most one history entry. Mutate remains a separate small-change action that preserves the selected geometry. All built-in presets remain deterministic and are migrated to `geometryId: mirror` at startup; preset normalization continues to accept future registered geometries.

The UI intentionally remains a simple select menu with category optgroups. There is no thumbnail browser, search, favorites, modal, or category-tab interface yet.

## 5. UI layout and controls

### Visual direction

- “Optical Instrument” creative-tool interface inspired by apertures, prisms, ground glass, and darkroom equipment.
- Optical black and blue-carbon surfaces with prism mint for active controls, spectral amber for optical/source indicators, and coral for errors.
- Outfit carries interface hierarchy; IBM Plex Mono is reserved for measurements, filenames, and output values.
- The supplied `Images and logos/Kaleidoscope-Image-Lab-Traced-Emblem.svg` is embedded byte-for-byte as the primary app-bar logo so it remains available inside the sandbox without a runtime file request.
- A thin spectral aperture ring around the canvas visualizes the current segment count and symmetry rotation.
- Restrained ambient grain, depth, and GSAP entrance/scroll motion keep the artwork dominant without changing renderer state.
- CSS is scoped under the application root.

### Top app bar

Contains:

- Kaleidoscope Image Lab title and subtitle.
- Open Image action.
- Current filename/status.
- Undo and Redo.
- Randomize.
- Reset.
- Export Image action.

### Workspace system

The UI is organized around the creative workflow rather than one continuous controls list:

- A compact left rail switches Source, Symmetry, Paint, Animation, Effects, and Library.
- The canvas remains the center and largest visual region. Show Original, preview background, and Focus Canvas stay directly above it as preview-only helpers.
- The right controls panel displays exactly one workspace at a time.
- Workspace switching changes UI visibility only. It does not call rendering, history, source loading, or painting-reset paths and does not alter `getState()` or `applyState()`.
- Native `details` groups provide consistent progressive disclosure. Core task groups open by default; import guidance, Pattern Pack, Polar, Crystal, Advanced Layers, and future Saved Work start collapsed. Closing a group never disables or clears its settings.

Major feature locations are:

- **Source:** compact import guidance plus one open Source Transform group for zoom, Horizontal/Vertical Source, and Center Source; Source Navigator stays collapsed until needed.
- **Symmetry:** presets and exploration lead into geometry, symmetry transform, mode-specific radial origin, and reflection/crop. Pattern Pack, Polar, and Crystal are presented as optional collapsed stages; Advanced Layers remains last and collapsed. Randomize remains globally available.
- **Paint:** Painting Mode, New Blank Canvas, Pencil, Marker, Soft Brush, Eraser, size, opacity, color, and status. Painting is presented as editing the active source canvas.
- **Animation:** Playback contains Play, Pause, Stop, and a global speed multiplier. Motion Channels contains ten opt-in sine-wave modulators for Global Rotation, Zoom/Scale, Source X, Source Y, Center X, Center Y, Pattern Rotation, Pattern Scale, Polar Rotation, and Crystal Strength. Organic Motion is a compact disclosure for Source Position, Center Position, Global Rotation, Zoom/Scale, Pattern Rotation, Polar Rotation, and Crystal Strength, with style, amount, speed, and Orbit Radius where relevant. Recording captures the existing preview canvas at 30 fps through the browser-supported `MediaRecorder` container, so both motion layers are captured automatically.
- **Effects:** Add Effect control plus a compact ordered stack with enable/disable, expand/collapse, settings, Move Up, Move Down, Duplicate, Remove, and a nearby Reset Stack footer; preview background remains a preview-only aid.
- **Library:** Export Settings remains available alongside the open Favorites Gallery. Favorites are compact visual snapshots with restore, rename, delete, and inline-confirmed Clear All; the Project section provides complete versioned Save/Open/New actions.

At 900px and wider, the shell is full-height with an 88px icon-and-label rail, flexible canvas, and 310–380px independently scrolling controls panel. Below 900px, the canvas and active panel stack while workspace navigation becomes a fixed six-item bottom dock. Below 700px, the global toolbar compacts into one exact six-action row and hides filename/status text to preserve canvas priority. Focus Canvas hides only the rail and controls panel; Show Controls or Escape restores the selected workspace and its state.

### Favorites Gallery

Milestone #5 adds a compact Favorites Gallery inside the existing Library workspace. Add to Favorites captures the current composition through the existing `getState()` serializer, including source transform, geometry and symmetry, Pattern Pack, Polar, Crystal, Effect Stack, animation, and Organic Motion settings. Favorites also retain a conservative 1200px-longest-side source snapshot so the composition can be restored without storing rendered geometry caches; Painting Mode is represented by the current painted source pixels, while its replayable stroke history and brush settings are not serialized because the existing state path does not safely include them.

Each favorite stores a 180px thumbnail generated through the shared preview renderer plus a compact WebP source snapshot, with PNG fallback. Gallery storage is capped at 24 items. The app attempts browser-local `localStorage` persistence and gracefully falls back to session-only favorites when the sandboxed iframe cannot access browser storage or the storage quota is exceeded. Restoring stops playback, applies the saved composition through `applyState()`, refreshes the shared preview/export path, and creates one ordinary history entry when the rendering state changes. Playback status, preview helpers, and project/workspace save data remain outside Favorite content.

## 6. History behavior

- History is an array of entries containing a `getState()` snapshot and a painting stroke index when applicable.
- The history limit is 80 states.
- Consecutive duplicate states are not stored.
- A new action after Undo discards the abandoned Redo branch.
- Undo and Redo buttons reflect availability and are also available through keyboard shortcuts:
  - Ctrl/Cmd+Z: Undo
  - Ctrl+Y: Redo
  - Ctrl/Cmd+Shift+Z: Redo
- Slider changes commit on `change`.
- Direct numeric changes commit on `change`.
- Mode switches, mirror/radial switches, rotation buttons, centering buttons, Reset Controls, presets, Randomize, and Mutate commit one state when changed.
- Preview source drags commit once on pointer release; cancellation restores the starting state.
- Radial-origin drags commit once on pointer release; cancellation restores the starting state.
- Wheel zoom changes are grouped into one history entry after 220ms without a new wheel event.
- Applying a preset is atomic from the user’s perspective: one `applyState()` call and at most one history entry.
- Enabling or disabling a symmetry layer commits one history entry; Undo/Redo restores both the toggle and rendered output.
- Geometry switching commits one history entry; Undo/Redo restores the selected geometry, description, and rendered output.
- One Smart Randomizer action applies one complete candidate and commits exactly one history entry; Undo/Redo restores the complete pre/post randomizer rendering state.
- Each completed effect setting edit commits one history entry. Add, remove, duplicate, reorder, enable/disable, Reset Stack, and full Reset each commit one stack-aware history entry; Undo/Redo restores ids, order, enabled states, and settings.
- Randomizer strength, scope, and lock changes are UI/session state and do not create rendering-history entries.
- Completed animation setting edits, including basic and Organic Motion enablement, style, Amount, Speed, and Orbit Radius, create ordinary history entries. Play, Pause, Stop, elapsed phase, and every rendered animation frame remain outside history, so playback never floods the 80-entry stack.
- A completed painting drag commits one history entry; Undo/Redo restores or removes the complete stroke, including eraser strokes, without splitting the drag into intermediate entries.
- Loading an image resets controls and creates a fresh history session; the loaded source image is not part of rendering history.

## 7. Image-loading behavior

All supported intake paths call `loadFile()`:

1. Validate MIME type or supported extension.
2. Decode with `createImageBitmap(file, { imageOrientation: 'none' })` when available.
3. Fall back to `createImageBitmap(file)`.
4. Fall back to an HTML `Image` element.
5. Rotate the decoded image 90 degrees left into a source canvas.
6. Cache a preview-sized source for interactive rendering.
7. Set the filename and default export name.
8. Reset controls and history, then render.

The source navigator uses the oriented cached preview source. Export reconstructs a full-resolution source from the original decoded image. When Painting Mode is enabled, the active painting canvas replaces `previewSource` for the same renderer path; loading a new image clears the prior painting source and starts a fresh session.

## 8. Export behavior

Export uses the active renderer and the original-resolution source path for imported images, or the active painting canvas when Painting Mode is enabled:

- `drawActiveRenderer()` renders to a temporary export canvas.
- Size choices: Automatic / Original-derived, 1080 × 1080, 2048 × 2048, 4096 × 4096, or Custom.
- Automatic uses a square side of `max(1200, fullSource.width, fullSource.height)`.
- Custom width and height accept whole numbers from 64 to 4096 pixels, with an enabled-by-default aspect-ratio lock.
- PNG retains transparency, including pixels outside a Radial Kaleidoscope circular crop.
- JPEG supports 60–100% quality, default 92%, and composites onto black, white, or custom-color backgrounds.
- The filename is sanitized and receives the correct `.png` or `.jpg` extension automatically.
- Export uses `canvas.toBlob()` and cleans temporary canvases and object URLs afterward.
- When available, `showSaveFilePicker()` provides a native Save As flow. Otherwise, the app uses a standard object-URL download link.
- The outer iframe includes `allow-downloads` so the fallback download is not blocked by the wrapper sandbox.
- Export controls are disabled while rendering/encoding and status text reports progress or failure.
- Canceling the native Save As picker does not trigger a second fallback download.
- Painted exports use the same symmetry renderer and settings as the preview, with the painting canvas pixels as the source.

Preview background, Show Original, distraction-free mode, and sidebar state do not affect exported pixels.

While animation is playing or paused, still export uses the current temporary modulated render state through the same `drawActiveRenderer()` path. Stopping animation returns preview and subsequent exports to the unchanged base composition.

The same `drawActiveRenderer()` and `PostProcessingPipeline` path is used for preview and export. PNG retains the post-processed alpha channel, while JPEG keeps the existing explicit background compositing step.
Crystal Mode runs on that same shared path, so PNG preview/export retains transparency and JPEG continues through the normal background compositing step.

## 9. Testing completed

### Static and runtime checks

- Current application loaded successfully in a localhost browser session with no console errors or warnings.
- Selector-to-markup wiring, renderer routing, state fields, history grouping, export settings, and responsive layout markers were inspected after the latest changes.
- Confirmed the preset library contains 24 unique ids with eight presets in each category.
- Confirmed all presets apply through the shared state path and use existing control ranges.
- Confirmed the updated radial interaction code has separate source-pan and radial-center branches, supports Option/Alt and Meta modifiers, and does not write Center X/Y from the wheel handler.

### Interactive tests completed

- Loaded a real PNG through the existing clipboard-image pipeline.
- Selected all 24 presets in sequence; all applied, all produced unique state fingerprints, and mode-specific controls matched the active renderer.
- Ran Bilateral Portrait → Mechanical Bloom → Undo → Redo → Alien Iris → Mutate.
- Confirmed Undo restored the prior preset state in one step and Redo reapplied it in one step.
- Confirmed the loaded source filename remained present through preset/history operations.
- Visually inspected non-blank Mirror and Radial previews.
- Invoked PNG export and confirmed the control returned to its idle state without console errors.
- Reloaded the latest radial-interaction build, loaded a real PNG, confirmed radial controls and center-marker guidance were visible, and confirmed no console errors.

### Milestone #8 QA completed

- Created `kaleidoscope-image-lab.html.backup-20260717-214058-layered-symmetry` before editing.
- Loaded a generated landscape PNG through the existing clipboard-image pipeline.
- Confirmed the Symmetry Layers panel renders Mirror Layer and Radial Layer with enable toggles, collapsible entries, status text, and placeholder settings.
- Switched Mirror Lab and Radial Kaleidoscope and confirmed the corresponding layer status and mode-specific controls updated.
- Disabled the active Radial Layer and visually confirmed the centered source fallback rendered without a blank canvas; re-enabled it and confirmed the rendered path resumed.
- Verified layer toggle Undo/Redo restored disabled and enabled states correctly.
- Applied all 24 presets, then ran Randomize and Mutate; no render or state errors appeared.
- Invoked PNG export after layer operations; export returned to idle with no error status.
- Checked the app at narrow and wide responsive viewports; the panel remained visible and usable.
- Browser console contained no warnings or errors during the milestone QA pass.

### Milestone #9 QA completed

- Created `kaleidoscope-image-lab.html.backup-20260717-223247-milestone9-pre-painting` before editing.
- Created a blank 1200 × 1200 canvas and confirmed Painting Mode, source navigator, filename, status text, and Undo/Redo wiring updated correctly.
- Confirmed brush selection, size, opacity, and color controls are present and paired range/number inputs stay synchronized.
- Toggled Painting Mode off and on, switched to Radial Kaleidoscope, and confirmed painting controls and radial controls remained compatible.
- Invoked Export Image from a blank painting source without an in-app error.
- Rechecked preset/randomize control wiring after the painting source changes.
- Browser console contained no new warnings or errors after the painting UI and source-path checks.

### Milestone #10 QA completed

- Implemented the reusable Geometry Library, Geometry Manager, eight generators, UI/state/history integration, legacy preset migration, geometry-aware Randomize, and shared preview/export/painting mapping without starting Milestone #11.
- Created `kaleidoscope-image-lab.html.backup-20260717-230534-milestone10-pre-geometry` before editing.
- Parsed all embedded script blocks after implementation and statically confirmed the eight registered geometry factories and centralized Geometry Manager.
- Loaded generated landscape (960 × 540), portrait (540 × 960), and square (720 × 720) PNG sources through the existing clipboard-image path.
- Ran Mirror, Triangle, Hexagon, Starburst, Spiral, Flower, Crystal, and Snowflake for every source aspect ratio. Sampled output hashes were distinct for all eight geometries in every aspect-ratio pass.
- Confirmed geometry descriptions update, geometry switching remains responsive while editing, and Triangle → Hexagon → Undo → Redo restores the expected geometry in each step.
- Confirmed Randomize selected bounded geometry-specific segment counts and switched from Mirror to Snowflake without an error.
- Loaded representative legacy Mirror and Radial presets; both mapped to Mirror geometry while retaining the correct Mirror/Radial compatibility mode.
- Disabled and re-enabled the active Radial Layer with Hexagon selected; the layer fallback/status and geometry selection remained intact.
- Invoked PNG export for Snowflake. The preview and export passed through the same 1200 × 1200 geometry output and produced matching sampled pixel hashes with no in-app error.
- Confirmed Reset Controls restores Mirror geometry, two segments, 1× zoom, and Mirror Lab.
- Checked 620 × 900 and 1200 × 800 viewports. Geometry remained visible; the layout changed from one column to the expected preview/sidebar grid.
- Final clean reload, square-image load, and Spiral switch completed with no in-app error and no browser console warnings or errors.

Files modified for Milestone #10:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `DEVELOPMENT_MANUAL.md`
- `MILESTONE_ROADMAP.md`

### Workspace UI reorganization QA completed — 2026-07-18

- Created and checksum-verified `kaleidoscope-image-lab.html.backup-20260718-000621-pre-workspace-ui` before editing.
- Compared the pre-edit and current markup inventories: every pre-existing interactive control id remains present exactly once, every script selector still resolves, and no live markup ids are duplicated.
- Loaded the application cleanly and visually confirmed the canvas is the dominant center region with a narrow workspace rail and contextual right panel.
- Switched through Source, Symmetry, Paint, Animation, Effects, and Library; each tab received selected/current semantics and exactly one workspace panel was visible.
- Used arrow-key navigation on the workspace rail and confirmed selected-workspace focus and ARIA state move together.
- Loaded generated PNG and JPEG sources through the supported clipboard-image path, verified source filename/state reset, Source Navigator visibility, and no console errors.
- Changed Zoom, Horizontal Source, and Vertical Source, switched workspaces, and confirmed the values remained synchronized and unchanged.
- Applied a preset, changed geometry, segments, and rotation, toggled a symmetry layer, and confirmed layer Undo/Redo worked while Paint remained the selected workspace.
- Created a blank 1200 × 1200 painting source, selected Pencil, Marker, Soft Brush, and Eraser, switched away and back, and confirmed the painting source, enabled state, and selected brush persisted.
- Invoked Randomize, Undo, and Redo from the global toolbar while Paint was active; rendering state changed/restored without changing the selected workspace or painting source.
- Collapsed and reopened Transform and confirmed settings remained unchanged; Advanced Layers disclosure state also persisted across workspace switches.
- Invoked 1080px PNG export from Library; the export control returned to idle with no status error, in-app error, warning, or console error.
- Used Focus Canvas and Show Controls; only the rail/panel visibility changed and the selected Library workspace and artwork state were retained.
- Measured layouts at 1200 × 800, 800 × 900, and 480 × 800. All had matching client/scroll widths and no horizontal overflow; wide layout used three regions, tablet stacked the horizontal rail/canvas/panel, and narrow layout used the 3 × 2 rail.
- Final clean desktop and narrow visual passes completed with no browser console warnings or errors.

### Milestone #6 QA completed — 2026-08-02

- Files changed: `kaleidoscope-image-lab.html`, `PROJECT_NOTES.md`, `DEVELOPMENT_MANUAL.md`, and `MILESTONE_ROADMAP.md`; timestamped application backup created separately.
- Created `kaleidoscope-image-lab.html.backup-20260802-224433-milestone6-pre-effects` before editing and checksum-verified it against the pre-edit application.
- Replaced the Effects empty state with compact Color, Light, Texture, and Gradient Map disclosure groups plus Reset Effects.
- Exercised Brightness, Contrast, Saturation, Monochrome, Vignette, lightweight Glow, Grain, Gradient Map enablement, both gradient colors, Swap Colors, Reset Effects, and full Reset.
- Confirmed effect slider/number synchronization, one-step Undo/Redo for effect edits, one-step Reset Effects, and neutral values after full Reset and new-image loading.
- Confirmed the shared pipeline is reached by preview and export through the existing `drawActiveRenderer()`/`LayerManager` route; PNG and JPEG export actions returned to idle with no in-app errors.
- Loaded a transparent RGBA PNG and a JPEG through the supported clipboard-image path. Radial Circular Crop plus Gradient Map visibly preserved checkerboard transparency around the crop.
- Switched all eight registered geometries, representative Mirror/Radial/Experimental presets, Randomize, Mutate, and Painting Mode without new console errors.
- Checked desktop, tablet (800 × 900), and narrow (480 × 800) layouts. The workspace stayed within its layout width and the Effects panel remained reachable in the narrow bottom-dock layout.
- Browser console review remained clean: no new errors or warnings.

Files modified for the workspace UI reorganization:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `DEVELOPMENT_MANUAL.md`

`MILESTONE_ROADMAP.md` was intentionally not modified because this reorganization is not a numbered rendering milestone.

### Optical Instrument visual redesign QA completed — 2026-07-26

- Created `kaleidoscope-image-lab.html.backup-20260726-pre-optical-redesign` before editing.
- Preserved every authoritative control id and all existing renderer, history, workspace, painting, and export bindings.
- Added the optical-black, blue-carbon, prism-mint, spectral-amber, and coral token system with Outfit and IBM Plex Mono roles.
- Added the inline spectral brand aperture, real geometry-name marquee, Lucide workspace icons, floating preview toolbar, and quieter contextual inspector.
- Bound the canvas aperture ring to live segment count and rotation through CSS custom properties without adding either value to application state.
- Added defensive GSAP/ScrollTrigger motion with reduced-motion protection; missing external motion resources do not block the editor.
- Confirmed Source → Symmetry → Paint workspace switching and ARIA selection on desktop and mobile.
- Changed Segments from 2 to 12 and confirmed the aperture interval updated from 180deg to 30deg.
- Checked the wide three-region layout and a 390 × 844 mobile layout with fixed six-item navigation, compact global actions, canvas-first reading order, and no horizontal overflow.
- Completed desktop and mobile browser passes with no console warnings or errors.

Files modified for the Optical Instrument redesign:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `DEVELOPMENT_MANUAL.md`

### Traced emblem integration QA completed — 2026-07-26

- Created `kaleidoscope-image-lab.html.backup-20260726-pre-svg-logo` before editing.
- Replaced the generated spectral brand aperture with the supplied traced SVG emblem as the primary logo.
- Embedded the SVG as a base64 data image because the sandboxed `srcdoc` CSP cannot reliably load a relative image asset.
- Confirmed the embedded SVG SHA-256 matches the supplied file exactly.
- Added an accessible `img` role and the name “Kaleidoscope Image Lab emblem.”
- Tuned only display size, background scale, brightness, contrast, and drop shadow; the SVG artwork itself is unchanged.
- Completed desktop and 390 × 844 mobile visual checks with no browser warnings or errors.

### Future pattern-systems roadmap planning — 2026-08-02

- Approved and added Milestones #20 through #25 to `MILESTONE_ROADMAP.md` as planned future work: Sacred Geometry Vector Mask Library, Mathematical Curve Library, Fractal Generator Pack, Number-Theory Pattern Pack, Tiling and Tessellation Pack, and Wave-Interference Pattern Pack.
- Recorded the approved dependency order: Milestones #20–#23 establish and validate the shared Pattern Pack framework; #24 may require stronger polygon, clipping, cell, and tiling infrastructure; #25 may require field sampling, displacement, and animation-performance infrastructure.
- Documented a shared Pattern Pack direction covering stable metadata, normalized/native browser rendering, deterministic generation, safe complexity limits, and compatibility with presets, Smart Randomizer, animation, painting, masks, overlays, export, and saved projects.
- Recorded Wolfram as an optional research, calculation, asset-generation, and validation aid only; it is not a required browser runtime or application service.
- Reconciled the overlapping `Recursive Kaleidoscope` backlog idea by referring it to Milestone #22 rather than retaining a competing description.
- Preserved all earlier milestone numbering and completion markers. Milestones #7 and #11–#19 remain defined in `Kaleidoscope_Image_Lab_Future_Milestones_v2.docx`; their pre-existing omission from the Markdown roadmap summary table was not expanded as part of this documentation-only update.
- No application code or pattern assets were changed, and none of Milestones #20–#25 were started.

### Temporary priority-based milestone renumbering — 2026-08-02 — superseded

- A documentation-only update temporarily substituted geometry-first execution positions for permanent milestone IDs. That numbering model was incorrect and was reversed in the correction entry below.
- The visual-first strategy itself remained approved, and no HTML, JavaScript, CSS, assets, application behavior, completed work, or pattern assets changed.

### Roadmap numbering correction — 2026-08-02

- Restored permanent original milestone IDs #1–#25 after the temporary priority-based renumbering. Milestone identity is now explicitly separate from recommended execution priority.
- Preserved completed work and its correct historical IDs: #1 Source Navigator is implemented; #8 Layered Symmetry Engine, #9 Symmetry Painting, and #10 Geometry Library are complete.
- Preserved the geometry-first execution strategy as a separate Priority field: visual finishing and transforms lead into Pattern Packs, exploration follows, and animation, live input, and project-platform work remain later.
- Identified Priority 1 — Milestone #11 Smart Randomizer as the next active implementation milestone at that documentation stage; it was completed later on 2026-08-02 in the record below.
- Confirmed existing DOCX prompt packs already use the permanent original IDs and require no correction. New prompts must use the permanent milestone ID, never the priority number.
- No completed milestone was removed or marked incomplete, and no application code, assets, runtime behavior, or pattern assets changed.

### Milestone #11 Smart Randomizer QA completed — 2026-08-02

- Created and checksum-verified `kaleidoscope-image-lab.html.backup-20260802-014914-milestone11-pre-smart-randomizer` before editing.
- Added Subtle, Balanced, and Wild strengths; Everything, Geometry Only, Transform Only, and Unlocked Values scopes; and six compact session-only locks in the existing Symmetry workspace.
- Added compact `settings.randomizer` metadata for Mirror, Triangle, Hexagon, Starburst, Spiral, Flower, Crystal, and Snowflake, with weighted geometry choices, preferred segment counts, safe zoom bands, density thresholds, center ranges, and shared correlation rules.
- Parsed every embedded script after implementation, confirmed 99 unique DOM ids with no duplicates, and verified all new selectors resolve.
- Ran Subtle, Balanced, and Wild 10 times each on an imported landscape PNG. All 30 states were unique within their strength run, remained in valid control ranges, retained the source filename, and produced no in-app errors. Subtle stayed near its starting geometry; Balanced and Wild exercised broader geometry/range sets, with Wild reaching the widest offsets and segment density.
- Exercised all eight registered geometries directly and through their Balanced geometry profiles. Profile samples produced bounded characteristic counts: Mirror 4, Triangle 9, Hexagon 6, Starburst 16, Spiral 10, Flower 5, Crystal 10, and Snowflake 12 in the recorded pass.
- Verified Geometry Only preserves all transform fields; Transform Only preserves geometry and segment count; Unlocked Values respects simultaneous Geometry and Rotation locks while changing other eligible values; Everything remains bounded.
- Verified Geometry, Segments, Rotation, Zoom, Source Position, and Center Position locks individually. Every protected field remained unchanged. Lock-only changes left Undo disabled in a fresh history session.
- Verified one Randomize action is one Undo step and one Redo step, with complete state restoration and the imported source filename preserved throughout.
- Applied all 24 presets, confirmed valid deterministic states and legacy Mirror geometry migration, then verified Mutate preserves geometry and Reset restores Mirror, two segments, zero rotation, 1× zoom, and Mirror Lab.
- Loaded generated landscape, portrait, and square PNGs plus a JPEG through the supported clipboard path; Balanced Randomize remained valid for every source.
- Enabled Painting Mode on an imported nonblank image, randomized, and undid the result. Painting Mode stayed active, the imported source identity remained intact, and the full rendering state was restored without an error.
- Invoked PNG export after Smart Randomizer operations; the button returned to idle with no export status error. Preview and export continue to share `drawActiveRenderer()` and the same state path.
- Checked 1200 × 800, 800 × 900, and 480 × 800 layouts. The settled 480px layout had matching 480px client/scroll widths, and the 480px Symmetry sidebar had matching 462px client/scroll widths with no horizontal overflow.
- Final browser-console review contained no warnings or errors.

Files modified for Milestone #11:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `DEVELOPMENT_MANUAL.md`
- `MILESTONE_ROADMAP.md`

### Milestone #18 — Procedural Preset Generator — implemented; runtime QA pending — 2026-08-17

- Added a compact Procedural Preset Generator inside the existing Symmetry → Presets and Exploration group. It provides eight style recipes: Balanced, Bold, Minimal, Geometric, Organic, Psychedelic, Dark, and Luminous; Gentle, Balanced, and Dramatic intensity; Generate Preset and Regenerate actions; and four compact include toggles for Pattern, Polar, Crystal, and Effects.
- Added bounded recipe-first generation for geometry, Mirror/Radial mode, segments, rotation, source transform, radial origin, reflection/crop, Pattern Pack family/pattern/use mode/appearance, Polar mode/settings, Crystal mode/settings, and a maximum three-entry Effect Stack with category-specific color palettes. Generated animation state is preserved and playback never starts automatically.
- Optional stages honor their include toggles: excluded Pattern, Polar, Crystal, and Effect Stack state is carried forward from the current composition. Generated output is applied through the existing `applyState()` path, clears the preset-library selection without overwriting saved presets, and reports the active generated stages with a reminder to use Library → Favorites for saving.
- Generator actions commit through `applyCreativeState()` as one complete rendering-history entry. Undo restores the prior composition and Redo restores the generated state; Smart Randomizer, Mutate, Favorites, Painting Mode, export, and the shared preview/export renderer remain separate and intact.
- Created and verified `kaleidoscope-image-lab.html.backup-20260817-225308-milestone18-pre-generator` before editing.
- Static QA passed: outer `srcdoc` extraction and HTML entity decoding, all 3 decoded inline application scripts with `node --check`, 230 unique DOM ids, and 33 generator Pattern Pack references matched to existing pack definitions. A temporary headless Chrome load also returned the authoritative outer document successfully.
- Interactive browser smoke QA remains pending. Python Playwright is not installed; the bundled Node Playwright browser launch aborted, and CDP connection to a manually launched Chrome was blocked by the environment. Physical desktop-browser validation is still required for generator interaction, visual coherence across all eight styles, one-step Undo/Redo, excluded-stage preservation, responsive layout, console review, and saving generated results through Favorites.

Files modified for Milestone #18:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `DEVELOPMENT_MANUAL.md`
- `MILESTONE_ROADMAP.md`

### Milestone #3 — Discovery Mode — complete — 2026-08-18

- Added a compact Discovery Mode at the top of the existing Library workspace. It offers Balanced, Minimal, Geometric, Organic, Psychedelic, Dark, Luminous, and Surprise Me flavors.
- Each batch is capped at four in-memory candidates. Candidates reuse the Milestone #18 Procedural Preset Generator recipes and intensity profiles, configure complete bounded rendering states, preserve the current animation settings without starting playback, and render 132 × 132 thumbnails through the existing `drawActiveRenderer()` path.
- Candidate generation does not call `applyState()`, change controls, or create history entries. Applying a candidate stops any active playback, applies the candidate through `applyState()`, clears the preset selection, and commits one ordinary rendering-history entry. Undo therefore returns to the pre-apply composition; Favorites remains the existing separate save action.
- Discovery batches are cleared when a new image or blank canvas is created so previews never remain attached to an earlier source. Regenerate Batch reuses the same bounded four-candidate flow.
- Created `kaleidoscope-image-lab.html.backup-20260817-120000-milestone3-pre-discovery` before editing. The backup is intentionally untracked.
- Static QA passed: outer `srcdoc` extraction and entity decoding, 8 embedded script tags with all 3 inline scripts passing `node --check`, 237 unique decoded DOM ids with no duplicates, and `git diff --check`.
- Per user report, desktop/browser QA was completed successfully on 2026-08-18. The previously deferred checks for flavor generation, four previews, Apply/Undo, Favorites handoff, responsive layout, and console behavior are closed.

Files modified for Milestone #3:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `MILESTONE_ROADMAP.md`

`DEVELOPMENT_MANUAL.md` was not changed because Discovery Mode reuses the existing procedural recipe, state/history, thumbnail, and renderer paths without adding a reusable architecture layer.

### Milestone #19 — Workspace / Project System Expansion — complete — 2026-08-18

- Added a compact Project section to the existing Library workspace with New Project, Open Project, and Save Project actions. No project browser, Recent Projects, autosave, cloud sync, or dirty-flag system was added.
- Project files are readable JSON with `{ "format": "kaleidoscope-image-lab-project", "version": 1 }`, a bounded source snapshot, and the complete serializable composition state from `getState()`. The state includes source transforms, geometry/symmetry, Pattern Pack, Polar, Crystal, Effect Stack, animation, and Organic Motion settings.
- Undo/Redo stacks, renderer caches, generated vector geometry, transient playback/recording values, preview-only values, and Discovery candidate batches are excluded.
- Imported images reuse the existing bounded Favorites source snapshot path and preserve alpha through the canvas data URL. Painted sources reuse the existing bounded painted-canvas snapshot path, restoring the flattened painted source without external file references.
- Save downloads `kaleidoscope-project.kilab.json` without changing the composition or history. Open validates format/version/state/source, ignores unknown fields, restores the source before applying state, resets history to one post-load entry, and restores animation settings without autoplay or recording. Invalid files leave the current composition unchanged and report a clear error.
- New Project reuses the existing blank-canvas and Reset/default paths, clears the prior source without reloading the page, and uses a compact inline discard confirmation when a source is active. Favorites, Procedural Presets, and Discovery remain separate systems.
- Created `kaleidoscope-image-lab.html.backup-20260818-010000-milestone19-pre-project-system` before editing.
- Static QA passed: outer `srcdoc` extraction and entity decoding, all 3 decoded inline application scripts with `node --check`, 246 unique decoded DOM ids with no duplicates, and `git diff --check`.
- Per user report, desktop/browser QA passed for native image/project pickers, downloaded JSON inspection, imported and painted source restoration, combined Pattern + Polar + Crystal + Effects restoration, history behavior, no-autoplay behavior, responsive Library layout, Favorites/Procedural Presets/Discovery smoke checks, and browser-console review.

Files modified for Milestone #19:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `MILESTONE_ROADMAP.md`
- `DEVELOPMENT_MANUAL.md`

Backup created for Milestone #11:

- `kaleidoscope-image-lab.html.backup-20260802-014914-milestone11-pre-smart-randomizer`

### Milestone #12 Effect Stack QA completed — 2026-08-02

- Created `kaleidoscope-image-lab.html.backup-20260802-230634-milestone12-pre-effect-stack` before editing.
- Replaced the fixed Effects controls with a compact ordered stack supporting Add Effect, enable/disable, expand/collapse, Move Up, Move Down, Duplicate, Remove, and Reset Stack.
- Added stable instance ids, normalized serializable settings, effect definitions, legacy flat-effect migration, neutral empty-stack defaults, preset fallback, and stack-aware `getState()`/`applyState()` history.
- Confirmed adding Brightness and Gradient Map, reordering, duplication, disabling, removing, editing a gradient color, Reset Stack, full Reset, Undo, and Redo through the browser UI. Undo/Redo restored stack order, entries, enabled states, and settings.
- Confirmed preview and export route through the same ordered `PostProcessingPipeline` path; PNG and JPEG export actions returned to idle without an in-app error.
- Reloaded the application, added a stack entry, and verified the Effects workspace remained usable after initialization.
- Checked the browser console after the stack and export interactions: no warnings or errors.

Files modified for Milestone #12:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `DEVELOPMENT_MANUAL.md`
- `MILESTONE_ROADMAP.md`

Known QA follow-ups for Milestone #12:

- The in-app browser did not expose the native image picker or saved download artifact, so independent preview-vs-file pixel comparison remains a physical desktop-browser follow-up.
- Order-sensitive visual comparison with a nonblank imported source and physical drag/wheel regression remain recommended desktop-browser checks.

### Milestone #14 Polar Engine QA completed — 2026-08-02

- Created `kaleidoscope-image-lab.html.backup-20260802-235900-milestone14-pre-polar-engine` before editing.
- Added the compact Polar Engine disclosure to the existing Symmetry workspace with Off, Polar Wrap, Radial Tunnel, Ring Repeat, Vortex, center, rotation, strength, scale, repeat-count, direction, and Reset Polar controls. Mode-specific controls remain hidden until relevant.
- Added `PolarEngine` as a reusable post-geometry, pre-Effect Stack stage. It uses a reusable pixel sample buffer, bounded settings, no per-pixel object allocation, no temporary canvases, and a fast Off path.
- Added nested Polar state to reversible `getState()`/`applyState()` and `statesMatch()`. Old states without Polar data fall back to Off; full Reset returns Off and default settings; mode changes, completed slider edits, Reset Polar, Undo, and Redo use the existing history path.
- Confirmed distinct visual output for all five modes on an imported PNG: Off, Polar Wrap, Radial Tunnel, Ring Repeat, and Vortex. Confirmed Vortex direction visibility and mode-specific Ring Repeat/Direction controls.
- Switched all eight registered geometries with Vortex active without rendering errors. Confirmed presets, Smart Randomizer, Mutate, full Reset, and Effect Stack remain usable while Polar is active.
- Invoked PNG and JPEG export at 1080 × 1080; both returned to idle with no in-app error and used the shared preview/export renderer path. The in-app browser did not expose a saved download artifact for independent file inspection.
- Checked desktop (1280 × 800), tablet (800 × 900), and narrow (640 × 900) layouts; all measured client and scroll widths matched with no horizontal overflow.
- Browser console review after the corrected pass contained no new warnings or errors. A pre-existing Effect Stack range-handler error found during QA was fixed with the smallest local variable-scope correction so effect tuning remains clean.

Files modified for Milestone #14:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `DEVELOPMENT_MANUAL.md`
- `MILESTONE_ROADMAP.md`

Backup created for Milestone #14:

- `kaleidoscope-image-lab.html.backup-20260802-235900-milestone14-pre-polar-engine`

### Milestone #14 known limitations

- Polar uses bounded nearest-neighbor pixel sampling. It is intentionally a reliable foundation rather than a high-quality resampling or iterative displacement system.
- The browser harness did not expose the saved PNG/JPEG artifacts or reliably support native file-picker selection, so independent file inspection, file-picker/drop, and pixel-by-pixel preview/export comparison remain physical desktop-browser follow-ups.
- Polar adds one reusable full-frame sample buffer when active and can make 1200 × 1200 or large export renders CPU-intensive; Off remains allocation-free and skips the stage.

### Milestone #13 Crystal Mode QA completed — 2026-08-02

- Created `kaleidoscope-image-lab.html.backup-20260803-000000-milestone13-pre-crystal-mode` before editing.
- Added the compact Crystal Mode disclosure to the existing Symmetry workspace with Off, Facet, Prism, Shard, Crystal Bloom, facet size/density, rotation, strength, spread/displacement, relevant direction/reflection controls, and Reset Crystal.
- Added `CrystalModeStage` after Polar Engine and before the Effect Stack. Preview, disabled-layer fallback, and original-resolution export share the same stage path; Off returns immediately.
- Confirmed nested Crystal state, legacy fallback to Off, full Reset defaults, one-step mode/control history behavior, Reset Crystal, Undo/Redo, and safe preservation through presets, Smart Randomizer, and Mutate.
- Exercised every Crystal mode and control at safe extremes, all eight geometries with Crystal active, all five Polar modes with Crystal active, and a post-crystal Effect Stack entry. No blank-output errors or console warnings/errors appeared.
- Confirmed transparent blank Painting Mode remains transparent through the Crystal path and that the Crystal controls remain reachable at a narrow viewport. PNG and JPEG export actions returned without an in-app error through the shared route.

Files modified for Milestone #13:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `DEVELOPMENT_MANUAL.md`
- `MILESTONE_ROADMAP.md`

Known QA follow-ups for Milestone #13:

- The in-app browser blocked direct `file://` navigation, did not expose the native picker or saved download artifacts, and did not provide reliable nonblank imported-source file comparison. Physical desktop checks for PNG/JPEG artifact pixels, imported PNG/JPEG, drag/drop, and preview/export pixel parity remain recommended.
- Physical desktop visual comparison across portrait, landscape, and square photographic sources remains recommended; the browser pass covered the registered geometry and stage paths with a transparent painted-source session.

### Milestone #20 Sacred Geometry Vector Mask Library — complete — 2026-08-03

- Created `Pattern Pack` foundation metadata for stable pack/pattern ids, names, descriptions, categories, versions, vector rendering type, supported use modes, defaults, safe ranges, and supported parameter keys. The contract is intentionally local to the standalone application so future Pattern Packs can reuse it without an external loader.
- Added the Sacred Geometry pack with eight procedural vector patterns: Seed of Life, Flower of Life, Vesica Piscis, Metatron-style Lattice, Nested Circle Mandala, Hexagonal Circle Grid, Radial Polygon Grid, and Star Polygon Mandala.
- Added compact controls in the existing Symmetry workspace for pack, pattern, Overlay/Mask/Painting Guide mode, enabled state, scale, rotation, center X/Y, line width, line color, opacity, and bounded pattern-specific density, overlap, ring count, polygon sides, point count, and inner ratio settings. Unsupported pattern-specific controls stay hidden.
- Added `SacredGeometryStage` as one shared vector rendering path after Geometry/Layer Manager, Polar Engine, and Crystal Mode, and before the ordered Effect Stack. The same path serves preview, fallback, and original-resolution export.
- Overlay blends vector color over existing pixels; Mask limits artwork alpha to the selected vector coverage while clearing fully transparent RGB channels; Painting Guide draws only on the preview after rendering and remains source-pixel and export independent.
- Added nested `pattern` state with safe fallback to disabled defaults for older states and presets. Pattern enablement, mode changes, completed control edits, Reset Pattern, full Reset, Undo, and Redo use the existing reversible history path. Smart Randomizer, presets, and Mutate preserve the current pattern state without generating new Sacred Geometry values.
- No Wolfram calls were used. The constructions use compact normalized procedural rules and bounded native Canvas vector commands; Wolfram remains optional and is not a runtime dependency.
- Browser QA covered all eight patterns, all three use modes, shared and pattern-specific controls, guide visibility, Reset Pattern, full Reset, controlled Undo/Redo, all eight registered geometries, all Polar modes, all Crystal modes, Effect Stack compatibility, Randomizer, Mutate, PNG/JPEG export invocation, a painted source, desktop/tablet/narrow layouts, and console review. No new warnings or errors appeared.
- The in-app browser did not expose a saved download artifact for independent pixel comparison, and direct native file-picker coverage remained unavailable. Physical desktop checks for imported PNG/JPEG, saved PNG/JPEG transparency, and exact preview/export pixel parity remain recommended follow-ups.

Files modified for Milestone #20:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `DEVELOPMENT_MANUAL.md`
- `MILESTONE_ROADMAP.md`

### Milestone #21 Mathematical Curve Library — complete with post-completion `srcdoc` escaping hotfix — 2026-08-03

- Extended the existing Pattern Pack metadata and shared `SacredGeometryStage` path with the Mathematical Curves pack; no second state system or curve-only renderer was introduced.
- Added eight normalized, deterministic, bounded sampled paths: Rose Curve, Lissajous Figure, Hypotrochoid, Epitrochoid, Archimedean Spiral, Logarithmic Spiral, Lemniscate, and Superformula Shape. Curve controls are generated from the active curve's parameter metadata and use creative labels with safe ranges.
- Overlay, Mask, and Painting Guide reuse the existing preview/fallback/export route. Guide mode is non-destructive and does not affect exports; all export-relevant curve settings participate in `pattern` state, Reset Pattern, full Reset, and Undo/Redo. Older states and presets fall back to the existing disabled Sacred Geometry default.
- No Wolfram calls were used: native JavaScript equations and bounded Canvas path sampling were sufficient, and no runtime dependency was added.
- Static QA parsed all embedded scripts successfully and confirmed the new pack definitions, bounded samplers, cached parameter-aware vector commands, and shared path drawing route. Browser automation could not be initialized in this environment, so physical desktop visual and saved-file comparisons remain follow-ups.
- Post-completion hotfix: an unescaped double quote in a new selector inside the outer iframe `srcdoc` attribute prematurely terminated the embedded application document. The embedded app therefore failed to initialize: loading a new image did nothing, and left-side workspace navigation did not respond.
- The corrective change escaped that selector quote for the outer document. The repaired wrapper has one intact `srcdoc` payload with no unescaped inner double quotes; after decoding, all three embedded scripts parse successfully. Future changes inside `srcdoc` must preserve the outer-document escaping as well as inner-script syntax.

Backup created before editing:

- `kaleidoscope-image-lab.html.backup-20260803-090000-milestone21-pre-mathematical-curves` (SHA-256 matched the pre-edit application)

Files modified for Milestone #21:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `DEVELOPMENT_MANUAL.md`
- `MILESTONE_ROADMAP.md`

Backup created before editing:

- `kaleidoscope-image-lab.html.backup-20260803-003405-milestone20-pre-sacred-geometry`
- `PROJECT_NOTES.md.backup-20260803-003405-milestone20-pre-sacred-geometry`
- `DEVELOPMENT_MANUAL.md.backup-20260803-003405-milestone20-pre-sacred-geometry`
- `MILESTONE_ROADMAP.md.backup-20260803-003405-milestone20-pre-sacred-geometry`

### Milestone #22 Fractal Generator Pack — complete — 2026-08-03

- Added the Fractal Generators pack to the existing Pattern Pack selector with eight deterministic, normalized generators: Koch Snowflake, Sierpinski Triangle, Dragon Curve, Hilbert Curve, Recursive Tree, Barnsley-style Fern, Apollonian Circle Pack, and Levy C Curve.
- The metadata-first Pattern Pack contract is reused. Every fractal provides a stable id, name, description, category, version, rendering type, defaults, safe ranges, parameter definitions, reusable `build(settings)` geometry, and compact complexity guidance. No alternate renderer or runtime dependency was added.
- Shared controls remain scale, rotation, center, line width/color, opacity, enablement, and use mode. The generated controls show only active fractal parameters: depth/order, Koch orientation, tree branch angle/scale and seed, fern point count and seed, or packing depth as applicable.
- Conservative caps prevent runaway work: Koch 3,072 segments, Sierpinski 729 triangles, Dragon and Levy 8,192 segments, Hilbert 4,095 segments, Recursive Tree 2,047 branches, Barnsley-style Fern 9,000 points, and Apollonian Circle Pack 364 circles. Generated geometry is cached by pack, pattern, and relevant parameters; only parameters, never point arrays, enter reversible state.
- Overlay, Mask, and Painting Guide remain on the shared Pattern Pack stage after Crystal Mode and before Effect Stack. Overlay and Mask therefore use the same preview/fallback/export rendering path; Mask multiplies alpha and clears fully transparent RGB; Painting Guide renders only after preview drawing and does not affect source pixels or exports.
- Pattern changes, completed control edits, Reset Pattern, full Reset, Undo, and Redo reuse the existing nested `pattern` state path. Older states and presets fall back safely to the disabled Sacred Geometry default. Smart Randomizer, presets, and Mutate preserve Pattern Pack state and do not generate fractal settings.
- No Wolfram use was needed. The formulas and iteration rules are compact native JavaScript and Wolfram is not a runtime dependency.
- QA performed: outer `srcdoc` decoding, decoded HTML presence check, all 8 embedded JavaScript blocks parsed with `new Function`, Fractal Pack registration check, and duplicate DOM-ID check (195 unique ids). The in-app browser blocks `file://` navigation, so physical desktop visual comparisons, imported PNG/JPEG, gesture behavior, and saved PNG/JPEG inspection remain follow-ups.

Backup created before editing:

- `kaleidoscope-image-lab.html.backup-20260803-milestone22-pre-fractals`

Files modified for Milestone #22:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `DEVELOPMENT_MANUAL.md`
- `MILESTONE_ROADMAP.md`

### Post-completion Pattern Pack Expansion — Sacred Geometry + Fractals — 2026-08-03

- This focused non-milestone pass extends completed Milestones #20 and #22 without starting Milestone #23. It preserves the existing Pattern Pack metadata, cache, renderer, `pattern` state, Undo/Redo, Reset, preview/export route, and Overlay/Mask/Painting Guide behavior.
- Sacred Geometry additions: Fruit of Life, Merkaba / Star Tetrahedron Projection, Sri Yantra–Inspired Grid, Platonic Solid Projection Set, Concentric Hexagram Lattice, and Vesica Chain Mandala. The Sri Yantra construction is explicitly an inspired, bounded vector grid rather than a claim of canonical ritual precision.
- Fractal additions: Pythagoras Tree, Minkowski Sausage, Gosper Curve, Vicsek Fractal, Hexaflake, and Cantor Dust. All generators produce compact normalized circles, polygons, line sets, or paths and remain separate from presentation style.
- Added one small backward-compatible Pattern Pack metadata extension: a parameter definition may specify a bounded `options` list. The existing dynamic control UI presents it as a native select and includes its compact numeric value in the existing reversible `pattern` state. Platonic Solid Projection uses this for Tetrahedron, Cube, Octahedron, Dodecahedron, and Icosahedron wireframe-style choices.
- New caps: Fruit of Life 19 circles; Sri Yantra–Inspired Grid 10 triangles; Platonic projection 36 lines; Hexagram Lattice 12 triangles; Vesica Chain 24 circles; Pythagoras Tree 511 squares; Minkowski Sausage 4,096 segments; Gosper Curve 2,401 segments; Vicsek 625 cells; Hexaflake 343 hexagons; Cantor Dust 1,024 cells. All values are clamped in the generator rules.
- No Wolfram use was needed and no Wolfram runtime dependency was added.
- QA performed: every decoded embedded script parses, all 12 new builder functions execute at maximum settings within their documented bounds, pack entries carry the requested metadata, outer `srcdoc` decodes successfully, and duplicate DOM-ID validation remains clean. Physical visual, imported PNG/JPEG, saved export, pointer interaction, responsive-layout, and browser-console checks remain desktop-browser follow-ups because local app navigation is blocked in this environment.

Backup created before editing:

- `kaleidoscope-image-lab.html.backup-20260803-pattern-pack-expansion-pre`

Files modified for this expansion:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `DEVELOPMENT_MANUAL.md`
- `MILESTONE_ROADMAP.md`

### Non-milestone UI/UX refinement — 2026-08-13

- Preserved the Optical Instrument palette, traced emblem, workspace rail, global toolbar, canvas priority, live aperture ring, authoritative control ids, rendering state, and renderer/export paths.
- Reordered Symmetry around the core workflow: presets and exploration → geometry → symmetry transform → mode-specific radial origin → reflection/crop, followed by clearly labeled optional Pattern Pack, Polar, Crystal, and Advanced Layers sections.
- Reduced default control density by collapsing import guidance, Pattern Pack, Polar, Crystal, Advanced Layers, and Saved Work. Closing any section remains UI-only and does not disable or clear active settings.
- Reshaped Pattern Pack into a clear Family → Pattern → Variant when relevant → Use As flow with nested Placement and Appearance & Parameters disclosures. Pattern selection and settings continue to use the existing controls and state/history path.
- Placed Center Source, Center Radial Origin, Reset Pattern, Reset Polar, Reset Crystal, and Reset Stack with the controls they affect; shortened guidance and removed future-implementation language from the live Geometry summary.
- Prioritized Export Settings above the unavailable Saved Work placeholder, contained the Animation placeholder, and made randomizer Strength/Scope compact at wider panel widths without changing functionality.
- Created and checksum-verified `kaleidoscope-image-lab.html.backup-20260813-223740-pre-ui-refinement` before editing.
- Static QA confirmed one intact decoded `srcdoc`, valid embedded scripts, unique DOM ids, and preserved selector ids. Localhost Chrome QA covered all six workspaces and ARIA selection, source-value persistence across workspaces, all six Pattern Pack families, categorical variant visibility, Pattern disclosure persistence, Polar/Crystal resets, radial-origin visibility/reset, geometry Undo/Redo, Paint state across workspace switches, Effect Stack add/reset, Focus Canvas restore, PNG download generation, wide and 480 px no-overflow layouts, and a clean console.
- Physical native picker/drop, painted strokes, canvas pan/wheel gestures, platform Save As, and independent desktop visual review remain follow-ups.

Files changed for this non-milestone pass:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `DEVELOPMENT_MANUAL.md`

`MILESTONE_ROADMAP.md` was reviewed and not changed because no milestone was started or reprioritized.

### Milestone #2 Live Animation — complete — 2026-08-14

- Replaced the Animation placeholder with compact Playback, Motion Channels, and Recording groups in the existing Optical Instrument workspace.
- Added a bounded `requestAnimationFrame` playback engine with a practical 30 fps render ceiling, global speed multiplier, resumable Pause, and Stop behavior that resets elapsed phase and restores the exact base composition.
- Added ten opt-in sine-wave channels: Global Rotation, Zoom/Scale, Source X, Source Y, Center X, Center Y, Pattern Rotation, Pattern Scale, Polar Rotation, and Crystal Strength. Each channel provides Enable, Amount, and Speed controls. Source X/Y modulate the existing source-position sampling controls.
- Animation settings are nested reversible state and use the existing `getState()` / `applyState()` / Undo / Redo path. The live modulation snapshot is composed only for rendering; base controls, loaded source pixels, painted pixels, and history remain unchanged during playback.
- Added preview-canvas recording at 30 fps using `canvas.captureStream()` and `MediaRecorder`, automatic WebM/MP4 MIME detection, Standard 6 Mbps and High 12 Mbps quality choices, Record/Stop controls, clear status feedback, object-URL download, unsupported-browser messaging, and origin-clean failure handling. No server, codec library, or external dependency was added.
- Created and checksum-verified `kaleidoscope-image-lab.html.backup-20260814-220231-milestone2-pre-live-animation` before editing.
- Static QA confirmed an intact outer `srcdoc` boundary, successful parsing of all 3 decoded inline scripts, 206 unique static ids, and complete selector wiring. Localhost headless Chrome QA loaded the supplied PNG, confirmed 10 animation channels including independent Source X/Y movement, exact baseline restoration after Stop, a frozen Pause frame, unchanged base controls/source pixels during playback, a base animation-setting edit that Undo restored in one step, and no playback history spam.
- Final recording QA produced a 522,878-byte WebM from the animated preview. Unsupported `captureStream()` was simulated and produced the visible message “Canvas recording is not supported by this browser.” A PNG still export produced a 3,118,686-byte file.
- Regression QA switched all 6 Pattern Pack families, all 8 geometries, Mirror/Radial modes, a preset, Randomize, Mutate, Painting Mode, Effect Stack add/reset, and all 6 workspaces. The 480 px layout had no inner or outer horizontal overflow, runtime ids remained unique, and browser console/page-error logs were clean.
- Physical native picker/drop, physical canvas and painting gestures, platform Save As behavior, independent media playback in other browsers, MP4 recording, recording alpha, and long-duration performance remain desktop-browser follow-ups.

Files changed for Milestone #2:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `DEVELOPMENT_MANUAL.md`
- `MILESTONE_ROADMAP.md`

### Live Animation Source Motion — non-milestone enhancement — 2026-08-14

- Added Source X and Source Y opt-in sine-wave channels to the existing Motion Channels area. Each uses the same Enable, Amount, and Speed controls as the other channels; no phase-offset control was added because the current animation model has no per-channel phase state.
- Source motion is applied only to the temporary render-state snapshot as bounded `offsetX`/`offsetY` modulation. Imported pixels and painted source pixels remain intact, base source-position controls remain unchanged, and Stop restores the exact base source position by returning to zero elapsed phase and the ordinary base state.
- Recording captures the animated preview canvas through the existing `captureStream(30)` path, so enabled Source X/Y motion is included automatically without a recording-specific renderer or history entries.
- QA performed: outer HTML and decoded `srcdoc` validation, all 8 embedded script parses, 208 unique static ids, duplicate-id check, static Source X/Y wiring, localhost Chrome playback with independent channels, Amount/Speed changes, Stop restoration, Pause behavior, existing-channel smoke check, animation-setting Undo/Redo, and recording-path smoke check. Imported-image runtime QA passed; painted-source and physical file-picker/gesture checks remain desktop-browser follow-ups.
- Known limitations: source motion remains one sine waveform per channel with one global speed multiplier; per-channel phase, timelines, noise, orbit systems, and long-duration performance work remain outside this focused enhancement. In the available headless Chrome run, recording started but finalized without video data, so a nonempty recording artifact still requires physical desktop-browser confirmation.

Files changed for this non-milestone enhancement:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `DEVELOPMENT_MANUAL.md`
- `kaleidoscope-image-lab.html.backup-20260814-source-motion-pre-edit`

### Milestone #7 Organic Motion Engine — implemented; browser QA pending — 2026-08-14

- Extended the existing Live Animation system with a compact Organic Motion disclosure rather than a competing animation engine. Supported styles are Drift, Orbit, Breathe / Pulse, Wobble, and bounded deterministic Wander.
- Supported Organic Motion targets are Source Position (paired Source X/Y), Center Position (paired Center X/Y), Global Rotation, Zoom / Scale, Pattern Rotation, Polar Rotation, and Crystal Strength. Orbit exposes an additional bounded radius control and applies circular or elliptical paired movement to Source/Center Position.
- Organic settings are nested in reversible `animation` state and use the existing `getState()` / `applyState()` / Undo / Redo path. Playback, Stop, Reset Organic Motion, base-state restoration, source immutability, and recording continue through the existing session-only render modulation and preview-canvas paths.
- Created and checksum-verified `kaleidoscope-image-lab.html.backup-20260814-225926-milestone7-pre-organic-motion` before editing (SHA-256 `2a289fc0c890d8b839daf6e595c71c2846b7b68bba3482783a78d29edb8a07d7`).
- Static QA passed: one intact outer `srcdoc`, three non-empty decoded inline scripts parsed successfully, no duplicate static IDs, and the source diff is limited to Organic Motion UI/state/render integration plus documentation.
- Runtime browser QA was attempted through the project Playwright workflow. Python Playwright is unavailable, the bundled Chromium executable is absent, the restricted environment blocks localhost binding, and Playwright-launched installed Chrome aborts before navigation. Console cleanliness, imported-image playback, painted-source behavior, recording artifact capture, and physical responsive/gesture behavior therefore remain unverified here.

Known limitations for this milestone:

- Organic Motion uses lightweight bounded trigonometric signals rather than physically simulated noise, audio reaction, timelines, keyframes, or per-channel phase controls.
- Source and Center Position are grouped as paired targets to keep Orbit genuinely circular/elliptical and the Animation workspace compact.
- Rendering remains synchronous and uses the existing 30 fps request ceiling; large Pattern Pack, Polar, Crystal, or Effect combinations may reduce the effective rate.

Files changed for Milestone #7:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `DEVELOPMENT_MANUAL.md`
- `MILESTONE_ROADMAP.md`
- `kaleidoscope-image-lab.html.backup-20260814-225926-milestone7-pre-organic-motion`

### Animation Playback Sticky Controls — UI usability refinement — 2026-08-14

- The existing Animation Playback section now uses `position: sticky` within the right-side settings scroll container, keeping Play, Pause, Stop, speed, and status accessible while Motion Channels and Organic Motion settings scroll underneath. The change reuses the existing controls and leaves the canvas, other workspaces, and global header unchanged.
- The sticky surface uses the existing panel background with a subtle separator shadow and remains active in the narrow stacked layout.

### Milestone #5 Favorites Gallery — implemented; runtime follow-ups pending — 2026-08-16

- Added the Favorites Gallery to the existing Library workspace with Add to Favorites, compact thumbnail cards, Restore, inline Rename, Delete, and inline-confirmed Clear All controls. The gallery uses one authoritative action for adding the current composition and keeps project save/load in the existing future-work disclosure.
- Favorites capture the complete current `getState()` payload: source transform, symmetry/geometry, Pattern Pack, Polar, Crystal, Effect Stack, animation configuration, and Organic Motion settings. They do not invent a second rendering-state model or store geometry caches. The current active source is also stored as a bounded raw or painted-source snapshot so a favorite remains restorable after a reload when browser storage is available.
- Thumbnail generation renders a lightweight 180 × 180 image through `drawActiveRenderer()` and the existing Pattern Guide path. Source snapshots are capped at 1200px on the longest side. WebP is preferred with PNG fallback; the 24-favorite limit prevents unbounded gallery growth.
- Browser-local persistence is attempted through `localStorage` with corrupted-record filtering and safe session-only fallback. The sandboxed iframe reported storage unavailable during QA, so persistence across reload remains environment-dependent rather than claimed as universal.
- Restoring stops playback, decodes the bounded source snapshot, applies the saved state through `applyState()`, refreshes the preview, and commits at most one normal rendering-state history entry. Paint stroke replay, original full-resolution source recovery, and playback status remain outside the current serializer and are documented limitations.
- QA performed: backup checksum verification; outer `srcdoc` extraction; all eight decoded scripts parsed with Node; no duplicate DOM IDs; localhost browser interaction with blank source creation, Add to Favorites, thumbnail presence, default naming, inline rename, restore after Randomize, Undo availability after restore, Clear All confirmation without deleting the test card, no browser console errors, and 480px responsive Library reachability with no horizontal overflow. The browser file chooser did not expose a usable chooser event, so native imported-file intake remains a desktop follow-up.

Files changed for Milestone #5:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `DEVELOPMENT_MANUAL.md`
- `MILESTONE_ROADMAP.md`
- `kaleidoscope-image-lab.html.backup-20260816-231729-milestone5-pre-favorites`

## 10. Known limitations

- There is no automated test suite, linter, package manifest, or build command.
- Geometry formulas in this milestone prioritize distinct, stable output and extensibility over mathematically exact tessellation; future geometry-specific controls are intentionally placeholders.
- All geometry rendering remains CPU-based on the main thread. Cached plans and allocation reuse reduce overhead, but repeated 1200 × 1200 renders and large exports can still be computationally expensive.
- The in-app browser harness does not reliably deliver physical pointer-drag or wheel gestures into the sandboxed preview iframe. Desktop-browser confirmation is still needed for radial source panning, center-marker dragging, Option/Alt-drag, wheel feel, cancellation, and gesture history grouping.
- The in-app browser harness did not expose its native file chooser or download artifact during Milestone #10 QA. Generated aspect-ratio images were loaded through the supported clipboard path, and export fidelity was checked at the shared renderer output; real-browser file-picker/drop and saved-file inspection remain recommended.
- The browser harness does not expose the native Save As/download result for independent file inspection. The export path has been invoked without console errors, but saved-file dimensions and transparency still need real-browser inspection.
- Full file-picker, operating-system drag-and-drop, and image-clipboard coverage across PNG, JPEG, WebP, GIF, BMP, and AVIF remains to be run with real sample files.
- The forced -90-degree import rotation may not be correct for every image source.
- Full-resolution export is synchronous on the main thread and may briefly freeze the UI or use substantial memory for large images.
- A 4096 × 4096 export requires large temporary pixel buffers; JPEG also needs a same-size compositing canvas.
- The 100dvh desktop shell and new 900px three-region breakpoint should be checked at target-browser zoom levels and viewport sizes.
- Preset descriptions are stored for future use but are not yet shown in a dedicated browser panel.
- The Layer Manager currently treats Mirror and Radial as alternative active symmetry layers selected by Mode; true multi-layer visual composition is intentionally deferred.
- Layer settings are placeholders; existing Mirror Settings and Radial Settings remain the source of truth for current controls.
- Manual QA used a generated landscape PNG. Additional portrait, square, and large-source testing remains appropriate for future renderer-heavy milestones.
- The in-app browser harness did not reliably deliver a physical drag into the sandboxed preview iframe during this milestone, so real desktop-browser confirmation is still needed for visible stroke smoothness, per-drag history, eraser behavior, and all brush types.
- Painting currently supports one internal source canvas without layers, pressure sensitivity, custom/texture brushes, tablet input, or symmetry masks; those are intentionally future architecture targets.
- Sacred Geometry patterns are procedurally vector-clean but their Mask mode uses simple filled/stroked coverage rather than a multi-layer mask editor. Pattern rendering is CPU-based and intentionally bounded; large exports can still be compute-intensive.
- Painting from an imported image uses the capped interactive preview as its editable source, so very large painted exports are limited to that painting-canvas resolution rather than reconstructing the original full-resolution pixels.
- Live Animation remains intentionally limited to bounded sine and Organic Motion signals, the supported target subset, and a single global speed multiplier. It has no keyframes, timeline, per-channel phase control, easing editor, audio reaction, or commit-current-frame action.
- Animation rendering remains synchronous on the main thread and is capped at 30 requested preview renders per second. Dense Pattern Packs and active Polar, Crystal, and Effect stages may run below that rate on slower hardware.
- Preview recording depends on `canvas.captureStream()` and `MediaRecorder`. Container, codec playback, effective bitrate, and alpha support vary by browser; Safari/Chromium/Firefox interoperability and long-duration memory use remain unverified.
- Favorites are implemented as bounded visual snapshots, but browser-local persistence depends on storage being available to the sandboxed iframe. The QA browser reported session-only storage, so reload survival still needs confirmation in a browser configuration that permits local storage. Favorites preserve a bounded source snapshot rather than the original full-resolution source; Painting Mode preserves the current painted pixels but not replayable stroke history or brush settings. Project Save/Open/New is complete under Milestone #19.
- Glow is intentionally a lightweight highlight lift rather than a spatial blur/bloom effect; a fuller bloom pass remains outside Milestone #12.
- During workspace UI QA, the in-app browser again did not deliver physical canvas drags/wheel events and did not expose native picker or saved-download artifacts. Brush selection/source creation, numeric source controls, export completion, shared state, and UI behavior were verified; physical brush strokes, file-picker/drop, wheel/pan feel, and independent saved-file comparison still require a desktop-browser pass.
- Smart Randomizer output is intentionally nondeterministic and has no seed, scoring, image analysis, variation browser, or favorites integration. Those systems were explicitly outside Milestone #11.
- The Milestone #11 browser harness again did not expose the native file chooser or downloaded PNG artifact, and its attempted physical painting drag did not commit a stroke. Clipboard image intake, imported-image Painting Mode source safety, export completion, shared preview/export routing, and state/history behavior passed; independent saved-file comparison and a real painted-stroke source remain recommended desktop-browser checks.
- Crystal Mode is intentionally a bounded nearest-neighbor treatment rather than a physically simulated refraction, blur, lighting, or 3D system. Larger exports can still be CPU-intensive because the shared pixel stages run synchronously on the main thread.

## 11. Recommended next development step

The recommended next development area is **Milestone #4 Compare & Evolution Mode**. Before or alongside that work, a short physical desktop-browser QA pass should confirm Favorites with imported PNG/JPEG sources, representative Pattern Pack/Polar/Crystal/Effect/animation restoration, Delete/Clear All completion, painting-source behavior, reload persistence where storage is available, and native file-picker/drop behavior.

## 11a. Milestone #23 — Number-Theory Pattern Pack — complete — 2026-08-03

- Added the Number Theory pack to the existing Pattern Pack selector with eight deterministic normalized vector generators: Ulam Spiral, Prime Spiral, Modular Multiplication Circle, Fibonacci Bloom, Golden-Angle Phyllotaxis, Farey Sunburst, Recamán Web, and Pascal Triangle Modulo Pattern.
- Each definition reuses the established metadata contract: stable id, name, description, `number-theory` category, version, vector rendering type, defaults, safe ranges, active-only parameter definitions, complexity guidance, and a compact `build(settings)` rule. No new renderer, workspace, network call, raster asset, or runtime dependency was introduced.
- Generator caps are enforced inside the builders: Ulam and Golden-Angle patterns use at most 1,600 positions; Prime Spiral tests at most 1,600 integers; Modular Multiplication Circle uses at most 240 chords; Fibonacci Bloom has at most 1,200 plotted points plus 986 Fibonacci petals; Farey Sunburst uses at most 325 reduced fractions plus six rings; Recamán Web uses 360 terms and 5,744 sampled arc segments; Pascal Triangle uses at most 48 rows and 1,176 triangular cells.
- The shared cached Pattern Pack stage remains responsible for presentation and routing. Overlay and alpha-preserving Mask run after Crystal Mode and before the Effect Stack for preview, fallback, and export; Painting Guide draws after preview rendering, leaves source pixels unchanged, and remains absent from exports. Pattern parameters—not generated geometry—remain the reversible state payload, so active pack/pattern changes, completed control changes, Reset Pattern, full Reset, Undo, and Redo continue through the existing `pattern` state path. Older states and presets still fall back safely, while Smart Randomizer, presets, and Mutate retain current Pattern Pack state without generating Number Theory values.
- No Wolfram use was needed: the modular, spiral, Farey, Recamán, and Pascal rules are compact native JavaScript and Wolfram is not a runtime dependency.
- QA performed: decoded outer `srcdoc` validation; all 8 embedded JavaScript blocks parsed successfully; 195 DOM ids remained unique; all eight pack entries were present; each builder was run twice at deliberately out-of-range settings to confirm deterministic output, clamping, finite coordinates, and documented caps. Static routing review confirmed shared Overlay/Mask/Guide, state/history, Reset, preview/export, Polar, Crystal, Effect Stack, and source-preservation paths remain unchanged.
- The in-app browser security policy blocks local `file://` navigation, so live visual comparison, native PNG/JPEG intake, physical painting, responsive UI inspection, console review, and saved PNG/JPEG inspection remain a required desktop-browser follow-up. No claim is made that those blocked browser-only checks ran in this environment.

Files changed for Milestone #23:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `MILESTONE_ROADMAP.md`

Backup created before editing:

- `kaleidoscope-image-lab.html.backup-20260803-020827-milestone23-pre-number-theory` (SHA-256 matched the pre-edit application)

## 11b. Milestone #24 — Tiling & Tessellation Pack — complete — 2026-08-03

- Added the Tiling & Tessellation pack to the existing Pattern Pack selector with eight bounded normalized vector families: Triangular Grid, Hexagonal Grid, Square Truchet Tiles, Triangle Truchet Tiles, Islamic Star Grid, Voronoi Cells, Penrose-Inspired Rhombus Tiling, and Archimedean Tiling Set.
- Square and Triangle Truchet use deterministic seeded orientation with compact visual variants. Voronoi uses deterministic seeded points and a small half-plane clipping helper to produce outline polygons without storing cells in state or exposing an editor. Penrose is intentionally a bounded visual approximation, not an exhaustive mathematical Penrose engine. Islamic Star Grid is explicitly inspired rather than historically exact.
- Archimedean Tiling Set provides Trihexagonal, Snub Square, Truncated Square, and Rhombitrihexagonal variants through the existing active-only parameter control renderer. Existing Pattern Pack controls provide scale, rotation, center, line presentation, opacity, and use mode; only relevant Cell Size, Density, Cell Count, Seed, and Variant controls appear per pattern.
- Conservative builder caps prevent unbounded work: 512 triangular cells, 361 hexagons, 324 square Truchet tiles, 512 triangle Truchet tiles, 338 Islamic-star polygons, 64 Voronoi cells/seeds, 240 Penrose-inspired rhombi, and 225 Archimedean motifs. All ranges are clamped inside the generators.
- No rendering architecture changed. The shared cached Pattern Pack stage continues to use one normalized command route for Overlay, alpha-preserving Mask, and non-exporting Painting Guide. Pattern parameters—not generated arrays—remain the reversible history payload, so control changes, pack/pattern selection, Undo/Redo, Reset Pattern, full Reset, preview, fallback, and original-resolution PNG/JPEG export stay on the existing paths.
- QA performed: created `kaleidoscope-image-lab.html.backup-20260803-tiling-tessellation-pre` before editing; decoded and validated the outer `srcdoc`; parsed all 8 embedded JavaScript blocks; confirmed 195 unique DOM IDs; verified each new builder at deliberately out-of-range settings for finite deterministic output and its cap; and reviewed the limited diff against the backup. The workspace has no usable Git worktree, so Git diff/status checks are unavailable.
- The in-app browser security policy blocks local `file://` navigation in this environment. Physical visual checks for every pattern and variant, Overlay/Mask/Guide interaction, Undo/Redo, Reset Pattern, narrow layouts, browser console, and independent saved PNG/JPEG comparison remain desktop-browser follow-ups. No claim is made that those blocked browser-only checks ran here.

Files changed for Milestone #24:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `MILESTONE_ROADMAP.md`

`DEVELOPMENT_MANUAL.md` was reviewed and not changed because the shared Pattern Pack contract and rendering architecture were preserved.

## 11c. Milestone #25 — Wave-Interference Pattern Pack — complete — 2026-08-13

- Added the Wave Interference pack to the existing Pattern Pack selector with eight bounded normalized vector families: Concentric Waves, Dual-Source Ripple Interference, Standing Wave Grid, Radial Interference Field, Moire Line Field, Circular Moire Rings, Harmonic Stripe Field, and Chladni-Inspired Contour Pattern.
- Each pattern reuses the shared active-only metadata controls for frequency, phase offset, source distance, density, strength/amplitude, angle difference, harmonic count, or X/Y frequencies as relevant. Existing shared controls continue to own scale, rotation, center X/Y, line width, line color, opacity, use mode, and Reset Pattern. No seed is exposed because all eight families are deterministic without randomized placement.
- Added a small backward-compatible sampled-field helper using bounded marching-squares-style line contours. Dual-Source Ripple Interference and the Chladni-inspired pattern use the helper; the remaining families build normalized line segments directly. Generator math remains separate from the shared `SacredGeometryStage` styling and rendering path, and generated command arrays never enter reversible state or history.
- Safe limits are enforced inside the builders: 40 concentric rings / 3,840 segments; 72 × 72 sampled cells, six contour levels, and 12,000 segments for dual-source interference; 24 standing-wave bands / 2,304 segments; 24 radial contours / 3,072 segments; 80 moire lines; 64 circular moire rings / 6,144 segments; five harmonics, 70 stripes, and 5,040 segments; and 64 × 64 sampled cells, seven contour levels, and 12,000 segments for Chladni-inspired contours. Unsafe numeric settings are clamped before generation.
- The established cached Pattern Pack stage remains the only presentation and routing layer. Overlay and alpha-preserving Mask run after Crystal Mode and before the Effect Stack for preview, fallback, and original-resolution export. Painting Guide draws after preview rendering and remains absent from exports. Parameters—not sampled fields—remain in the existing nested `pattern` state, so completed edits create one Undo entry and Undo/Redo, Reset Pattern, full Reset, prior states/presets, and older Pattern Packs retain their established behavior. Smart Randomizer, presets, and Mutate continue to preserve Pattern Pack settings.
- No Wolfram call or runtime dependency was needed. The fields and contours use compact native JavaScript approximations rather than a physics, shader, audio-reactive, or custom-field system.
- Static QA: confirmed one intact outer `srcdoc`; decoded the embedded document; parsed all 8 embedded JavaScript blocks; confirmed 193 unique DOM IDs; verified all eight pack definitions/builders; and ran each builder twice at deliberately out-of-range minimum and maximum settings to confirm deterministic, finite, nonempty output and declared caps.
- Localhost Chromium QA used an actual generated opaque PNG source and decoded downloaded PNG artifacts. All eight patterns rendered distinctly at defaults and stayed responsive at minimum and maximum controls. Overlay preview and export sampled hashes matched; Mask preview/export contained both transparent and visible alpha; Painting Guide changed preview pixels but its exported PNG matched the disabled-pattern export byte-for-byte. One-step control Undo/Redo, Reset Pattern plus Undo restoration of the complete wave state, switching through all five earlier Pattern Packs, 480 × 800 responsive reachability/no overflow, and a clean console also passed.
- Known limitations: the Chladni and interference contours are visually useful bounded approximations rather than physically exact simulations. Sampling and full-resolution export remain synchronous main-thread work. Headless browser QA did not exercise native file-picker/drop UI, physical painting or pan/wheel gestures, the platform Save As picker, JPEG background compositing, or an independent desktop-browser visual review.

Files changed for Milestone #25:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `MILESTONE_ROADMAP.md`

Backup created and checksum-verified before editing:

- `kaleidoscope-image-lab.html.backup-20260813-210718-milestone25-pre-wave-interference` (SHA-256 `3579ad57a4af5aa22d1803251f09732babe2894158b263cb2da0c9f27f1e3c92`)

`DEVELOPMENT_MANUAL.md` was reviewed and not changed because the shared Pattern Pack contract and rendering architecture were preserved.

### Final feature-freeze UI/UX polish — 2026-08-18

- Completed a conservative final UI/UX polish pass without starting a milestone or changing rendering, state, animation, project, Pattern Pack, persistence, history, or export architecture.
- Added one shared contextual-tooltip registry to the existing Floating UI hover/focus mechanism, with concise action-oriented guidance for Randomize/Mutate, locks, Pattern Pack, Polar, Crystal, Painting Guide, animation and recording, Effects, Discovery, Favorites, Presets, and Project actions. Existing `title` values remain as native fallback guidance, and tooltip triggers expose `aria-describedby` while open.
- Tightened visible terminology with `Mutate`, `Pattern Scale`, `Pattern Rotation`, and `Pattern Center X/Y`; replaced the stale geometry future-milestone placeholder with a compact no-additional-settings message. Reset actions remain beside the controls they affect.
- Added small narrow-layout adjustments for Discovery and Favorites action rows and tuned the dark tooltip surface to match the Optical Instrument palette. The canvas-first workspace rail, sticky Animation Playback group, collapsible sections, and single-control ownership remain unchanged.
- Static QA passed: one intact outer `srcdoc`, decoded inner document, all three decoded inline scripts parsed, 244 unique DOM IDs with no duplicates, and `git diff --check` passed.
- Localhost Chromium QA passed with a generated blank canvas: 199 tooltip triggers including dynamic animation and Organic Motion controls, hover and keyboard-focus tooltip exposure, clean tooltip dismissal, all six workspace panels, sticky Animation Playback, Play/Pause/Stop restoration status, Library and Procedural Preset reachability, New Project confirmation/cancel, 480px no-overflow inspection, and no application console/page errors. Recording entered and left its active state, but headless Chromium finalized without video data; saved recording artifact verification remains a physical desktop-browser check. Decorative CDN payloads were replaced with successful empty responses in the harness so the standalone document could initialize without network stalls; external CDN delivery itself was not revalidated.
- Physical desktop-browser checks remain appropriate for native file-picker/drop behavior, real painting and pointer/wheel gestures, platform Save As/download inspection, JPEG compositing, and independent visual review.

Files changed:

- `kaleidoscope-image-lab.html`
- `PROJECT_NOTES.md`
- `DEVELOPMENT_MANUAL.md`

Backup created before editing:

- `kaleidoscope-image-lab.html.backup-20260818-feature-freeze-ui-pre`

## 12. Constraints for future changes

- Do not change Mirror Lab renderer math unless explicitly requested.
- Preserve original-resolution export; never regress export to the visible preview canvas.
- Preserve slider/number synchronization, pointer coordinate conversion, pointer capture, and history grouping.
- Preserve the two-mode separation and mode-specific control visibility.
- Keep `PostProcessingPipeline` centralized after geometry output so preview, export, fallback rendering, and future effects share one ordered, alpha-preserving path. Keep the empty stack and neutral entries visually inert.
- Keep geometry implementations independent and registered through `GeometryManager`; do not add geometry-specific conditional chains to the shared renderer.
- Preserve `geometryId` in reversible state, preset normalization, Reset, Randomize, and future state serialization.
- Preserve the traced primary emblem, Optical Instrument design system, live aperture indicator, canvas-first hierarchy, and responsive workspace behavior unless a new design requirement supersedes it.
- Preserve the standalone wrapper’s full-viewport iframe sizing and `allow-downloads` permission.
- Keep the preset library centralized and reuse `applyPreset()` rather than scattering preset behavior across event handlers.
- Organize future controls by user workflow: Source, Symmetry, Paint, Animation, Effects, or Library. Do not add a new top-level panel until the feature has been evaluated against these workspaces.
- Keep `activeWorkspace` UI-only. Workspace navigation must never reset render controls, reload a source, clear painting, interrupt future animation, or create history entries.
- Keep animation as a temporary render-state composition layer. Do not write per-frame modulation into controls/history or create a separate preview/export renderer; future motion systems should extend the same bounded timing and render-state pattern.
- Preserve one authoritative control node per setting; do not duplicate controls across workspaces with separate state.

## 13. Documentation

- **Current state:** Read `PROJECT_NOTES.md` for the implemented architecture, testing history, known limitations, and constraints.
- **Long-term standards:** Read `DEVELOPMENT_MANUAL.md` for the project vision, development philosophy, architecture overview, workflow, QA, performance, UI, backlog, changelog, and lessons-learned guidance.
- **Future milestones:** Read `MILESTONE_ROADMAP.md` for permanent milestone IDs, current execution priorities, goals, dependencies, status, and testing guidance. Existing DOCX prompt packs already use the permanent original IDs.
