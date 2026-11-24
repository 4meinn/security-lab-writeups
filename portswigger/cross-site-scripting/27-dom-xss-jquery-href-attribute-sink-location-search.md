# DOM XSS in jQuery Anchor href Attribute Sink Using location.search Source

- **Lab:** DOM XSS in jQuery anchor href attribute sink using location.search source
- **Platform:** PortSwigger Web Security Academy
- **Category:** Cross-Site Scripting (OWASP A03:2021, Injection), DOM-based, jQuery href attribute sink
- **Difficulty:** Apprentice
- **Tool:** Web browser (DevTools)
- **URL:** https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-href-attribute-sink

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

The submit feedback page has a "back" link whose destination is read from the page URL. A jQuery one-liner takes the `returnPath` query parameter and assigns it as the link's `href`, with no check on the scheme. This is DOM-based cross-site scripting, and the sink is an `href` attribute rather than an HTML-parsing sink. The source is `location.search`. The goal is to make the back link call `alert(document.cookie)`.

## Reconnaissance

The back link's `href` is set on page load:

```js
$(function() {
    $('#backLink').attr("href", (new URLSearchParams(window.location.search)).get('returnPath'));
});
```

The `returnPath` parameter flows straight into the anchor's `href` through jQuery's `attr()`. An `href` does not parse HTML, so injecting a tag is pointless here, but it does act on the URL scheme: a `javascript:` URL in an `href` runs its code when the link is followed. The parameter is attacker-controlled and the scheme is never validated, so I can point the link at a `javascript:` URL of my choice.

## Exploitation

### Step 1: Set the href to a javascript: URL

I loaded the feedback page with `returnPath` pointing at a `javascript:` URL:

```
/feedback?returnPath=javascript:alert(document.cookie)
```

jQuery assigned that value to the back link, so in the DOM the anchor became:

```html
<a id="backLink" href="javascript:alert(document.cookie)">Back</a>
```

### Step 2: Trigger on click

Clicking the "Back" link followed the `javascript:` URL, which executed `alert(document.cookie)`. The dialog showed the page cookies and the lab registered as solved. Unlike the HTML-sink labs, this one fires on a user action, not on load.

## Impact

The data flows through the page's own JavaScript into a link's `href`, so server-side encoding never touches it, and an `href` will execute a `javascript:` scheme URL. An attacker sends a crafted link with `returnPath` set to a `javascript:` payload, and a victim who opens it and clicks the back link runs the attacker's code in their session. `alert(document.cookie)` here stands in for the real aim: reading the session cookie or acting as the user. The interaction required is minor, a single expected click on a normal-looking navigation link.

## Remediation

- **Validate the scheme before using untrusted input as a URL.** Allow only safe schemes (`http`, `https`, or a relative path) and reject `javascript:`, `data:` and the like. This is the root-cause fix.
- **Prefer a strict allowlist of permitted return paths** over echoing a caller-supplied URL into a link.
- **Encode and validate in the client code,** since the sink is reached there, not on the server.
- **Set a Content-Security-Policy** without inline script, as defence in depth, to shrink what a bypass can do.

## Key takeaway

Not every XSS sink parses HTML. An `href` is a URL sink: it will not run an injected tag, but it will execute a `javascript:` scheme URL when the link is followed. Here jQuery copied the `returnPath` parameter straight into the back link's `href`, so `javascript:alert(document.cookie)` ran on click. The fix is to validate the scheme and allow only safe, ideally allowlisted, destinations.
