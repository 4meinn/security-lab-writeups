# User Role Controlled by Request Parameter

- **Lab:** User role controlled by request parameter
- **Platform:** PortSwigger Web Security Academy
- **Category:** Access Control (OWASP A01:2021, Broken Access Control)
- **Difficulty:** Apprentice
- **Tool:** Web browser (DevTools)
- **URL:** https://portswigger.net/web-security/access-control/lab-user-role-controlled-by-request-parameter

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

There is an admin panel at `/admin` that is meant to be restricted to administrators. The first two labs in this series left the admin function unprotected and only hid its URL. This one actually performs a check, but it reads the caller's role from an `Admin` cookie, a value that is sent with every request and is fully controlled by the client. The goal is to escalate from a normal account (`wiener`) to administrator and delete the user `carlos`.

## Reconnaissance

I logged in with the supplied account `wiener:peter`, a normal non-admin user. After login the server set two cookies:

```
session=OdJZ0k9seCQI24DWUua7R3ToK4wIN8jZ
Admin=false
```

The `session` cookie is the real identifier. The `Admin` cookie is the interesting one: the application uses it to decide whether the caller is an administrator. Two things stand out. The role is carried in the request itself, so it is attacker-controlled, and the `Admin` cookie is not flagged `HttpOnly`, so it can be read and edited straight from the browser with no intercepting proxy. Visiting `/admin` as `wiener` (`Admin=false`) is denied.

## Exploitation

### Step 1: Locate the role value

In DevTools, under Application, Cookies, the `Admin=false` cookie sat next to the session cookie. This, not the session, is what gates the admin panel, and it is a plain editable cookie.

### Step 2: Forge the role

I double-clicked the `Admin` value and changed it from `false` to `true`. Because the cookie is not `HttpOnly`, the browser accepted the edit directly. From now on every request carries `Admin=true`.

### Step 3: Reach the panel and act

I navigated to `/admin`. With `Admin=true` in the request, the server treated me as an administrator and returned the panel, with the user list and a delete control for each account. I deleted `carlos`. The lab registered as solved.

## Impact

A normal authenticated user can promote themselves to administrator by flipping a single cookie value, then reach every function the admin panel offers. The boundary between user and admin is enforced by nothing on the server: the role is taken from input the user owns. The cookie not being `HttpOnly` makes the attack trivial, editable from the browser, but even with `HttpOnly` set the attacker would only have to change the value in an intercepting proxy instead. The weakness is not where the value is stored, it is that the server trusts a client-supplied value for the authorisation decision at all.

## Remediation

- **Derive the role from server-side session state, never from the request.** The session identifier should map, on the server, to the account and the privileges it holds. The client should have no say in its own role.
- **Do not trust client-controlled input for authorisation.** Cookies, query and body parameters, and headers are all attacker-controllable and must never drive an access decision.
- **Enforce access control on `/admin` and on every action it exposes,** with that server-side role check, denying by default.
- Not placing the role in a cookie, and setting `HttpOnly`, are hardening steps only. They raise the effort slightly; they do not fix the root cause.

## Key takeaway

The first two labs in this series hid the admin panel and checked nothing. This one checks something, but it checks a value the user controls. A cookie, a parameter, or a header is part of the request, and the request is written by the client, so basing "are you an admin" on `Admin=true` is asking the attacker to grade their own exam. Authorisation has to be derived from trusted server-side session state, not read back from input the client can edit.
