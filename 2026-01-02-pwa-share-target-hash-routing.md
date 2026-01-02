# PWA Web Share Target with Hash Routing

*Learned: 2026-01-02*

## Context

Implementing Android share sheet functionality in a PWA that uses HashRouter, allowing users to share text/links from any app to IntexuraOS.

## Insight

### 1. The Share Target API

The Share Target API enables PWAs to receive content from the native share sheet, just like native applications. You declare this capability in your web app manifest (`manifest.json`) by specifying the action endpoint, HTTP method, and expected parameters.

Example manifest declaration:

```json
{
  "share_target": {
    "action": "/#/share-target",
    "method": "GET",
    "params": {
      "title": "title",
      "text": "text",
      "url": "url"
    }
  }
}
```

When a user shares from another app, the OS invokes your PWA with these parameters populated in the URL.

### 2. Hash Routing Constraint

Single-page applications using HashRouter face a critical constraint: the share target action is processed by the web server *before* JavaScript runs. If your server doesn't support SPA fallback (like Google Cloud Storage bucket backends), the server won't know how to handle the route.

**Solution:** Use the hash route directly in the action path: `"/#/share-target"` instead of `"/share-target"`.

This works because:
- The share URL becomes `/#/share-target?text=hello&url=https://example.com`
- The server receives a request to `/` (with the hash as client-side routing)
- The server serves `index.html` to `/`
- JavaScript loads, the hash router intercepts `/#/share-target`
- The share target component receives the parameters

Without the hash, a static hosting backend returns 404 for `/share-target` since it's not a real file.

### 3. GET vs POST Trade-off

Two approaches exist for passing shared content:

**GET (URL Parameters):**
- Pros: Works with static hosting, simple, no server-side handling needed
- Cons: Limited by URL length (~2000 chars), visible in browser history
- Best for: Text and link sharing

**POST (multipart/form-data):**
- Pros: Supports file sharing, larger data capacity
- Cons: Requires server-side endpoint to handle form parsing, incompatible with static hosting
- Best for: File sharing and large content

For text/link sharing in a static-hosted PWA, GET is the simpler choice.

### 4. Installation Requirement

The Share Target API only works when the PWA is **installed to the home screen**. Users must:
1. Open your PWA in the browser
2. Install it (via browser menu → "Install app" or similar)
3. Launch from home screen
4. Then the app appears in native share sheets

Share target will not appear in the share sheet for browser tabs or uninstalled PWAs. This is a security measure to prevent every web page from appearing as a share target.

## Implementation Checklist

- [ ] Add `share_target` to `manifest.json`
- [ ] Use hash route in action: `"/#/share-target"`
- [ ] Create share-target route component that reads URL params
- [ ] Handle text, url, and title query parameters
- [ ] Test on Android device with installed PWA
- [ ] Verify share sheet shows your app
- [ ] Test sharing from various applications (Chrome, messaging apps, etc.)

## Sources

- Learned from implementation context
