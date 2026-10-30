Thank you for taking this on. You are writing a small app for Brett, a researcher who wants to drop a pile of images into a browser window and get an adjustable slideshow. The whole point is zero footprint: no copies of his images, no cache folders, no exported files, nothing to clean up afterwards. Closing the tab must leave nothing behind except a few remembered settings.

Write ONE complete file, `slideshow.html`, that does everything below.

## Hard constraints

- Single self-contained HTML file: inline `<style>` and `<script>`. Vanilla JavaScript (ES2020), no frameworks, no libraries.
- No external resources of any kind: no CDN, no web fonts, no icon fonts, no images from URLs. Use the system font stack and plain text or Unicode characters for icons (e.g. ▶ ❚❚ ✕ ⤢).
- No network: no fetch, no XHR, no WebSocket, no service worker.
- No file writes: no downloads, no IndexedDB, no Cache API, no File System Access API.
- Images are referenced with `URL.createObjectURL(file)` and the File objects stay in memory. Call `URL.revokeObjectURL` when an image is removed or the list is cleared.
- `localStorage` may store ONLY the settings (slide duration, transition, transition duration, fit, loop, caption toggle). Never the image list. Wrap every localStorage access in try/catch so the app works when storage is blocked.
- Must work in current Chrome and current Safari when opened directly from a `file://` URL.
- Readable code with brief comments. Aim for 600–900 lines in total.

## Adding images

- Buttons: "Add images" (`<input type="file" accept="image/*,.heic,.heif" multiple>`), "Add folder" (`<input type="file" webkitdirectory multiple>`), "Clear all".
- Drag and drop onto the whole window, with a visible overlay hint while dragging ("Drop images or folders").
- Dropped folders must work: iterate `e.dataTransfer.items`, call `item.webkitGetAsEntry()` synchronously inside the drop handler for every item first, then process. For a directory entry, recurse with `entry.createReader()` and call `reader.readEntries()` repeatedly until it returns an empty batch (Chrome returns at most 100 entries per call). Convert file entries with `entry.file()` wrapped in a Promise. Collect every file from one drop before adding any, and add them in natural filename order (`a.localeCompare(b, undefined, {numeric: true, sensitivity: 'base'})`, comparing the relative path).
- Accept a file if `file.type` starts with `image/` OR its extension is one of jpg jpeg png gif webp avif bmp svg heic heif tif tiff. Silently skip everything else (e.g. `.DS_Store`) but report how many were skipped in a status line.
- Skip duplicates: a file with the same name + size + lastModified as one already in the list is not added again.
- New images are appended to the end of the current sequence.

## The sequence (thumbnail grid)

- Show every image as a thumbnail tile in sequence order: an `<img>` using the same object URL with `object-fit: cover`, `loading="lazy"`, `decoding="async"`; its 1-based position number; its filename (truncated with ellipsis); a remove button (✕).
- Drag a tile onto another to reorder (HTML5 drag and drop with `draggable="true"`; show a visible insertion indicator). Reordering must not be confused with the window-level file drop: only treat a drop as a file drop when `e.dataTransfer.types` includes `"Files"`.
- Sort controls: "Name" (natural sort as above), "Date" (file.lastModified, oldest first), "Shuffle" (Fisher–Yates), "Reverse".
- Double-click a tile to start the show at that image.
- If a thumbnail fails to decode (its `onerror` fires; this happens for HEIC in Chrome), mark the tile with a red outline and the note "can't display in this browser (try Safari)" and skip that image during playback.
- Show a count: "N images" (and "M can't be shown" when relevant).
- When the list is empty, show a large friendly empty state explaining drag-and-drop.

## Settings panel

- Slide duration: range slider plus number input, kept in sync, 1–60 seconds, step 0.5, default 5.
- Transition: select with Cut, Crossfade (default), Slide, Zoom, Ken Burns, Random.
- Transition duration: range slider plus number input, 0.2–5 seconds, step 0.1, default 1.
- Fit: Contain (default; whole image visible, letterboxed) or Cover (fills screen, crops).
- Loop: checkbox, default on.
- Show filename caption during the show: checkbox, default off.
- Changes take effect immediately, including during playback (the next slide uses the new values).
- A large "▶ Play" button, disabled when there are no showable images.

## Playback

- Playing opens a stage: a `position: fixed; inset: 0` element with a black background, containing two stacked `<img>` layers used alternately for transitions. Call `stage.requestFullscreen()` (with the `webkitRequestFullscreen` fallback for Safari); if fullscreen is refused, still play in the full-window stage.
- Before each transition, load the next image into the hidden layer and `await img.decode()` (catch errors) so transitions never show a half-loaded image.
- Use the Web Animations API (`element.animate`) for all transitions so they can be cancelled. If the user navigates while a transition is running, call `.finish()` on the running animations first, then start the new transition.
  - Cut: swap instantly.
  - Crossfade: incoming opacity 0→1 while outgoing goes 1→0.
  - Slide: incoming translateX(100%)→0 while outgoing 0→-100%. Reverse the direction when going to the previous slide.
  - Zoom: incoming scale(1.15) and opacity 0 → scale(1) and opacity 1, while outgoing fades out.
  - Ken Burns: crossfade, and every slide while visible slowly scales from 1.0 to 1.12 with a small random translate (up to ±3%), over (slide duration + transition duration), easing linear.
  - Random: pick Crossfade, Slide or Zoom at random for each change.
- Timing: after a transition completes, wait the slide duration, then advance. Any manual navigation resets the timer.
- Keys: Space toggles pause, Right arrow next, Left arrow previous, Escape exits the show, F toggles fullscreen, H toggles the HUD. Exiting fullscreen with the browser's own control (the `fullscreenchange` / `webkitfullscreenchange` event with no fullscreen element) must also stop the show and return to the grid. Clicking the right half of the stage goes next, the left half previous.
- Pausing stops the timer and pauses any running Ken Burns animation (`animation.pause()`); resuming continues it.
- HUD: a small translucent overlay at the bottom showing "n / N", the filename, and "Paused" when paused. It appears on mouse move or key press and fades after 2 seconds. Hide the cursor while the HUD is hidden.
- Caption: if enabled, show the filename in the bottom-left corner of the stage at all times during the show.
- End of sequence: if Loop is on, wrap to the first image; if off, stay on the last image and show "End" in the HUD.

## Look

Dark neutral UI (near-black background, light grey text, one accent colour), system font stack, comfortable spacing, responsive grid (`grid-template-columns: repeat(auto-fill, minmax(140px, 1fr))`). The settings panel and buttons sit in a toolbar above the grid.

## Output format and stopping rule

Return ONLY the complete file, between these two marker lines, with nothing before or after them:

===BEGIN slideshow.html===
<!doctype html>
<!-- NOTES: ... -->
...
===END slideshow.html===

The `NOTES` comment directly after the doctype must list any requirement you did not implement or are unsure works, one short phrase each. Write `NOTES: none` if everything is implemented. Write the file once, completely. Do not leave stubs or `// TODO` placeholders; if a feature would take you over the length budget, leave it out and name it in NOTES instead of half-implementing it. Saying a part could not be done is a perfectly good answer.
