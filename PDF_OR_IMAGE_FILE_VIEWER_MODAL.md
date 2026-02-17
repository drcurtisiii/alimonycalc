# PDF or Image File Viewer Modal

## Purpose
Provide a modal-based attachment viewer that can display PDFs and images with scrolling, zoom controls, and a fullscreen mode. This is the in-app experience the user wants replicated.

## User-Facing Behavior
- Clicking an attachment opens a modal that displays the file (PDF or image) in a scrollable viewport.
- PDFs render inside the modal with dedicated controls: previous/next page, zoom in/out, and a page counter.
- Images display at full available size with scroll for overflow; they can be zoomed in fullscreen via Ctrl+mouse wheel.
- A Fullscreen button expands the attachment display; users can keep scrolling and zooming while fullscreen.
- Keyboard controls work for navigation:
  - Non-fullscreen: left/right arrows move PDF pages; Escape closes the modal.
  - Fullscreen: Escape exits fullscreen; left/right arrows move PDF pages; up/down arrows move between attachments.
- The viewer supports thumbnail navigation for attachments; PDFs generate a live thumbnail from page 1.

## Where It Lives In This App
- Modal markup and UI controls: `index.html` (Attachment modal section).
- Viewer styles: `index.html` inline styles (Attachment Viewer Modal and PDF.js Viewer Styles).
- Behavior and PDF rendering logic: `main.js` (attachment viewer methods).
- PDF rendering library: PDF.js loaded via CDN in `index.html`.

## Key UI Structure (Markup)
- The attachment modal (`#attachmentModal`) contains:
  - `#attachmentDisplay`: main scrollable viewport that hosts the rendered file.
  - `#attachmentThumbnails`: horizontal strip of thumbnails to switch between attachments.
  - `#fullscreenAttachment`: button to request fullscreen on `#attachmentDisplay`.
  - `#fullscreenAttachmentNav`: overlay nav shown in fullscreen to move between attachments.

See: `index.html` (Attachment modal block around the `#attachmentModal` markup).

## Rendering Flow (main.js)
### 1) Opening the viewer
- `viewAttachments(itemIndex, attachmentType)` sets `currentViewingAttachments` and `currentAttachmentIndex`, then calls `showAttachment(0)` and opens `#attachmentModal`.

### 2) Showing a file
- `showAttachment(index)` decides how to render based on MIME type:
  - Images (`image/*`): build an `<img>` inside a wrapper, replace the display content.
  - PDFs (`application/pdf`): create a wrapper and call `renderPDFWithJS(...)`.
  - Videos/audio/other types are supported but not relevant to this feature.
- After rendering, it updates thumbnails and fullscreen nav, and resets scroll to the top.

### 3) PDF rendering
- `renderPDFWithJS(attachment, display)`:
  - Converts the base64 data URL to a `Uint8Array`.
  - Loads the PDF via `pdfjsLib.getDocument({ data })`.
  - Builds PDF controls (prev/next page, zoom out/in, attachment nav, counters).
  - Renders the current page onto a `<canvas>` inside `.pdf-canvas-container`.
  - Zoom modifies `currentScale` and re-renders the current page.
  - Updates the page count and zoom labels on every render.
  - Adds key handling for left/right arrows (page navigation) and Escape (close modal).

Key code: `main.js` `renderPDFWithJS`, `showAttachment`, `updatePDFAttachmentControls`.

## Fullscreen Behavior
- Fullscreen is applied to `#attachmentDisplay` (not the entire modal) via `requestFullscreen()`.
- `toggleFullscreen()`:
  - If not fullscreen: enters fullscreen, sets default PDF zoom to 100%, installs fullscreen listeners.
  - If fullscreen: exits fullscreen.
- `addFullscreenEventListeners()`:
  - Adds `fullscreenchange` to toggle `.fullscreen-active` class on the display.
  - Adds key handlers (Escape, arrow keys) to control page/attachment navigation.
  - Adds a wheel handler: Ctrl+wheel zooms PDFs (by clicking PDF zoom buttons) or images (CSS transform scale).
- Fullscreen nav overlay (`#fullscreenAttachmentNav`) is shown/hidden by `updateFullscreenNav()`.

Key code: `main.js` `toggleFullscreen`, `setFullscreenZoom`, `addFullscreenEventListeners`, `updateFullscreenNav`.

## CSS/Styling Notes
- `.attachment-display` is a scrollable container with a neutral background and border.
- `.pdf-viewer-container` and `.pdf-canvas-container` are flex layouts to keep canvas centered and scrollable.
- `.attachment-display:fullscreen` changes background and hides page labels.
- Thumbnail strip gets a fixed overlay position when fullscreen (`.attachment-display:fullscreen .attachment-thumbnails`).

See: `index.html` inline styles for Attachment Viewer Modal and PDF.js Viewer Styles.

## Dependencies
- PDF.js is loaded via CDN:
  - `https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js`
- PDF rendering uses the global `pdfjsLib` from that script.

## Implementation Checklist For Another App
- Build an attachment modal with a scrollable display area and a fullscreen button.
- Store attachment list + current index; render based on MIME type.
- Integrate PDF.js, convert base64 to `Uint8Array`, render pages to `<canvas>`.
- Provide PDF controls for page nav and zoom.
- Implement fullscreen on the display container, not the whole modal.
- Add keyboard and wheel handlers for page/attachment navigation and zoom.

## Files To Reference In This App
- `index.html` (modal markup, viewer styles, PDF.js script tags)
- `main.js` (viewer logic: `viewAttachments`, `showAttachment`, `renderPDFWithJS`, `toggleFullscreen`)
