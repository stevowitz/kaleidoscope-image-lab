# Kaleidoscope Image Lab

## Project

Self-contained browser image editor for turning loaded or painted sources into mirror, radial, geometric, procedural-pattern, organic, and crystalline compositions. It supports reversible controls/history and PNG/JPEG export.

## Structure and runtime

- `kaleidoscope-image-lab.html` is the authoritative application source and browser-ready entry point.
- The file is a standalone wrapper containing a full-viewport sandboxed iframe (`allow-scripts allow-downloads`). The iframe `srcdoc` contains the complete embedded HTML, scoped CSS, and JavaScript; there are no source modules, package manifest, framework, build configuration, or runtime dependency manifest.
- `Images and logos/` contains the supplied traced emblem and logo assets. `docs/` contains DOCX reference manuals/prompt packs. `PROJECT_NOTES.md`, `DEVELOPMENT_MANUAL.md`, and `MILESTONE_ROADMAP.md` are the project documentation sources of truth for current state, standards, and milestone order.
- The app uses browser-native DOM, Canvas 2D, file/clipboard/drag-and-drop, and download APIs. CDN scripts currently pinned in the HTML are GSAP 3.12.5, ScrollTrigger 3.12.5, Floating UI Core 1.7.3, Floating UI DOM 1.7.4, and Lucide 1.17.0.
- No language, framework, runtime, package-manager, or dependency versions are declared beyond the browser APIs and CDN versions above. Do not introduce a version requirement without repository configuration or an explicit product decision.

## Architecture and conventions

- The embedded app is scoped to `#kaleidoscope-image-lab` and runs inside one IIFE. Preserve the outer-document/inner-`srcdoc` escaping when changing HTML, CSS, JavaScript, selectors, template strings, or SVG.
- `LayerManager` is the shared rendering pipeline. Registered geometry implementations go through `GeometryManager`; preview, fallback, Painting Mode, and original-resolution export must continue through the same renderer path.
- The processing order is Geometry/Layer Manager → Polar Engine → Crystal Mode → Pattern Pack → Effect Stack → preview/export. Pattern Packs use normalized procedural vector commands and bounded deterministic builders; do not create per-pack renderers or export paths.
- `getState()`/`applyState()` are the authoritative reversible state path. Keep rendering state, presets, Randomize/Mutate, Reset, and Undo/Redo synchronized through it. Preview/session-only values must not enter rendering history.
- Preserve the existing naming style: JavaScript variables/functions use `camelCase`, classes use `PascalCase`, constants use `const` with descriptive `camelCase` names, DOM ids use the `kaleido-` prefix and kebab case, and CSS classes use kebab case. Existing embedded CSS/JS is generally two-space indented.

## Required behavior and constraints

- Do not change Mirror Lab renderer math unless explicitly required. Preserve the two-mode separation and mode-specific control visibility.
- Preserve original-resolution export; interactive previews may use the capped 1400px-longest-side cached source, while exports use the original decoded source when available. Keep PNG alpha and export-only JPEG compositing behavior.
- Preserve slider/number synchronization, shared pointer-coordinate conversion, pointer capture, grouped gesture history, and the maximum 80-entry history behavior.
- Preserve the `importRotation = -90` behavior unless real-image testing justifies a deliberate change.
- Keep the traced primary emblem artwork/accessibility, Optical Instrument design system, full-viewport iframe sizing, `allow-downloads`, workspace-based UI, and one authoritative control node per setting.
- Geometry, effect, polar, crystal, and pattern work must retain bounded settings, reusable buffers/plans where established, alpha preservation, and preview/export parity. Avoid per-pixel object allocation, unnecessary canvas copies, and unbounded generator work.
- Treat milestone IDs as permanent historical identifiers. Complete one milestone at a time and update `PROJECT_NOTES.md` with implementation, testing, limitations, and next-step information.

## Development and validation

- There is no project-defined build, test, lint, or package-manager command. Do not invent one; the application is edited and run as a standalone HTML file in a desktop browser.
- Narrow validation after an HTML change: verify the outer HTML and escaped `srcdoc` remain valid, validate every embedded script after decoding the `srcdoc`, then manually smoke-test the changed feature through the shared preview/export path and check the browser console.
- For renderer or milestone changes, also run the applicable regression checks from `DEVELOPMENT_MANUAL.md` and `MILESTONE_ROADMAP.md`: representative JPG/PNG sources, geometry/preset/Randomize/Mutate, drag and wheel interactions, Undo/Redo/Reset, PNG export inspection, workspace switching, and responsive resizing.
- Create or verify a timestamped backup of `kaleidoscope-image-lab.html` before milestone edits, and update `PROJECT_NOTES.md` after implementation and QA. Use permanent milestone IDs in prompts, backups, records, and changelogs.

## Do not edit directly / environment limits

- Do not edit `kaleidoscope-image-lab.html.backup-*`; these are historical pre-edit snapshots. Do not edit the supplied logo artwork in `Images and logos/` unless the product requirement explicitly changes the asset.
- The in-app browser harness does not reliably support local `file://` navigation, physical pointer/wheel gestures into the sandboxed iframe, native file pickers, or saved download artifacts. Desktop-browser testing is still required for those behaviors, file intake/drop, physical painting strokes, responsive inspection, console review, and independent PNG/JPEG file inspection.
- Rendering and full-resolution export are synchronous CPU work on the main thread; large previews/exports can be expensive and 4096×4096 exports can use substantial memory or briefly freeze the UI.
- Animation, favorites, saved presets, and project save/load are not implemented; do not assume those workflows exist.
