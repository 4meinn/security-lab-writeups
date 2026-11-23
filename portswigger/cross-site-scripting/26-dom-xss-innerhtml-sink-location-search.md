# DOM XSS in innerHTML Sink Using Source location.search

- **Lab:** DOM XSS in innerHTML sink using source location.search
- **Platform:** PortSwigger Web Security Academy
- **Category:** Cross-Site Scripting (OWASP A03:2021, Injection), DOM-based, innerHTML sink
- **Difficulty:** Apprentice
- **Tool:** Web browser (DevTools)
- **URL:** https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-innerhtml-sink

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

The blog search runs in client-side JavaScript. It reads the search term from the page URL and writes it into a div through an `innerHTML` assignment, with no encoding or validation. This is DOM-based cross-site scripting: the flaw is in the browser code, not in anything the server reflects. The source is `location.search` and the sink is the `innerHTML` assignment. The goal is to call the `alert` function.

## Reconnaissance

The search result message is populated by this script:

```js
function doSearchQuery(query) {
    document.getElementById('searchMessage').innerHTML = query;
}
var query = (new URLSearchParams(window.location.search)).get('search');
if(query) {
    doSearchQuery(query);
}
```

The term from `location.search` is assigned straight to the div's `innerHTML`, so any markup in it is parsed into the DOM. One thing does not carry over from the `document.write` case: markup set through `innerHTML` is parsed, but a `<script>` element inserted this way does not execute, by HTML specification. So the payload has to run through an element that fires an event by itself, not through a script tag.

## Exploitation

### Step 1: Pick an element that fires without a script tag

Since `innerHTML` will not run an injected `<script>`, I used an image with a broken source and an error handler:

```
<img src=1 onerror=alert(1)>
```

The `src` is invalid, so the image fails to load, and that failure fires `onerror`.

### Step 2: Script executes

Assigned into the div's `innerHTML`, the `<img>` was parsed, its load failed, and the `onerror` handler ran `alert`. The dialog fired and the lab registered as solved. The broken image icon left in the search message marks where the element landed.

## Impact

The dangerous data flows through the page's own JavaScript, so any output encoding the server applies never touches it: the client reads the URL and writes to the DOM itself. An attacker exploits it with a crafted link carrying the payload in the `search` parameter, and a victim who opens the link runs the script in their session on the site. From there it can act as that user, read non-`HttpOnly` cookies, or lift data shown on the page. The script tag being inert through `innerHTML` is no protection: an event handler on a broken image runs code just the same.

## Remediation

- **Do not assign untrusted data to `innerHTML`.** Use `textContent`, which inserts the value as text without parsing markup, so no tag or handler can form. This is the root-cause fix.
- **If HTML must be built from the value, sanitise it first** with a vetted library such as DOMPurify, or encode it for the HTML context.
- **Validate or allowlist the value** where its format is known, as defence in depth.
- **Set a Content-Security-Policy** that blocks inline event handlers, so a payload that slips through has no easy path to run.

## Key takeaway

An `innerHTML` sink parses whatever markup it is given, so DOM XSS here needs no script tag, which would not run through `innerHTML` anyway: an element that fires its own event, like an image with a broken `src` and an `onerror` handler, runs the code instead. The source was `location.search`, and the fix is in the client: write user data with `textContent`, or sanitise before assigning to `innerHTML`.
