# Reflected XSS into a Template Literal with Angle Brackets, Single, Double Quotes, Backslash and Backticks Unicode-Escaped

- **Lab:** Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped
- **Platform:** PortSwigger Web Security Academy
- **Category:** Cross-Site Scripting (OWASP A03:2021, Injection), reflected, JavaScript template literal context
- **Difficulty:** Practitioner
- **Tool:** Web browser (DevTools)
- **URL:** https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-template-literal-angle-brackets-single-double-quotes-backslash-backticks-escaped

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

The search term is reflected inside a JavaScript template literal, a backtick-delimited string. Every character that would normally let me out is locked: angle brackets, single quotes and double quotes are HTML-encoded, and backticks and backslashes are Unicode-escaped. So I can neither inject a tag nor close the string. But a template literal has a feature a normal string does not, expression interpolation with `${...}`, and those characters are not filtered. This is reflected cross-site scripting in a template literal context. The goal is to run `alert` from inside the template literal.

## Reconnaissance

The term is reflected into a template literal:

```html
<script>
    var message = `0 search results for 'metinBerkaySen'`;
    document.getElementById('searchMessage').innerText = message;
</script>
```

The value sits between backticks. Quotes come back HTML-encoded and the backtick is escaped, so the usual breakouts are closed. The point is that I do not need to leave the string at all: inside a template literal, `${expression}` is evaluated as JavaScript when the literal is built, and the `$`, `{` and `}` characters pass through untouched.

## Exploitation

### Step 1: Inject a template expression

I searched:

```
${alert(1)}
```

The literal became:

```js
var message = `0 search results for '${alert(1)}'`;
```

### Step 2: Interpolation runs the code

When the template literal was constructed, the `${alert(1)}` expression was evaluated, so `alert(1)` ran and the lab registered as solved. The result message read "0 search results for 'undefined'", because `alert` returns `undefined` and that value was interpolated into the string. No quote, backtick or tag was needed.

## Impact

A template literal evaluates any `${...}` it contains, so reflecting untrusted input into one is as dangerous as reflecting into any other script context, even with quotes and backticks fully locked down. An attacker crafts a link with `${...}` in the search parameter, and a victim who opens it runs the attacker's code on page load, in their session. Escaping the string delimiters gives a false sense of safety here: the interpolation syntax is a second way into the same code context, and it was left open.

## Remediation

- **Do not place untrusted input inside a template literal.** The `${...}` syntax makes it an execution context, not just a string; pass data through an HTML data attribute or JSON the script reads instead. This is the root-cause fix.
- **If a value must go in, encode it for the template literal context,** which means neutralising `${` as well as the backtick and backslash, not only the quotes.
- **Validate or allowlist** where the format is known, as defence in depth.
- **Set a Content-Security-Policy** that forbids inline script, which removes the sink.

## Key takeaway

A template literal is not just a string, it is an expression context: anything in `${...}` runs. Here angle brackets, quotes, backticks and backslashes were all escaped, so no classic breakout worked, but `${alert(1)}` needed no breakout, because interpolation evaluates inside the literal. Encoding the delimiters is not enough when the interpolation syntax itself is open; the fix has to treat `${` as dangerous as the backtick.
