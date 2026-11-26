# Reflected XSS into a JavaScript String with Angle Brackets HTML Encoded

- **Lab:** Reflected XSS into a JavaScript string with angle brackets HTML encoded
- **Platform:** PortSwigger Web Security Academy
- **Category:** Cross-Site Scripting (OWASP A03:2021, Injection), reflected, JavaScript string context
- **Difficulty:** Practitioner
- **Tool:** Web browser (DevTools)
- **URL:** https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-angle-brackets-html-encoded

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

The search query tracking reflects the search term into the page inside a JavaScript string, within a `<script>` block. Angle brackets are HTML-encoded, so no new tag can be injected. The single quote that delimits the string is not encoded in that context, so breaking out of the string and writing my own JavaScript runs code with no tag at all. This is reflected cross-site scripting in a JavaScript string context. The goal is to break out of the string and call `alert`.

## Reconnaissance

Searching a test term, it comes back inside a script, assigned to a variable:

```html
<script>
    var searchTerms = 'aberMetin';
    document.write('<img src="/resources/images/tracker.gif?searchTerms='+searchTerms+'">');
</script>
```

In the HTML body the term's quotes are returned encoded (as `&apos;`) and angle brackets come back as `&lt;` / `&gt;`, so a tag is out. But inside the `<script>`, the single quote that wraps the value is a literal `'`. That means I can close the string, add my own expression, and reopen a string so the statement still parses.

## Exploitation

### Step 1: Break out of the string

I searched:

```
'-alert(1)-'
```

The assignment became:

```js
var searchTerms = ''-alert(1)-'';
```

The first `'` closes the empty string, `-alert(1)-` evaluates `alert(1)` as part of an arithmetic expression, and the trailing `'` opens a new empty string so the line stays valid JavaScript.

### Step 2: Script executes

The script ran on page load, evaluated the expression, and `alert(1)` fired. The lab registered as solved. No angle brackets were used, so their encoding was irrelevant.

## Impact

The term is placed inside live script with only its angle brackets encoded, which does nothing for a payload that stays inside the JavaScript. An attacker crafts a link with the payload in the search parameter, and any victim who opens it runs the attacker's code on page load, in their session on the site, with no interaction at all. From there it can act as the user, read non-`HttpOnly` cookies, or take data from the page. Reflecting untrusted input straight into a script is among the most dangerous sinks, since the attacker is writing code into a context that is already code.

## Remediation

- **Do not reflect untrusted input into a script.** Pass data to the client through an HTML data attribute or a JSON block the script reads, rather than building script text from the input. This is the root-cause fix.
- **If a value must go into JavaScript, encode it for the JavaScript string context** (escape the quote, backslash and line terminators), not just for HTML. Encoding angle brackets does nothing inside a string.
- **Validate or allowlist** where the format is known, as defence in depth.
- **Set a Content-Security-Policy** that forbids inline script, which removes this class of sink.

## Key takeaway

Encoding angle brackets guards the HTML context, not the JavaScript one. Here the search term sat inside a quoted JavaScript string with the quote left raw, so `'-alert(1)-'` closed the string and ran code, no tag involved. Output encoding has to match the context the data lands in: inside a script string, the string delimiter and escape characters are what must be encoded.
