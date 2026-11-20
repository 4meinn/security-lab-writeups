# Reflected XSS into HTML Context with Nothing Encoded

- **Lab:** Reflected XSS into HTML context with nothing encoded
- **Platform:** PortSwigger Web Security Academy
- **Category:** Cross-Site Scripting (OWASP A03:2021, Injection), reflected, HTML context
- **Difficulty:** Apprentice
- **Tool:** Web browser (DevTools)
- **URL:** https://portswigger.net/web-security/cross-site-scripting/reflected/lab-html-context-nothing-encoded

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

The blog's search feature reflects the search term straight back into the page. The value is placed into the HTML response with no output encoding, so any markup in it is parsed as markup rather than shown as text. This is a reflected cross-site scripting flaw: input from the request is echoed into the immediate response and run by the browser. The goal is to exploit it to call the `alert` function.

## Reconnaissance

The search box sends the term as a query parameter:

```
GET /?search=metinberkaysen
```

The response drops it straight into the page heading:

```html
<h1>0 search results for 'metinberkaysen'</h1>
```

Inspected in DevTools, the term comes back verbatim, not entity-encoded: the characters I sent are the characters in the HTML. The value lands in a plain HTML context, between tags, with nothing to stop the browser reading `<` and `>` as the start of an element. A `<script>` dropped here is therefore parsed and executed, not displayed.

## Exploitation

### Step 1: Confirm raw reflection

I searched a harmless unique string and inspected the response. It sat inside the `<h1>` exactly as typed, with no encoding, which confirms the output is not sanitised on the way back.

### Step 2: Inject a script element

I submitted the payload in the search box:

```html
<script>alert(1)</script>
```

which reaches the server as:

```
GET /?search=<script>alert(1)</script>
```

### Step 3: Script executes

The server reflected the payload into the heading unencoded, so the browser parsed it as a real `<script>` element and ran it. The `alert` dialog fired, confirming arbitrary JavaScript execution in the page. The lab registered as solved.

## Impact

Any value that reaches this endpoint runs as code in the visitor's browser, in the security context of the vulnerable site. Because the flaw is reflected, an attacker weaponises it with a crafted link: the payload sits in the `search` parameter, so any victim who opens the link executes it on the target origin. From there the script can act as that user, reading session cookies that are not `HttpOnly`, sending authenticated requests, lifting data rendered on the page, or defacing it. Nothing has to be stored first; the single request carries the attack, so a convincing link is the only requirement.

## Remediation

- **Encode on output, for the context.** Reflecting user input into HTML means HTML-encoding it first, so `<`, `>`, `&`, `"` and `'` become entities and can never open a tag. This is the root-cause fix.
- **Validate input where the format is known,** as defence in depth, but encoding is what stops execution, not validation.
- **Set a Content-Security-Policy** that blocks inline script, so a reflected payload has no easy path to run even if one slips through.
- **Mark session cookies `HttpOnly`,** so a successful XSS cannot read them from script.

## Key takeaway

Reflected XSS is the server echoing request input into its response without encoding it, so the browser runs what should have been text. Here the search term was dropped straight into an `<h1>` with nothing encoded, so `<script>alert(1)</script>` in the query string executed on the page. The fix is context-aware output encoding at the point of reflection; everything else is secondary hardening.
