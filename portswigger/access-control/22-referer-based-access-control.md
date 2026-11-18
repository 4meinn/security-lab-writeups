# Referer-Based Access Control

- **Lab:** Referer-based access control
- **Platform:** PortSwigger Web Security Academy
- **Category:** Access Control (OWASP A01:2021, Broken Access Control), access control tied to the Referer header
- **Difficulty:** Practitioner
- **Tool:** Burp Suite (Repeater)
- **URL:** https://portswigger.net/web-security/access-control/lab-referer-based-access-control

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

Administrators change user roles from the admin panel, and the panel performs that action with a `GET` request to `/admin-roles`. The access control protecting the action trusts the `Referer` header: if the request looks like it came from the admin panel, it is allowed through, with no fresh check of the caller's role. Two accounts are provided: `administrator:admin` to study the admin panel and `wiener:peter`, a normal user. The goal is to exploit the flawed control, as `wiener`, to promote `wiener` to administrator.

## Reconnaissance

Logged in as `administrator`, the admin panel upgrades and downgrades users. Upgrading a user issues:

```
GET /admin-roles?username=carlos&action=upgrade
Referer: https://0a4200e304ed4821825dbf5c00f70058.web-security-academy.net/admin
```

The privileged operation is a plain `GET`, and the only thing on it that ties it to the admin panel is the `Referer` header the browser attaches. That is the weak point: `Referer` is set by the client and can be sent at will, so anything the server decides from it is attacker-controlled.

## Exploitation

### Step 1: Capture the privileged request

With traffic proxied through Burp, I upgraded a user as `administrator` and sent the resulting `GET /admin-roles` request to Repeater. It carried the `Referer` header pointing at `/admin`.

### Step 2: Confirm the control keys on Referer

Logged in separately as `wiener`, I requested the same endpoint without the admin panel's `Referer`:

```
GET /admin-roles?username=carlos&action=upgrade
```

The server refused it as unauthorized. So the endpoint does gate the action, but on the presence of the `Referer` header rather than on the caller's role.

### Step 3: Retarget the captured request as wiener

Back in the Repeater request, which still held the valid `Referer`, I changed the username to my own account and swapped the administrator session cookie for `wiener`'s:

```
GET /admin-roles?username=wiener&action=upgrade
Cookie: session=2Fi4g5DDTlirK8J6PLwGHQRVUzuz5l8e
Referer: https://0a4200e304ed4821825dbf5c00f70058.web-security-academy.net/admin
```

The request now promotes `wiener`, sent as `wiener`, but it still carries the `Referer` the check wants to see.

### Step 4: Send and escalate

The server accepted the request and upgraded `wiener` to administrator. The identity on the session was the non-admin user; the only thing that satisfied the control was a header I copied across. The lab registered as solved. The session cookie is from a disposable lab instance and is already dead.

## Impact

The authorisation check read a request header, `Referer`, that the client sets and can forge, instead of the server-side session and role. Any user who knows the shape of the admin action can attach the expected `Referer` and run it, so a normal account escalates to full administrator in a single request. `Referer` is especially weak ground for a security decision: it is routinely stripped, spoofed, or rewritten, and was never meant to carry trust. A control built on it is both trivial to bypass and liable to break for legitimate users whose browser or proxy withholds the header.

## Remediation

- **Key the access control on the session and role, server-side, not on the `Referer` header.** Decide from who is authenticated, not from where the request claims to come.
- **Never trust client-settable headers for authorisation.** `Referer`, `Origin`, `X-Forwarded-For` and the like are attacker-controlled and cannot carry a security decision.
- **Deny by default,** and apply the same identity and role check at the point the action is performed, for every route that reaches it.
- **Do not expose state-changing operations over `GET`,** and keep the real check regardless of method, since restricting the verb is hardening, not the control.

## Key takeaway

The `Referer` header is set by the client, so any access control that trusts it is trusting the attacker. Here the admin role change was allowed whenever the request carried a `Referer` pointing at `/admin`, so copying that one header onto a request sent with a normal user's session walked straight past the check. Authorisation has to be decided from the session and role on the server, never from a header the caller can write.
