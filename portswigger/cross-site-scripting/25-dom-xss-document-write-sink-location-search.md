# DOM XSS in document.write Sink Using Source location.search

- **Lab:** DOM XSS in document.write sink using source location.search
- **Platform:** PortSwigger Web Security Academy
- **Category:** Cross-Site Scripting (OWASP A03:2021, Injection), DOM-based, document.write sink
- **Difficulty:** Apprentice
- **Tool:** Web browser (DevTools)
- **URL:** https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-document-write-sink

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

The blog's search tracking runs entirely in client-side JavaScript. It reads the search term from the page URL and writes it into the page with `document.write`, with no encoding or validation. This is DOM-based cross-site scripting: the flaw is in the browser code, not in anything the server reflects. The source is `location.search` and the sink is `document.write`. The goal is to call the `alert` function.

## Reconnaissance

The page tracks searches with this script:

```js
function trackSearch(query) {
    document.write('<img src="/resources/images/tracker.gif?searchTerms='+query+'">');
}
var query = (new URLSearchParams(window.location.search)).get('search');
if(query) {
    trackSearch(query);
}
```

The term from `location.search` is concatenated straight into an `<img>` tag and handed to `document.write`. Nothing encodes it, and it lands inside the `src` attribute value. Breaking out of that attribute and the tag lets me inject my own markup, which `document.write` commits to the page as real HTML.

## Exploitation

### Step 1: Break out of the attribute

I submitted, as the search term:

```
"><svg onload=alert(1)>
```

The `">` closes the `src` value and the `<img>` tag, and `<svg onload=alert(1)>` is then a fresh element.

### Step 2: Script executes

`document.write` emitted:

```html
<img src="/resources/images/tracker.gif?searchTerms="><svg onload=alert(1)>">
```

The browser parsed the injected `<svg>`, its `onload` handler ran, and the `alert` dialog fired. The lab registered as solved.

## Impact

The dangerous data flows through the page's own JavaScript, so any output encoding the server applies never touches it: the client reads the URL and writes to the DOM itself. An attacker exploits it with a crafted link carrying the payload in the `search` parameter, and a victim who opens the link runs the script in their session on the site. From there it can act as that user, read non-`HttpOnly` cookies, or lift data shown on the page. Because the sink parses and commits arbitrary markup, a single query parameter is enough to run code.

## Remediation

- **Avoid `document.write` with untrusted input.** Build the DOM with APIs that do not parse markup, such as `textContent` or `createElement`, so injected tags cannot form.
- **Encode for the HTML context in the client code** if data must be written into markup, so `<` and `"` cannot break out.
- **Validate or allowlist the value** where its format is known, as defence in depth.
- **Set a Content-Security-Policy** that blocks inline event handlers and script, so a payload that slips through has no easy path to run.

## Key takeaway

DOM-based XSS lives in the client's own JavaScript: a source the attacker controls, `location.search`, flows into a sink that parses markup, `document.write`, with no encoding in between. Here the search term was written into an `<img>` tag, so `"><svg onload=alert(1)>` broke out and ran. The fix is in the browser code, not the server: avoid markup-parsing sinks and encode before writing.
