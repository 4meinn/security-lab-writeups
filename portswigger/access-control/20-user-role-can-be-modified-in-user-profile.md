# User Role Can Be Modified in User Profile (Mass Assignment)

- **Lab:** User role can be modified in user profile
- **Platform:** PortSwigger Web Security Academy
- **Category:** Access Control (OWASP A01:2021, Broken Access Control), privilege escalation via mass assignment
- **Difficulty:** Apprentice
- **Tool:** Burp Suite (Repeater)
- **URL:** https://portswigger.net/web-security/access-control/lab-user-role-can-be-modified-in-user-profile

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

The admin panel at `/admin` is restricted to users whose `roleid` is 2. The profile update feature (change email) sends its data as a JSON body, and the server binds the fields in that body onto the user record without an allowlist. That means a field the user should never control, `roleid`, can be set from the request. This is mass assignment. The goal is to set my own `roleid` to 2, reach the admin panel, and delete the user `carlos`.

## Reconnaissance

Logged in as `wiener:peter`, the account page has an update-email feature. Submitting it sends:

```
POST /my-account/change-email
Content-Type: text/plain;charset=UTF-8

{"email":"metin@sen.com"}
```

The useful detail is the response. The server answers with the full account object, which includes a `roleid`:

```
{"username":"wiener","email":"metin@sen.com","apikey":"...","roleid":1}
```

So the account carries a `roleid` (1 for a normal user), and the update endpoint round-trips it. If that endpoint binds whatever fields arrive in the request, `roleid` can be set from the request body.

## Exploitation

### Step 1: Identify the role field

Sending the email change and reading the JSON response confirmed the field name is `roleid` and a normal user's value is `1`.

### Step 2: Mass-assign the role

In Repeater I added `roleid` to the JSON body:

```
{"email":"metin@sen.com","roleid":2}
```

and sent it. The response came back with the new value:

```
{"username":"wiener","email":"metin@sen.com","apikey":"...","roleid":2}
```

The server accepted the extra field and updated the role from attacker-controlled input. The `Content-Type` stayed `text/plain`; the endpoint parsed the body as JSON regardless.

### Step 3: Use the new privilege

With `roleid` now 2, the admin panel at `/admin` became accessible. I deleted `carlos`. The lab registered as solved.

## Impact

The profile update binds request fields straight onto the user record with no allowlist, so an attribute the user must never control, `roleid`, is settable from the request body. A normal user promotes themselves to administrator by adding one key to a routine email-change request. Mass assignment turns an innocuous self-service endpoint into privilege escalation whenever a sensitive attribute lives on the same object the endpoint writes.

## Remediation

- **Bind only an explicit allowlist of fields from the request.** A change-email endpoint should accept `email` and nothing else. Never pass the raw request object into the data model.
- **Keep security-sensitive attributes (`roleid`, permissions) off any object a user-facing endpoint can write,** and set them only through a separate, access-controlled path.
- **Do not return sensitive fields like `roleid` to the client.** This is hardening, not the fix; it only hides the hint. The field binding is the real problem.
- Server-side authorisation on `/admin` by role was correct here. The hole was letting the user set their own role.

## Key takeaway

Mass assignment is letting the shape of the request decide what gets written. Here a change-email endpoint bound every JSON field onto the account, so adding `"roleid":2` to an email update made the user an administrator. An endpoint must write only the fields it is meant to, chosen by an allowlist on the server, never whatever the client happens to send. A user should never be able to set their own role.
