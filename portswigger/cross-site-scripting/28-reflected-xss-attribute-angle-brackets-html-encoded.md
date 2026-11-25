# Reflected XSS into Attribute with Angle Brackets HTML-Encoded

- **Lab:** Reflected XSS into attribute with angle brackets HTML-encoded
- **Platform:** PortSwigger Web Security Academy
- **Category:** Cross-Site Scripting (OWASP A03:2021, Injection), reflected, HTML attribute context
- **Difficulty:** Practitioner
- **Tool:** Web browser
- **URL:** https://portswigger.net/web-security/cross-site-scripting/contexts/lab-attribute-angle-brackets-html-encoded

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

The blog search reflects the search term back into the page, but inside an HTML attribute value rather than the element body. Angle brackets are HTML-encoded on the way out, so a new tag cannot be injected. The quotation mark that delimits the attribute is not encoded, and that is enough: breaking out of the attribute and adding an event handler runs script without any tag. This is reflected cross-site scripting in an attribute context. The goal is to call the `alert` function.

## Reconnaissance

Searching a test term, it comes back inside an attribute value, in the shape of:

```html
<input ... value="TERM">
```

In the response, `<` and `>` are returned as `&lt;` and `&gt;`, so I cannot open a new tag: `<script>` or `<svg>` would only be printed as text. But the double quote that closes the `value` is reflected as a literal `"`. That lets me end the attribute early and append attributes of my own to the existing element, including an event handler, with no tag needed.

## Exploitation

### Step 1: Break out of the attribute

I searched:

```
"onmouseover="alert(1)
```

The leading `"` closes the `value` attribute, and `onmouseover="alert(1)` adds an event handler to the same element. Reflected, the element became:

```html
<input ... value=""onmouseover="alert(1)">
```

### Step 2: Fire the handler

The element now carried an `onmouseover` handler. Moving the mouse over it fired `alert(1)`, confirming script execution, and the lab registered as solved. The payload used no angle brackets at all, so the HTML-encoding of `<` and `>` never came into play.

## Impact

Encoding angle brackets stops tag injection but leaves attribute-context injection open when the quote is not encoded as well. An attacker crafts a link with the payload in the search parameter, and a victim who opens it and moves the mouse over the element runs the attacker's script in their session. The interaction is slight, mouse movement over a reflected element, and handlers such as `onmouseover`, or `autofocus` with `onfocus` for a variant that needs no interaction at all, make it reliable. From there the script acts as the user: session theft, actions on their behalf, or exposure of data on the page.

## Remediation

- **Encode for the attribute context, not just the body.** HTML-encode the double quote (and single quote) as well as the angle brackets, so the value cannot be closed early. This is the root-cause fix.
- **Always quote attribute values and encode the matching quote character,** so breaking out is impossible.
- **Validate or allowlist the input** where its format is known, as defence in depth.
- **Set a Content-Security-Policy** that blocks inline event handlers, so a stray handler has no effect.

## Key takeaway

Encoding angle brackets only blocks new tags; it does nothing for an injection that stays inside an attribute. Here the search term landed in a quoted value with the quote left unencoded, so `"onmouseover="alert(1)` closed the attribute and added a handler, no tag required. Output encoding has to match the context: in an attribute, the delimiting quote must be encoded too.
