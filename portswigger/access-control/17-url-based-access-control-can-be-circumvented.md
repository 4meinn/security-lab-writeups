# URL-Based Access Control Can Be Circumvented

- **Lab:** URL-based access control can be circumvented
- **Platform:** PortSwigger Web Security Academy
- **Category:** Access Control (OWASP A01:2021, Broken Access Control), front-end access control bypass via HTTP header
- **Difficulty:** Apprentice
- **Tool:** Burp Suite (Repeater)
- **URL:** https://portswigger.net/web-security/access-control/lab-url-based-access-control-can-be-circumvented

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

Access control for the admin panel is enforced at the front-end, which blocks any request whose URL is `/admin`. The back-end, however, honours the `X-Original-URL` header and routes on its value instead of on the request line. Because the front-end only inspects the request-line URL, a request that looks harmless to it can still resolve to `/admin` on the back-end. The goal is to reach the admin panel this way and delete the user `carlos`.

## Reconnaissance

Requesting `/admin` directly returned a block. The response was stripped-down and generic, with none of the application's usual layout, a strong hint that it came from a front-end layer sitting in front of the application rather than from the application itself. That split is the whole setup: if the front-end is the only thing enforcing the rule, anything that reaches the back-end without the front-end recognising it as `/admin` is unguarded.

Some front-end/back-end pairs support headers such as `X-Original-URL` or `X-Rewrite-URL`, which let an upstream component tell the application which path to serve. If the back-end trusts that header, it, and not the request line, decides the route.

## Exploitation

### Step 1: Confirm the back-end honours `X-Original-URL`

I sent the request to Burp Repeater, set the request line to a path the front-end allows (`/`), and added the header pointing at a path that does not exist:

```
GET / HTTP/2
X-Original-URL: /invalid
```

The back-end answered `Not Found`. That is the tell: the front-end allowed the request because the request line was `/`, and the back-end resolved the route from the header value, found nothing at `/invalid`, and returned a 404. A couple of other bogus paths (random strings) behaved the same way, confirming that the header, not the request line, drives routing on the back-end.

### Step 2: Reach the admin panel

I changed the header value to `/admin`:

```
GET / HTTP/2
X-Original-URL: /admin
```

The back-end served the admin panel. The front-end still saw only `/` on the request line, so its `/admin` rule never fired. The panel listed the users and their delete links, exposing the delete endpoint and its parameter: `/admin/delete?username=...`.

### Step 3: Delete carlos

The delete endpoint takes its target from a `username` query parameter. The query string has to ride on the real request line, since the back-end reads it from there and not from the `X-Original-URL` value, so I put `username=carlos` on the request line and pointed the header at the delete path:

```
GET /?username=carlos HTTP/2
X-Original-URL: /admin/delete
```

The back-end routed to `/admin/delete`, read `username=carlos` from the query string, and deleted the account. The lab registered as solved.

## Impact

Access control enforced only at the front-end is access control that can be bypassed. Here the back-end trusted a client-supplied header (`X-Original-URL`) to decide which route to serve, while the front-end's rule looked only at the request-line URL. Any user who knows the header can walk straight past the restriction and reach privileged functionality, in this case user deletion, which is effectively full control over accounts. Trusting the header turns a single-layer URL rule into no rule at all.

## Remediation

- **Enforce access control at the back-end, on the application itself,** not only at a front-end proxy. The component that executes the privileged action must be the one that checks the caller is authorised.
- **Do not trust URL-rewriting headers from clients.** `X-Original-URL`, `X-Rewrite-URL` and similar should be stripped at the edge unless they originate from a trusted component, and the back-end should route on the actual request rather than on an overridable header.
- **Base authorisation on identity and role, not on the URL string.** Deny by default, and apply the check at the point of action regardless of how the route was reached.
- **Keep the front-end and back-end views of the request in sync,** so a path the front-end permits cannot resolve to a different, restricted path on the back-end.

## Key takeaway

A URL rule is only as good as the layer that can be routed around it. The front-end blocked `/admin`, but the back-end let a client header rewrite the path after the check had already passed, so a request that reads as `/` to the guard becomes `/admin` to the application. Access control belongs on the component that performs the action, keyed to who is asking rather than to a URL string an attacker can bypass.
