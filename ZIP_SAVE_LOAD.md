# ZIP Save/Load + Privacy Splash

## Goal
Provide an offline-first save/load mechanism where **all case data, attachments, screenshots, and notes live only on the user’s device**. Users explicitly export a ZIP file and can later import that ZIP to restore the case. **No cloud storage** and no server uploads are involved.

This app already implements the pattern; use it as the blueprint.

## User-Facing UX
- **Top bar buttons**: “Save ZIP” and “Load ZIP”.
- **Privacy Notice splash**: On app launch, show a modal that states data stays on the user’s computer; only network traffic is loading the app itself. After the user closes the modal, tutorials can start.
- **Import flow**: “Load ZIP” opens a hidden file picker restricted to `.zip`.

Relevant UI elements:
- `index.html` has `#saveZipBtn`, `#loadZipBtn`, and a hidden `<input type="file" id="zipFileInput" accept=".zip" hidden>`.
- Privacy modal: `#privacyModal` in `index.html` (see text in the modal body).

## Core ZIP Implementation (main.js)
### Save
`saveZip()` creates a ZIP file with:
- `mediation_data.json` — the serialized app state **without** inline attachment data.
- `/attachments/` — each attachment saved as a separate file, base64-encoded.

Key steps:
1. `exportStateWithAttachments()`:
   - Deep-copies `appState`.
   - Removes any transient round data (`rounds`, `currentRoundId`).
   - Walks issues and pre-mediation notes.
   - Extracts each attachment into a dedicated list with unique `fileKey` names (sanitized + de-duplicated).
   - Strips `data` from each attachment in the state copy.
2. ZIP assembly:
   - `zip.file('mediation_data.json', JSON.stringify(exportData.state, null, 2))`
   - `attachments/` folder contains each file as raw base64 (no data URI prefix).
3. Download:
   - Creates a blob, builds an object URL, and triggers a download link.
   - Filename uses case name + ISO date, with invalid filename chars replaced.

Code references:
- `main.js`: `saveZip()`, `exportStateWithAttachments()`, `stripAttachmentData()`, `getFileExtension()`

### Load
`loadZip(file)` reverses the process:
1. `JSZip.loadAsync(file)`
2. Reads `mediation_data.json` as text ? `state`.
3. Reads `/attachments/` folder entries, decoding base64 into a lookup object.
4. `importStateWithAttachments(state, attachments)`:
   - Restores `appState`.
   - Rebuilds `data:` URLs for each attachment using `buildDataUrl()`.
   - Normalizes status, removes obsolete round fields.
5. Calls `saveDraft()` and `renderAll()`.

Code references:
- `main.js`: `loadZip()`, `importStateWithAttachments()`, `buildDataUrl()`

### Attachment Rules
- Attachments are stored in state as objects with `name`, `type`, and `data` fields.
- On export: attachment `data` is stripped from `mediation_data.json` and saved separately in `/attachments/`.
- On import: attachments are rehydrated into `data:` URLs for display/use.
- Filename safety: `fileKey` is sanitized (no `< > : " / \ | ? *`) and de-duped.

## Privacy Splash Modal
On app initialization:
- `init()` calls `openModal(modals.privacy)`.
- Closing the modal triggers tutorial start.

Code references:
- `main.js`: `init()`, modal close handlers.
- `index.html`: `#privacyModal` content.

## No-Cloud Guarantee
- There is **no server API** for saves/loads.
- The only network calls are static asset loads (HTML/CSS/JS/JSZip/HTML2Canvas) and optional `presets.json` fetch.
- Draft autosave uses `localStorage` only.

## Implementation Notes for Another App
To recreate this feature in another codebase:
1. Add JSZip (CDN or bundled).
2. Add Save/Load buttons and a hidden file input for ZIP.
3. Implement:
   - `exportStateWithAttachments()` to separate binary data from JSON.
   - `saveZip()` to write JSON + attachments into a ZIP and download.
   - `loadZip()` to parse ZIP, rebuild attachments, and hydrate app state.
4. Add a privacy notice modal shown at first load, stating:
   - All data stays on the user’s computer.
   - App works offline after initial load.
   - No data is transmitted to a server.

## Files to Copy or Reference
- `index.html` (buttons, file input, privacy modal)
- `main.js` (save/load/export/import logic)
- `modules/attachments.js` (attachment handling utilities)

