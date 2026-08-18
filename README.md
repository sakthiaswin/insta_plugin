# InstaGrabber Plugins

Drop `.json` files in this `plugins/` folder. In the app, go to **Plugins**, paste
this repo's URL (`https://github.com/<you>/<this-repo>`), tap **Detect plugins**,
then **Install** the ones you want. The app scans this folder via the GitHub
Contents API — no build step, no app update needed to ship a fix.

## Included plugins

- `instagram_post.json` — single photo, single video, and carousel posts (`/p/...`)
- `instagram_reel.json` — reels (`/reel/...`, `/reels/...`)

## Honest caveat

Instagram's unauthenticated `?__a=1&__d=dis` JSON endpoint used here is the same
technique tools like instaloader/gallery-dl rely on, and Instagram tightens or
breaks it periodically (rate limits, login walls, response shape changes). When
that happens the app will surface the plugin's error message instead of crashing
silently — that's your signal to update the plugin, not the app. Common fixes:

- Point `request.urlTemplate` at a different working endpoint (embed page,
  GraphQL doc_id query, etc.) and adjust `extract.*` paths to match its shape.
- Add/adjust `headers` (Instagram sometimes gates on `X-IG-App-ID` or cookies).
- If Instagram starts requiring a logged-in session for a given content type,
  that's outside what a stateless plugin can do without you adding session
  cookie support to the request headers.

## Plugin JSON schema

```jsonc
{
  "id": "unique_id",
  "name": "Human readable name",
  "version": "1.0.0",
  "targets": ["post", "reel", "carousel"],
  "urlPattern": "regex with a (?<shortcode>...) or first capture group",
  "request": {
    "method": "GET",
    "urlTemplate": "https://.../{shortcode}/...",
    "headers": { "X-IG-App-ID": "..." }
  },
  "extract": {
    "ownerPath": "dot.path.to.username",
    "captionPath": "dot.path.to.caption",
    "itemsPath": "dot.path.to.an.array",
    "itemsPathFallback": "dot.path.to.a.single.object",
    "itemUnwrapKey": "node",
    "itemType": "is_video",
    "itemUrl": "video_url",
    "itemImageUrl": "display_url",
    "itemThumbUrl": "display_url",
    "itemWidthPath": "dimensions.width",
    "itemHeightPath": "dimensions.height"
  }
}
```

No compiled code is ever downloaded or executed — plugins are pure data, parsed
and interpreted by the app's `PluginEngine`. That's what makes installing a
plugin from a repo safe.
