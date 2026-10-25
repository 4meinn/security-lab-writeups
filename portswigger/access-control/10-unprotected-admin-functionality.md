# Unprotected Admin Functionality

- **Lab:** Unprotected admin functionality
- **Platform:** PortSwigger Web Security Academy
- **Category:** Access Control (OWASP A01:2021, Broken Access Control)
- **Difficulty:** Apprentice
- **Tool:** Web browser only
- **URL:** https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

The application has an admin panel that performs privileged actions, including deleting users. The panel enforces no access control: it is never checked who is calling it. Instead of protecting the function, the developer only kept its location unlinked, treating the URL itself as the secret. The goal is to find the panel and delete the user `carlos`.

## Reconnaissance

The admin panel is not linked anywhere in the site's navigation, so it cannot be found by clicking through the application. The place to look is `/robots.txt`, a file that tells search-engine crawlers which paths not to index. That file is served publicly to anyone, and listing a path there in order to hide it from crawlers has the opposite effect for a human: it advertises exactly the paths the developer considers sensitive.

Requesting `/robots.txt`:

```
User-agent: *
Disallow: /administrator-panel
```

The one disallowed path is the admin panel. The attempt to keep it out of search results disclosed its location in a file that is readable by everyone.

## Exploitation

### Step 1: Reach the panel

I navigated directly to the disclosed path:

```
/administrator-panel
```

The panel loaded. No login prompt, no "forbidden" response, no redirect. As an unauthenticated visitor I was looking at the admin interface, with the registered users listed and a delete control next to each one. That alone proves the vulnerability: a privileged function is reachable by anyone who knows the URL, and nothing on the server checks whether the caller is allowed to use it.

### Step 2: Perform the privileged action

I clicked **Delete** next to `carlos`. The action went through and the user was removed. The lab registered as solved.

## Impact

Any unauthenticated visitor can reach the admin panel and delete arbitrary user accounts. The only thing standing between an attacker and full administrative functionality was knowledge of the URL, and `robots.txt` handed that URL out. "Delete a user" is the specific action here, but the real exposure is the whole admin interface sitting open: whatever privileged operations it offers are available to everyone, with no authentication and no authorization.

## Remediation

- **Enforce access control on the server, for every request.** The application must check the caller's identity and role on each call to the admin panel and to each action it performs, and deny by default. Authorization is the control; the URL being hard to guess is not.
- **Do not rely on secret URLs.** Security through obscurity is not access control. A path that is unlinked is still reachable, and anything that discloses it (robots.txt, JavaScript, referrer headers, logs) removes even that thin cover.
- **Keep sensitive paths out of robots.txt.** The file is public and is one of the first places an attacker looks. A path that must be hidden from crawlers still needs real authorization; robots.txt only publishes its existence.
- **Separate and harden admin interfaces** where possible: a distinct host or network segment, plus strong authentication, so the admin surface is not simply another path on the public application.

## Key takeaway

Access control is an authorization problem, not a discoverability one. A function that can be reached is a function that has to be authorized on the server, on every request. Hiding the URL does not protect the function, it only delays the discovery of an open door, and `robots.txt` shortens that delay to nothing by naming the door out loud.
