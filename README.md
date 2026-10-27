# Pod Tracer

Track the podcast episodes you've listened to. Not a player: it keeps your place in each show.

- Look up any podcast by name and track it; every episode is loaded from the show's feed
- **Up Next**: each show's next episode, with a ✓ to mark it listened; the square fills as you get through the show
- Guests are pulled out of titles and shown under them, e.g. *(with JANE DOE)*
- New episodes are checked for automatically (on open, at most every 3 hours). A show you're caught up on jumps back to the top of Up Next; a show you're partway through just updates its count
- Each show's page: every episode, filter by played/unplayed, mark one or mark everything up to it
- Colors come from each podcast's name, the same way Stack Tracker colors wishlist books
- Backup, Export and Import (Data tab)

Everything is stored in your browser on this device. Use **Data → Backup** to save a copy, and import it on another device or browser.

## Online and offline

The app itself works offline: after the first visit, the service worker (`sw.js`) keeps a copy of every file, so it opens with no connection, and your library and Up Next are all there.
Searching for podcasts and checking for new episodes need a connection. Searches use Apple's podcast directory; episode lists come from each show's own feed (through a public relay if the feed blocks browsers, or Apple's list of the latest 200 as a last resort).

It can also be installed to your home screen (browser menu → **Install app** / **Add to Home Screen**).

## Files

| File | What it is |
|---|---|
| `index.html` | the app |
| `sw.js` | service worker: offline copy of the app |
| `manifest.webmanifest` | name, colours and icons for installing |
| `icon-192.png`, `icon-512.png`, `apple-touch-icon.png` | app icons |
| `.nojekyll` | tells GitHub Pages to serve the files as they are |

## Host it on GitHub Pages

1. Create a repository (for example `PodTracer`) and upload all the files above, together in the root. `.nojekyll` starts with a dot, so it can be hidden on some computers. Upload it too.
2. In the repository, go to **Settings → Pages**.
3. Under **Build and deployment**, pick **Deploy from a branch**, choose `main` and `/ (root)`, and save.
4. After a minute the app is live at `https://<your-username>.github.io/<repository-name>/`.

## Updating

When you upload a new `index.html`, also change `VERSION` at the top of `sw.js` (for example `pod-tracer-v2`).
That tells installed copies to fetch the new files; they switch over the next time the app is opened.
