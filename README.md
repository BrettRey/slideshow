# Slideshow

A slideshow for a pile of images that leaves nothing behind. Open `slideshow.html` in a browser, drop in images or folders, and play. The page reads your original files in place through in-memory references (`URL.createObjectURL`). It makes no copies, writes no files and sends nothing over the network, so when you close the tab the images are gone from the page. Between sessions the browser's local storage keeps only your settings and the order of your file names.

## Opening it

```sh
open ~/projects/LLM-CLI-projects/tools/slideshow/slideshow.html
```

That opens it in your default browser. You can also bookmark the page or drag the file onto a browser window. There's no server and nothing to install.

## Adding images

- Drag files or whole folders anywhere onto the window. Subfolders are included.
- Or use **Add images** or **Add folder**.
- Each batch goes on the end of the list in natural filename order (`img2` before `img10`). Non-image files are skipped, and so is a file that's already in the list.
- **Clear all** empties the list (it asks first).

## Arranging the sequence

- Drag a thumbnail onto another to move it; the blue bar shows which side it'll land on.
- **Name**, **Date** (file modification date, oldest first), **Shuffle** and **Reverse** reorder the whole list.
- The ✕ that appears on hover removes a thumbnail.
- Double-click a thumbnail to start the show from it.

## Keeping an arrangement

- While images are loaded, the browser asks before a reload or a closed tab.
- The app remembers the order of the files you arranged (by name, size and modification date, not their contents). If the tab does close, add the same files again: a bar offers to restore the old order, and **Restore order** puts them back. Anything new goes at the end.
- While that offer is open the old order stays saved, so a second accident doesn't lose it. **Dismiss**, or any rearranging of your own (drag, sort, remove, clear), replaces it with the current order.
- Only one arrangement is remembered: the last one you made.

## Settings

| Setting | Options |
|---|---|
| Slide duration | 1 to 60 seconds |
| Transition | Cut, Crossfade, Slide, Zoom, Ken Burns (slow zoom and drift while each image is up), Random (crossfade, slide or zoom, picked per slide) |
| Transition duration | 0.2 to 5 seconds |
| Fit | Contain shows the whole image; Cover fills the screen and crops |
| Loop | On wraps to the start; off stops on the last image |
| Filename caption | Shows the file name in the corner |

## During the show

| Key | Action |
|---|---|
| Space | Pause or resume |
| → and ← | Next and previous (or click the right or left half of the screen) |
| F | Toggle fullscreen |
| H | Show or hide the position bar |
| Esc | Back to the thumbnails |

The position bar appears when you move the mouse or press a key, and fades after two seconds along with the cursor.

## Limits

- **HEIC photos from an iPhone** display in Safari but not in Chrome. Chrome outlines them in red and the show skips them.
- **The images themselves have to be added again after a reload.** A web page can't reopen your files without you handing them over again. Chrome's File System Access API could do it with your permission; that isn't built.
- **Very large batches.** Thumbnails are the original images scaled by the browser, so memory grows with how many are on screen. It has been tried with a handful of images, the largest a 52 MB PNG, not with hundreds.

## How it was built

The code was written by qwen3.8:27b running locally through Ollama, in three rounds: a build from a written spec, a repair against 16 defects found by reading the first version, and the arrangement-keeping features. Claude (Opus 5.5) reviewed and tested each round in Chrome, fixed what was left, and wrote the restore bar's HTML and CSS. `dev/` holds the three briefs and qwen's versions as returned, so `diff dev/qwen-v3.html slideshow.html` shows what changed after the last qwen round. `DECISIONS.md` lists the fixes. Source: [github.com/BrettRey/slideshow](https://github.com/BrettRey/slideshow).

Testing was in Chrome with simulated input, because the browser automation couldn't send real keystrokes or mouse drags. Safari, real drag-and-drop from Finder and real fullscreen haven't been tried yet.
