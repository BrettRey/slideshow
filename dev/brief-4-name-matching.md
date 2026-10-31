Thank you once more. Your remembered-order feature works, and testing passed. One gap has come up. Brett asked for the order to be remembered by file name, but it matches files by their full identity key (`name|size|lastModified`). So a photo he has edited or exported again (same name, new size or date) is not recognised, and the restore offer silently doesn't appear for it. Please make the matching fall back to the file name, safely.

You are rewriting only the block below: the two functions `updateRestoreOffer()` and `restoreOrder()`. Return a replacement for exactly this block. You may add one helper function inside it. Nothing else in the script changes, and the saved format stays the same.

Things the rest of the script provides, which you can use:
- `images`: the current list, in sequence order; each item has `.file` (a File) and `.name` (the file name).
- `keyOf(file)`: returns `file.name + '|' + file.size + '|' + file.lastModified`.
- `pendingOrder`: the saved array of such keys, or `null`.
- `restoreBar`, `restoreMsg`, `renderGrid()`, `setStatus(text)`.

## Matching rule

Write a helper, `matchSaved()`, that assigns current images to positions in `pendingOrder`, and use it in both functions.

1. **Exact first.** An image whose `keyOf(im.file)` equals a saved key is matched to that key's index.
2. **Then by name, only when unambiguous.** For each image still unmatched, take the saved keys that are still unmatched and compare names. The name in a saved key is everything before the last two `|` fields (`key.split('|').slice(0, -2).join('|')`), because a file name can itself contain `|`. Match the image to a saved key by name only if that name occurs exactly once among the unmatched saved keys **and** exactly once among the unmatched current images. Otherwise leave it unmatched. Two different files that happen to share a name (IMG_0001.jpg from two cameras) must never be matched by guesswork.
3. Return the positions (for example a `Map` from image to saved index) and the number matched by name.

## Using it

- `updateRestoreOffer()`: same behaviour as now, but using `matchSaved()`. Show the bar only if at least 2 images match and the matched images, in their current order, are not already in increasing saved-index order. Message: `Restore the order from last time? ${matches} of ${pendingOrder.length} images found.` When some were matched by name, append ` (${byName} by name only, edited or re-exported)`. In all other cases hide the bar and leave `pendingOrder` as it is.
- `restoreOrder()`: matched images first, sorted by saved index, then unmatched images in their current relative order. Then, as now: `pendingOrder = null`, hide the bar, `renderGrid()`, `setStatus('Order restored')`.

## Output format and stopping rule

Return ONLY the replacement block between these marker lines, nothing before or after:

===BEGIN block===
  // NOTES: ...
  ...
===END block===

Keep the two-space indentation of the surrounding code. The first line is a `// NOTES:` comment listing anything you couldn't do or are unsure of (`// NOTES: none` otherwise). No stubs or TODOs. If this spec is wrong about the code, say so in NOTES.

## Current block


(The full prompt appended the two functions as they then stood in slideshow.html, in a js code fence.)
