# Pod Tracer

Track the podcast episodes you've listened to. Not a player: it keeps your place in each show.

- Look up any podcast by name and track it; every episode is loaded from the show's feed
- **Up Next**: each show's next episode, with a ✓ to mark it listened; the square fills as you get through the show
- Guests are pulled out of titles and shown under them, e.g. *(with JANE DOE)*
- New episodes are checked for automatically (on open, at most every 3 hours). A show you're caught up on jumps back to the top of Up Next; a show you're partway through just updates its count
- Each show's page: every episode, filter by played/unplayed, mark one or mark everything up to it
- Tag episodes as Main, Bonus or Promo (feeds' own bonus/trailer labels are picked up automatically); a show with tags gets tabs for each
- **Edit** in a show's window: change an episode's title, release date or tag; edits survive feed refreshes and go into backups
- Colors come from each podcast's name, the same way Stack Tracker colors wishlist books
- Backup, Export and Import (Data tab)
- Starts empty: tap **Add New Podcast** to find your first show

Everything is stored in your browser on this device. Use **Data → Backup** to save a copy, and import it on another device or browser.

**Backups on a phone:** in **Data → Backup**, tap **Backup location** and pick a folder once. Backups then save straight into it (the newest 10 are kept), Daily/Weekly backups happen by themselves, and the newest three show on **Import** to tap and restore. Without a folder, backups go to the share sheet (Save to Files / Drive) or a download.

## Online and offline

The app itself works offline: after the first visit, the service worker (`sw.js`) keeps a copy of every file, so it opens with no connection, and your library and Up Next are all there.
Searching for podcasts and checking for new episodes need a connection. Searches use Apple's podcast directory; episode lists come from each show's own feed (through a public relay if the feed blocks browsers, or Apple's list of the latest 200 as a last resort).

It can also be installed to your home screen (browser menu → **Install app** / **Add to Home Screen**).

## Full episode lists: your own feed relay (optional, free)

Most podcast feeds don't let a web page read them, so Pod Tracer fetches them through public relays. If a host blocks those relays, the show falls back to Apple's list (the newest 200 episodes) and appears in **Data → Feeds** with the reason.
A relay of your own fixes that for good. A free Cloudflare Worker takes about five minutes:

1. Sign up at **dash.cloudflare.com** (free), then go to **Workers & Pages → Create → Create Worker**, name it (e.g. `pod-feeds`) and **Deploy**.
2. **Edit code**, replace everything with the code below, and **Deploy** again.
3. In Pod Tracer, open **Data → Feeds**, enter `https://pod-feeds.<your-subdomain>.workers.dev/?url=` and tap **Save Relay**. It tests itself on one of your feeds.
4. Tap **Retry All**.

```js
// Pod Tracer feed relay: fetches a podcast feed and hands it back with permission for the app to read it
export default {
  async fetch(request) {
    const target = new URL(request.url).searchParams.get('url');
    if (!target || !/^https?:\/\//i.test(target)) return new Response('Add ?url=<feed address>', { status: 400 });
    const r = await fetch(target, { headers: { 'User-Agent': 'PodTracer/1.0 (+podcast feed reader)' }, redirect: 'follow' });
    return new Response(r.body, {
      status: r.status,
      headers: { 'Access-Control-Allow-Origin': '*', 'Content-Type': r.headers.get('Content-Type') || 'application/xml; charset=utf-8' }
    });
  }
};
```

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
