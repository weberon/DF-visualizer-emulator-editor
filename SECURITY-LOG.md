# Security Log

Documents security decisions, mitigations, and known considerations for `dialogflow-visualize.html`.

---

## Architecture: Local-only processing

**All data stays in the browser.**

- No server, no network requests for user data.
- Uploaded agent ZIPs are read via the [File API](https://developer.mozilla.org/en-US/docs/Web/API/FileReader) entirely in memory.
- Exported ZIPs are generated in memory and downloaded via `URL.createObjectURL()` — never uploaded anywhere.
- The only outbound network request is loading JSZip from a CDN (see CDN section below).

---

## XSS mitigations

### `esc()` function

All user-supplied strings (intent names, training phrases, context names, entity values) are passed through `esc()` before being interpolated into HTML:

```js
function esc(s) {
  return String(s)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;');
}
```

This prevents stored XSS from malicious agent ZIP content being rendered as live HTML.

### innerHTML usage

`innerHTML` is used for rendering SVG nodes and detail/edit panels. All dynamic values are routed through `esc()`. Static structural HTML (toolbar, modals) is hardcoded and contains no user data.

### `textContent` for agent name

The agent name pill text node is set via `textContent`, not `innerHTML`, when restoring from the rename input. This is an additional safe guard for that specific field.

---

## ZIP parsing safety

JSZip's `loadAsync()` is used to parse uploaded ZIPs. Considerations:

- **Zip bombs**: JSZip has no built-in decompression size limit. A maliciously crafted ZIP with high compression ratio could exhaust browser memory. Mitigated by: the tool targets known Dialogflow ES agent ZIPs (typically < 5 MB uncompressed); no mitigation for intentionally crafted input.
- **Path traversal**: JSZip returns file paths as strings. The tool only reads files matching known patterns (`intents/*.json`, `entities/*.json`, `agent.json`). No paths are used to write to the filesystem (browser cannot do that).
- **Arbitrary file execution**: Not possible — browser JS cannot execute files from a ZIP.

---

## CDN dependency

JSZip is loaded from:
```
https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js
```

**Risk**: CDN compromise would allow injecting arbitrary JS into the tool.

**Mitigation options** (not currently implemented):
- Self-host `jszip.min.js` alongside the HTML file — eliminates CDN dependency entirely.
- Add a `<script integrity="sha384-...">` SRI hash to detect tampering.

**Current posture**: Acceptable for an internal/developer tool not exposed on a public server. If deploying publicly, self-host JSZip.

---

## `crypto.randomUUID()`

Used to generate intent IDs for newly created intents. `crypto.randomUUID()` is a browser-native CSPRNG — no external entropy source needed, cryptographically random, no known collision risk for UUIDs in this context.

Requires a [secure context](https://developer.mozilla.org/en-US/docs/Web/Security/Secure_Contexts) (HTTPS or `localhost`). When opened via `file://` in Chrome/Edge this is still considered secure; Firefox `file://` may vary.

---

## `URL.createObjectURL()` and memory

The export function creates an object URL for the ZIP blob and revokes it immediately after the download link is clicked:

```js
const url = URL.createObjectURL(blob);
a.href = url;
a.click();
setTimeout(() => URL.revokeObjectURL(url), 1000);
```

This prevents the blob from persisting in memory longer than necessary.

---

## Audit history

| Date | Version | Finding | Status |
|---|---|---|---|
| 2026-04-01 | 2.0.0 | Initial security review for editor release | Documented above |
