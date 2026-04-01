# Changelog

All notable changes to this project are documented here.

---

## [2.0.0] — 2026-04-01

### Added — Editor Mode

- **Edit Mode toggle** (`✏️ Edit` button in toolbar): switches the tool from read-only visualizer to a live agent editor. A pink indicator badge shows when edit mode is active.
- **Intent editing panel**: clicking a node in edit mode opens a full form to edit:
  - Display name
  - Training phrases (chip editor — press Enter or `,` to add)
  - Input contexts (chip editor)
  - Output contexts (name + lifespan rows, `+ Add` button)
  - Quick reply suggestions (chip editor)
  - Text responses (multi-line textarea)
  - Fallback intent checkbox
  - Welcome intent checkbox
- **New Intent modal** (`+ New Intent` button, visible in edit mode): create intents from scratch with all the same fields. Assigns a UUID automatically.
- **Delete intent**: `Delete Intent` button in edit panel with confirmation dialog.
- **Agent rename**: click the agent name pill in the toolbar to rename inline.
- **Export ZIP** (`⬇ Export ZIP` button): generates and downloads a valid Dialogflow ES-importable ZIP containing:
  - `agent.json` — agent metadata
  - `package.json` — version manifest
  - `intents/{name}.json` — intent definition (contexts, responses, events, flags)
  - `intents/{name}_usersays_en.json` — training phrases
  - `entities/` folder (passthrough of any loaded entities)
- **From-scratch agent creation**: if no ZIP is loaded, creating the first intent via the modal unlocks all UI controls so a full agent can be built without any pre-existing file.
- **JSZip integration** (CDN): used for both reading incoming ZIPs and writing exportable ZIPs.

### Changed

- Node click behavior is now mode-aware: view mode → detail panel; edit mode → edit form.
- Toolbar reorganised to accommodate new Edit / New Intent / Export buttons.

---

## [1.1.0] — 2025 (Phase 2 improvements)

### Added

- **Intent Detail Panel**: slide-in panel showing name, contexts, training phrases, responses, and quick replies for the selected node.
- **Entities Panel**: sidebar listing all entity types and their synonym entries.
- **Help Modal** (`?` button): keyboard shortcuts and usage guide.
- **Floating draggable chat emulator**: simulate a conversation with the loaded agent directly in the browser.
- **New node types**: `data` (database/webhook nodes) and `global_fallback`.
- **Node icons**: SVG icons per node type (welcome, fallback, data, etc.).
- **Edge context labels**: edges display the context name that connects two intents.
- **Toolbar row**: consolidated top bar with agent name pill, zoom controls, layout toggle, and stat counters.
- **Entities stat** in toolbar (count of entity types).

---

## [1.0.0] — 2025 (Initial release)

### Added

- Single-file HTML visualizer for Dialogflow ES agents.
- Drag-and-drop ZIP loading (no server required, fully local).
- SVG graph with pan/zoom; nodes coloured by type (welcome, fallback, regular).
- BFS layered layout with collision-aware positioning.
- Animated pulse rings on welcome/fallback nodes.
- Edge routing with arrow markers.
- Zero external dependencies (pure HTML/CSS/JS).
