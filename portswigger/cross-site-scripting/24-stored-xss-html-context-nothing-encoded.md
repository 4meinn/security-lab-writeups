# Stored XSS into HTML Context with Nothing Encoded

- **Lab:** Stored XSS into HTML context with nothing encoded
- **Platform:** PortSwigger Web Security Academy
- **Category:** Cross-Site Scripting (OWASP A03:2021, Injection), stored, HTML context
- **Difficulty:** Apprentice
- **Tool:** Web browser (DevTools)
- **URL:** https://portswigger.net/web-security/cross-site-scripting/stored/lab-html-context-nothing-encoded

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

Each blog post accepts comments, and a posted comment is saved and then shown on the post to everyone who opens it. The comment body is written into the page with no output encoding, so markup in it is parsed rather than displayed. This is a stored cross-site scripting flaw: the payload is persisted server-side and runs in the browser of every later visitor. The goal is to post a comment that calls the `alert` function when the post is viewed.

## Reconnaissance

The comment form posts to a single endpoint, carrying the comment alongside the post id and an anti-CSRF token:

```html
<form action="/post/comment" method="POST">
  <textarea name="comment"></textarea>
  <input name="name">
  <input name="email">
  <input name="website">
  <input type="hidden" name="postId" value="1">
  <input type="hidden" name="csrf" value="...">
</form>
```

The comment body is the field of interest. It is stored against the post and rendered back on the post page. If that render carries no encoding, a `<script>` in the comment becomes a live element for every reader, not only in a single reflected response to the author.

## Exploitation

### Step 1: Submit a script as the comment

In the comment field I entered the payload and filled the other required fields with arbitrary values:

```
POST /post/comment

csrf=<token>&postId=1&comment=<script>alert(1)</script>&name=metin&email=test@gmail.com&website=https://metinberkaysen.com
```

The server accepted the comment and confirmed it was submitted.

### Step 2: Payload runs on view

Opening the blog post, the stored comment was written into the page unencoded, so the browser parsed the `<script>` element and executed it. The `alert` dialog fired on load and the lab registered as solved. Unlike the reflected case, nothing had to be carried in my request to trigger it: the script now lives on the post and runs for anyone who reads it.

## Impact

A stored payload executes for every visitor to the post, with no crafted link and no interaction beyond viewing the page, which makes it markedly more dangerous than reflected XSS. It runs in each victim's session on the site, so a single comment can read non-`HttpOnly` cookies, act as each user who loads the page, harvest data shown to them, or reach an administrator who opens the post while moderating. Because the attack is persisted, it keeps working until the comment is found and removed, and it reaches users the attacker never contacted.

## Remediation

- **Encode on output, for the context.** When the stored comment is written into HTML, HTML-encode it so `<`, `>`, `&`, `"` and `'` become entities and cannot open a tag. Encoding at output, not at storage, is the root-cause fix, since the same data may later be used in a different context.
- **Validate input where the format is known,** as defence in depth, but it does not replace output encoding.
- **Set a Content-Security-Policy** that blocks inline script, so a stored payload has no straightforward path to run.
- **Mark session cookies `HttpOnly`,** so a successful XSS cannot read them from script.

## Key takeaway

Stored XSS is the same encoding failure as reflected XSS, but the payload is saved on the server and served to everyone who views the page, so it needs no crafted link and runs again on every visit. Here a blog comment holding `<script>alert(1)</script>` was stored and rendered unencoded, so it executed for anyone opening the post. The fix is context-aware output encoding wherever stored data is written into the page.
