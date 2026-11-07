# Method-Based Access Control Can Be Circumvented

- **Lab:** Method-based access control can be circumvented
- **Platform:** PortSwigger Web Security Academy
- **Category:** Access Control (OWASP A01:2021, Broken Access Control), access control tied to HTTP method
- **Difficulty:** Practitioner
- **Tool:** Burp Suite (Repeater)
- **URL:** https://portswigger.net/web-security/access-control/lab-method-based-access-control-can-be-circumvented

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

Administrators can change user roles through the admin panel, an action the interface performs with a `POST` request. The access control that protects that action is bound to the `POST` method rather than to the caller's identity, so the same operation sent with a different method is not checked. Two accounts are provided: `administrator:admin` to study the admin panel and `wiener:peter`, a normal user. The goal is to exploit the flawed control, as `wiener`, to promote `wiener` to administrator.

## Reconnaissance

Logged in as `administrator`, the admin panel has a control to upgrade or downgrade users. Upgrading a user issues:

```
POST /admin-roles
Content-Type: application/x-www-form-urlencoded

username=carlos&action=upgrade
```

and the server answers `302`, applying the change. This is the privileged operation. The question is what the protection on it actually checks: the user's role, or just the shape of the request.

## Exploitation

### Step 1: Capture the privileged request

With the traffic proxied through Burp, I performed a role upgrade as `administrator` and sent the resulting `POST /admin-roles` request to Repeater.

### Step 2: Retarget it and drop the privilege

I changed the body to `username=wiener&action=upgrade` so the request promotes my own account, and I replaced the administrator session cookie with `wiener`'s session. The request now asks to make `wiener` an administrator, sent as the non-admin user `wiener`.

### Step 3: Confirm the control blocks the POST

Sent as `POST` with `wiener`'s session, the server refused:

```
HTTP/2 401 Unauthorized
"Unauthorized"
```

So the endpoint does enforce access control against a non-administrator, for the `POST` method.

### Step 4: Bypass by changing the method

Using Burp's **Change request method**, I converted the request to `GET`, which moved the parameters onto the query string, still carrying `wiener`'s session:

```
GET /admin-roles?username=wiener&action=upgrade
```

The server accepted it and upgraded `wiener` to administrator. The access control that rejected the `POST` never ran for the `GET`. The lab registered as solved.

## Impact

The authorisation check was wired to the HTTP method the admin interface happened to use, not to the action and the caller. Because only `POST` was ever expected, only `POST` was guarded, and the identical privileged operation was reachable by `GET` with no check at all. A normal user escalates to full administrator in a single request. Any protection keyed to the shape of a request rather than to who is making it leaves the operation open through every method and route the developer did not explicitly think to cover.

## Remediation

- **Enforce the access control on the action and the caller's role, independent of the HTTP method.** The same authorisation check must run for `POST`, `GET`, and every other method.
- **Deny by default.** Do not assume only the method the UI uses will ever be sent to an endpoint.
- **Do not expose state-changing operations over `GET`,** and still apply the identity and role check regardless of method; restricting the method is hardening, not the control.
- **Apply the check server-side, at the point where the action is performed.**

## Key takeaway

Access control has to key on who is asking and what they are trying to do, never on how the request is shaped. Here the check was attached to the `POST` method the admin panel used, so re-sending the exact same action as a `GET` walked straight around it. A privileged operation must be guarded by the same identity and role check on every method and every route, otherwise an attacker simply chooses the one the developer forgot to protect.
