# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## What this project is

A single-file HTML tool (`dialogflow-visualize.html`) that can:
1. **Visualize** a Dialogflow ES agent as an interactive SVG graph
2. **Emulate** conversations via a floating chat panel
3. **Edit** the agent (add/modify/delete intents, rename agent)
4. **Export** a Dialogflow ES-importable ZIP file

No build step. No server. Open the HTML file directly in a browser.

---

## Running

```bash
# Just open in browser — no server needed
xdg-open dialogflow-visualize.html         # Linux
open dialogflow-visualize.html             # macOS
```

For drag-and-drop ZIP loading to work, the file must be opened via `file://` or a local server. Chrome/Edge work fine with `file://`; Firefox may need `--allow-file-access-from-files`.

---

## Architecture

Everything lives in **one HTML file**. Structure:

```
<style>   CSS variables + all component styles
<body>    Toolbar, SVG canvas, side panels, modals
<script>  All logic — ~1500 lines of vanilla JS
```

### Data pipeline

```
ZIP drop → JSZip.loadAsync()
         → parseDialogflow(files)     builds allIntents[], allEntities[]
         → buildEdges(allIntents)     context-name matching → edges[]
         → computeLayout(nodes,edges) BFS layers + collision avoidance → x,y
         → renderGraph()              draws SVG nodes + edges
```

### Key global state

| Variable | Type | Description |
|---|---|---|
| `allIntents` | `Intent[]` | Source of truth — mutated by editor actions |
| `allEntities` | `Entity[]` | Loaded entity types (passed through to export) |
| `agentName` | `string` | Editable agent display name |
| `editMode` | `boolean` | Toggles click behaviour: view panel vs edit form |
| `editDraft` | `Intent\|null` | Deep copy of the intent being edited |
| `nodes`, `edges` | arrays | Computed graph state, rebuilt after every edit |

### Intent schema (internal)

```js
{
  id: string,           // UUID
  name: string,         // display name = filename without extension
  trainingPhrases: string[],
  inputContexts: string[],
  outputContexts: [{name: string, lifespan: number}],
  responses: string[],  // text response lines
  quickReplies: string[],
  isFallback: boolean,
  isWelcome: boolean,   // detected by WELCOME event
  isData: boolean,      // detected by webhook/data naming convention
}
```

### Export ZIP structure (Dialogflow ES)

```
agent.zip
├── agent.json
├── package.json
└── intents/
    ├── {name}.json
    └── {name}_usersays_en.json
```

`exportZip()` builds this with JSZip and triggers a browser download via `URL.createObjectURL()`.

---

## Editor functions reference

| Function | Purpose |
|---|---|
| `toggleEditMode()` | Flips `editMode`, shows/hides Edit buttons, changes cursor |
| `showEditPanel(id)` | Deep-copies intent → `editDraft`, renders edit form in detail panel |
| `renderEditForm(draft)` | Returns HTML string for the edit form |
| `saveIntent()` | Reads form DOM → updates `allIntents[idx]` → rebuilds graph |
| `deleteIntent(id)` | `confirm()` → filters `allIntents` → rebuilds graph |
| `openNewIntentModal()` | Shows `#modal-new-intent` |
| `createIntentFromModal()` | Reads modal form → `crypto.randomUUID()` → pushes to `allIntents` |
| `exportZip()` | Builds and downloads Dialogflow ES ZIP via JSZip |
| `chipKeydown(e, editorId)` | Enter/comma adds a chip to a chip editor |
| `getChips(editorId)` | Reads chip values from chip editor DOM |
| `getOutputContexts(listId)` | Reads name+lifespan rows from output-context list |
| `startRenameAgent()` | Swaps agent name pill for `<input>` |
| `finishRenameAgent()` | Reads input → updates `agentName` → restores pill |

---

## Chip editor pattern

Arrays (training phrases, input contexts, quick replies) use a **chip editor**:
- `Enter` or `,` adds a chip
- `×` button on each chip removes it
- `getChips(editorId)` reads all current values from the DOM
- `chipHtml(value)` generates a chip element HTML string

Output contexts use a separate **oc-row** pattern (two inputs per row: name + lifespan).

---

## Context linking = edges

Intents are connected when an **output context name** on intent A matches an **input context name** on intent B. This is how `buildEdges()` works and how users should connect intents in the editor. There is no drag-to-connect UI — context names are the wiring.

---

## Dependencies

- **JSZip** — loaded from CDN (`https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js`)
- Everything else: vanilla HTML/CSS/JS, no build toolchain

---

## Branch

Active development branch: `Gold`

---

## Sample agents for testing

See `/home/rhn/df/` for sample Dialogflow ES agent ZIP files used during development.
