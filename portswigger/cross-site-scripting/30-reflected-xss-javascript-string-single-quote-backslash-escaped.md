# Reflected XSS into a JavaScript String with Single Quote and Backslash Escaped

- **Lab:** Reflected XSS into a JavaScript string with single quote and backslash escaped
- **Platform:** PortSwigger Web Security Academy
- **Category:** Cross-Site Scripting (OWASP A03:2021, Injection), reflected, JavaScript string context
- **Difficulty:** Practitioner
- **Tool:** Web browser (DevTools)
- **URL:** https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-single-quote-backslash-escaped

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

The search query tracking reflects the search term inside a JavaScript string in a `<script>` block, the same shape as the previous lab. This time the string is hardened: a single quote in the input is escaped to `\'`, and a backslash is escaped to `\\`, so the string cannot be closed with a quote and the escaping cannot be cancelled. Angle brackets, however, are not encoded. This is reflected cross-site scripting in a JavaScript string context where the string is locked but the surrounding HTML is not. The goal is to break out and call `alert`.

## Reconnaissance

The search term is reflected into a script variable:

```html
<script>
    var searchTerms = 'testMetin';
    document.write('<img src="/resources/images/tracker.gif?searchTerms='+searchTerms+'">');
</script>
```

Sending a single quote confirmed the hardening: `test'` came back as `'test\''`, the quote escaped, so I cannot end the string that way, and a backslash is doubled, so I cannot cancel the escape either. But `<` and `>` were reflected literally, not as `&lt;` / `&gt;`. That reopens the attack at a different layer: if I cannot leave the string, I can leave the whole script element.

## Exploitation

### Step 1: Close the script element

The HTML parser finds the end of a `<script>` by scanning its raw text for the literal `</script>` sequence, with no regard for JavaScript string context. So a `</script>` inside the string still ends the element. I searched:

```
</script><script>alert(1)</script>
```

### Step 2: Inject a fresh script

Reflected, the block became:

```html
<script>
    var searchTerms = '</script><script>alert(1)</script>';
    ...
</script>
```

The parser closed the original script at my `</script>`, left the broken remainder as inert text, and parsed `<script>alert(1)</script>` as a new element. It ran on load, `alert(1)` fired, and the lab registered as solved. The string escaping never mattered, because I never tried to break the string.

## Impact

Reflecting untrusted input into a script is a high-severity sink: the attacker writes into a context that is already code. An attacker crafts a link with the payload in the search parameter, and any victim who opens it runs the injected script on page load, in their session, with no interaction. From there it can act as the user, read non-`HttpOnly` cookies, or exfiltrate data on the page. The instructive point is that the string was escaped and the attack still worked: hardening one layer, the JavaScript string, left another, the HTML element, wide open.

## Remediation

- **Encode for every layer the data passes through.** Escaping the JavaScript string is correct but not enough while the value still reaches raw HTML; HTML-encode the angle brackets as well, so `</script>` can never appear.
- **Better, do not build script from input at all.** Pass data through an HTML data attribute or a JSON block the script reads.
- **Validate or allowlist** where the format is known, as defence in depth.
- **Set a Content-Security-Policy** that forbids inline script, which removes the sink.

## Key takeaway

Escaping the string delimiter locks one door and does nothing if another is open. Here the single quote and backslash were escaped, so the string could not be broken, but angle brackets were not encoded, so `</script>` ended the whole script element and a fresh `<script>` ran. The HTML parser closes a script on the literal `</script>` regardless of JavaScript context, so defence has to cover every layer the input touches, not just the innermost one.
