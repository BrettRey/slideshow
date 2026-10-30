Thank you, the first version of slideshow.html is a solid base: the layout, settings, sorting and file handling are what Brett asked for, and nothing in it touches the network or the disk, which is the point of the app. A careful read turned up sixteen defects in the script. Please fix them.

The HTML and CSS stay exactly as they are (same element IDs, same class names: `.stage`, `.layer`, `.cover`, `.hide-cursor`, `.tile`, `.thumb`, `.broken`, `.drop-before`, `.drop-after`, `.hud.show`, and so on). You are rewriting only the contents of the `<script>` block, which is given in full at the end of this message. The original constraints still hold: vanilla JS, no network, no file writes, object URLs only, localStorage for settings only.

## Defects to fix

1. **Multi-item drops lose items.** `handleDrop` awaits inside its loop over `e.dataTransfer.items`; the browser empties that list as soon as the event handler yields, so only the first dropped item is processed. Fix: before any `await`, synchronously turn every item into either its entry (`webkitGetAsEntry()`) or, if that is null, `getAsFile()`, and collect them in an array. Process the array afterwards.

2. **Dropped folders come back empty or partial.** `readDir` resolves as soon as `readEntries` returns an empty batch, but its `e.file(cb)` calls and its recursive `readDir(sub).then(...)` are asynchronous and never awaited. Fix: make it `async`, promisify `readEntries` and `entry.file`, and `await` every file and every subdirectory (e.g. with `Promise.all`) before returning.

3. **Dropped-folder files sort by bare name.** Files from `entry.file()` have an empty `webkitRelativePath`. Carry each entry's `fullPath` (leading `/` stripped) alongside its File, sort the drop by that path, and store it as the image's `path`. `addFiles` should accept items of the form `{file, path}`; picker files use `file.webkitRelativePath || file.name` as before.

4. **Navigating during a transition corrupts the layers.** `finishRunning()` calls `finish()`, but `onfinish` fires asynchronously, so the layer swap in `done()` has not happened when the next transition starts. The new image is then loaded into the layer that is fading in, and later the stale `done()` swaps the layers again and schedules another timer. Fix: store the running transition's completion function in `show.pending`; `finishRunning()` calls it synchronously; guard the completion so it runs at most once and the later `onfinish` does nothing.

5. **Stale async runs.** Rapid key presses start several transitions whose `decode()` promises resolve later, and each one runs. Fix: increment `show.token` for each transition; `run()` and the completion function return immediately if their token is no longer current.

6. **Black screen after switching transition type.** The `fill: 'forwards'` animations are never cancelled, so their end values keep overriding the inline styles. After any crossfade, a Cut (which only writes inline styles) leaves a layer stuck at opacity 0. Fix: in the completion function, write the final inline styles first, then `cancel()` every animation of that transition.

7. **Crossfade dips toward black at its midpoint**, because both layers fade at once and each has an opaque black background. Fix: put the incoming layer on top (`style.zIndex = 2`, outgoing `1`). For Crossfade, Zoom and Ken Burns animate only the incoming layer (opacity 0 to 1; Zoom also scales 1.15 to 1); the outgoing layer stays at opacity 1 until completion, when it is set to opacity 0. Slide keeps moving both layers.

8. **Ken Burns leaks into other transitions.** Its `fill: 'forwards'` animation is cancelled only when the next Ken Burns starts, so after switching to another transition a layer keeps `scale(1.12)` for later images. Fix: keep one Ken Burns animation per layer. At each transition's completion, cancel the outgoing layer's Ken Burns animation. Start the incoming layer's Ken Burns animation when the transition *starts* (so the image is already moving as it fades in), and only if the transition setting is Ken Burns. Also start it on the first slide in `startShow` when Ken Burns is selected. Pausing pauses and resuming resumes the visible layer's Ken Burns animation.

9. **The caption never changes after the first slide.** Call `updateCaption()` whenever the current slide changes.

10. **The cursor never hides.** The CSS rule is `.stage.hide-cursor`, but the script toggles `hide-cursor` on `document.body`. Toggle it on `stage`.

11. **The HUD should appear on handled keys too** (Space, arrows, F), not only on unhandled keys. H still toggles it.

12. **Double-click starts at the wrong slide** when broken images precede the clicked tile, because the tile passes its index in `images` while `startShow` indexes into `showable()`. Fix: pass the image id and look up its index in `showable()`.

13. **Undecodable images reach playback.** Thumbnails load lazily, so offscreen images may never have been marked broken. When `decode()` fails during playback, mark that image broken, re-render the grid (it is hidden behind the stage), and show the next showable image instead; stop the show if none remain. Load the first slide in `startShow` through the same decode path.

14. **Empty-list crashes.** Guard `transition()` and `startShow()` against an empty `showable()` list.

15. **Tile drags can be mistaken for file drops.** The thumbnail `<img>` is draggable by default, so a drag that starts on the picture is an image drag, and in Chrome its `dataTransfer.types` can include `"Files"`, which the window-level handler treats as a file drop. Fix: set `img.draggable = false`; track the internal drag in a variable (`dragId`, set on `dragstart`, cleared on `dragend`); the window-level file-drop handlers ignore every event while it is set; the tile handlers use it instead of reading `text/plain`.

16. **A leftover timer can fire mid-transition.** When a transition starts, clear the pending advance timer.

## Output format and stopping rule

Return ONLY the complete new contents of the `<script>` block, between these two marker lines, with nothing before or after them, and no `<script>` tags:

===BEGIN script===
"use strict";
// NOTES: ...
...
===END script===

The `// NOTES:` line must list, by number, any defect you could not fix or are unsure you fixed; write `// NOTES: none` if all sixteen are fixed. Write the script once, completely, with no stubs or `// TODO` placeholders. Change only what the fixes need; leave working code alone. If one of the defects above is wrong about the code, say so in NOTES rather than forcing a change.

## Current script


(The full prompt appended the v1 script, i.e. the <script> block of qwen-v1.html, in a js code fence.)
