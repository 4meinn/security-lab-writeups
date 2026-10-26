# Unprotected Admin Functionality with Unpredictable URL

- **Lab:** Unprotected admin functionality with unpredictable URL
- **Platform:** PortSwigger Web Security Academy
- **Category:** Access Control (OWASP A01:2021, Broken Access Control)
- **Difficulty:** Apprentice
- **Tool:** Web browser only
- **URL:** https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality-with-unpredictable-url

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

The application has an admin panel that can delete users and enforces no access control on it. Unlike the previous lab, the panel's location is not a guessable path and is not listed in `robots.txt`. The address is randomised, and the developer relies on that randomness to keep the panel hidden. The problem is that the address is shipped to every visitor inside the page's own JavaScript. The goal is to recover the URL, reach the panel, and delete the user `carlos`.

## Reconnaissance

`robots.txt` was the giveaway in the previous lab. Here it does not list the panel, so that route is closed. The home page, however, carries a block of client-side JavaScript that decides whether to show an "Admin panel" link:

```html
<script>
var isAdmin = false;
if (isAdmin) {
    var topLinksTag = document.getElementsByClassName("top-links")[0];
    var adminPanelTag = document.createElement('a');
    adminPanelTag.setAttribute('href', '/admin-5f8l0t');
    adminPanelTag.innerText = 'Admin panel';
    topLinksTag.append(adminPanelTag);
    ...
}
</script>
```

Two problems are visible at once. The authorization decision, `isAdmin`, is made in the browser, in code the user fully controls. And the panel's real path, `/admin-5f8l0t`, is written directly into that script. `isAdmin` is `false`, so the link is never drawn on the page, but the script is still delivered in full to every visitor. The URL is unlinked, not secret. (The random suffix differs on each lab instance.)

## Exploitation

### Step 1: Recover the URL from the page source

Viewing the home page source (`Ctrl+U`) showed the script above. Reading the `href` gave the panel's path with no guessing required: `/admin-5f8l0t`.

### Step 2: Reach the panel

I navigated straight to that path. The panel loaded for me as an unauthenticated visitor, with no login prompt. The server never consulted the `isAdmin` value at all; that check only ever existed in the browser. The user list and its delete controls were there.

### Step 3: Perform the privileged action

I clicked **Delete** next to `carlos`. The user was removed and the lab registered as solved.

## Impact

This is the same failure as the predictable-URL case, with the hiding place moved from `robots.txt` into client-side code. Access control is enforced nowhere on the server: it is a flag in JavaScript, and the browser belongs to the attacker. The "unpredictable" URL that was meant to compensate is handed to every visitor in the page they load, so it compensates for nothing. Anyone who reads the page source gains administrative access and can delete any account.

## Remediation

- **Enforce access control on the server, for every request,** and deny by default. The `isAdmin` decision belongs in a server-side session check, not in a script the client can read and edit.
- **Never make authorization decisions in client-side code.** Anything sent to the browser is visible to and controllable by the user. A JavaScript `isAdmin` gate controls nothing.
- **Do not embed privileged URLs in client-side code.** If the server does not authorize the panel, randomising its address buys nothing, because the address is shipped to everyone.
- **Keep the admin surface genuinely separated and authenticated,** not merely unlinked.

## Key takeaway

Client-side access control is not access control. Everything the server sends to the browser, the user can read: the `isAdmin` check and the panel's "secret" URL were both sitting in the page source. Randomising the URL only changes how the address is found, from guessing to reading. It does not decide who is allowed through. That decision has to be made on the server, on every request, or there is no decision at all.
