# Multi-Step Process with No Access Control on One Step

- **Lab:** Multi-step process with no access control on one step
- **Platform:** PortSwigger Web Security Academy
- **Category:** Access Control (OWASP A01:2021, Broken Access Control), missing check on one step of a multi-step process
- **Difficulty:** Practitioner
- **Tool:** Burp Suite (Repeater)
- **URL:** https://portswigger.net/web-security/access-control/lab-multi-step-process-with-no-access-control-on-one-step

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

Changing a user's role in the admin panel is a two-step process: a first request loads a confirmation, and a second request, carrying `confirmed=true`, actually applies the change. The access control guards the earlier part of the flow, but the final confirmation step performs the change without re-checking that the caller is an administrator. Two accounts are provided: `administrator:admin` to study the flow and `wiener:peter`, a normal user. The goal is to exploit the gap, as `wiener`, to promote `wiener` to administrator.

## Reconnaissance

Logged in as `administrator`, upgrading a user is not a single click. The first `POST /admin-roles` (`username=carlos&action=upgrade`) returns a confirmation page, and confirming sends a second request:

```
POST /admin-roles
Content-Type: application/x-www-form-urlencoded

action=upgrade&confirmed=true&username=carlos
```

This second request, the one with `confirmed=true`, is what actually changes the role. The question is whether that final request verifies the caller's privilege on its own, or whether it simply trusts that anyone reaching it must already have passed through the guarded first step.

## Exploitation

### Step 1: Capture the final step

With the traffic proxied through Burp, I performed an upgrade as `administrator` and sent the second `POST /admin-roles` request, the one carrying `confirmed=true`, to Repeater.

### Step 2: Retarget it and drop the privilege

I changed the body to `action=upgrade&confirmed=true&username=wiener` so the request promotes my own account, and I replaced the administrator session cookie with `wiener`'s session. The request is now the final confirmation step, sent by the non-admin user `wiener`, with none of the earlier steps performed.

### Step 3: Send it

The server accepted the request and applied the change:

```
HTTP/2 302 Found
Location: /admin
```

`wiener` was now an administrator, confirmed by loading `wiener`'s account. The lab registered as solved.

## Impact

The access control was applied to part of the multi-step flow but not to the step that performs the change. The application assumed that anyone reaching the final confirmation request must have legitimately moved through the earlier, guarded steps, so the final request re-checked nothing. An attacker just sends that request directly. A normal user escalates to administrator by replaying a single request and skipping the entire guarded part of the process.

## Remediation

- **Enforce access control on every step of a sensitive multi-step process,** above all the step that commits the state change. Do not leave any step unguarded.
- **Re-verify the caller's identity and role at the point of action,** server-side, on the final request, independent of how it was reached.
- **Do not treat progress through a UI flow as proof of authorisation.** Each request stands on its own and must be checked on its own; reaching a later step is not evidence that an earlier gate was passed.
- **Deny by default.**

## Key takeaway

A guard on the front door counts for nothing if the back door is open. The role change was gated at the start of the flow, but the request that actually made the change checked nothing, on the assumption that it could only be reached by passing the gate. Requests are independent, so an attacker sends the final one directly. Every step of a privileged process, and especially the one that commits the change, has to enforce the same authorisation check.
