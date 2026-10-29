# User ID Controlled by Request Parameter, with Unpredictable User IDs (IDOR)

- **Lab:** User ID controlled by request parameter, with unpredictable user IDs
- **Platform:** PortSwigger Web Security Academy
- **Category:** Access Control (OWASP A01:2021, Broken Access Control), IDOR / horizontal privilege escalation
- **Difficulty:** Apprentice
- **Tool:** Web browser only
- **URL:** https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-unpredictable-user-ids

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

This is the same IDOR as the previous lab: the account page loads a user's data from an `id` in the request, with no check that the record belongs to the caller. The difference is that the identifier is no longer a username like `carlos`, it is an unpredictable GUID. The defence being relied on is that an attacker cannot guess that value. The goal is to find carlos's GUID somewhere in the application, use it to read his account, and obtain his API key.

## Reconnaissance

I logged in with `wiener:peter`. My own account page was at:

```
/my-account?id=<my-guid>
```

The `id` is now a random GUID, not the username, so substituting `carlos` as in the previous lab does not work. The identifier has to be discovered. The application leaks it: in the Blog section, carlos is the author of a post ("Finding Inspiration"), and the author's name is a link that carries his user ID. Inspecting that link (page source / Elements panel) showed:

```html
<span id="blog-author">
  <a href="/blogs?userId=53c3bf0c-c074-448b-a55c-b96252a136c5">carlos</a>
</span>
```

carlos's GUID, `53c3bf0c-c074-448b-a55c-b96252a136c5`, is unpredictable but not secret. The application publishes it in the author link.

## Exploitation

### Step 1: Recover carlos's GUID

From the blog post author link above, carlos's user ID is `53c3bf0c-c074-448b-a55c-b96252a136c5`.

### Step 2: Use the GUID on the account page

I set the account page `id` to carlos's GUID:

```
/my-account?id=53c3bf0c-c074-448b-a55c-b96252a136c5
```

The server returned carlos's account page, including his API key, with no check that my session belonged to carlos.

```
Your username is: carlos
Your API Key is: xNrfkNK10FFEvVX6VuLgnmGy6svodkfA
```

### Step 3: Solve

Submitting carlos's API key solved the lab. The key is from a disposable lab instance and is already dead.

## Impact

This is identical to the predictable-ID IDOR. The only added protection was that the identifier was a random GUID rather than a username, and the application defeated that itself by publishing the GUID in a blog author link. Any authenticated user can still read any other user's data and credentials. An unpredictable ID slows down blind enumeration, but it does nothing once the value is exposed anywhere, and applications routinely expose user IDs in author links, profile pages, URLs, and API responses.

## Remediation

- **Enforce an ownership check on every record access,** server-side, derived from the session. This is the same fix as the predictable-ID case, and it is the only one that actually closes the hole.
- **Do not treat an unguessable ID as access control.** It is defence in depth at best. Here the same application that used the GUID also handed it out in an author link; GUIDs also leak through referrers, logs, browser history, and shared URLs.
- **Minimise exposure of internal identifiers** where they are not needed, as hardening, not as the control.
- **Treat the exposed API key as compromised and rotate it.**

## Key takeaway

Swapping a username for a GUID changes how hard the identifier is to guess, not whether the server checks who is allowed to see the record. "Unpredictable" is not "secret": the same application that uses the GUID also publishes it in a blog author link. The previous lab's lesson holds exactly. The fix is an ownership check on every object reference, derived from the session, not a harder-to-guess identifier.
