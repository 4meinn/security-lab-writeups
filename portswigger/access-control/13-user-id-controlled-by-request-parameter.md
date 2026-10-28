# User ID Controlled by Request Parameter (IDOR)

- **Lab:** User ID controlled by request parameter
- **Platform:** PortSwigger Web Security Academy
- **Category:** Access Control (OWASP A01:2021, Broken Access Control), IDOR / horizontal privilege escalation
- **Difficulty:** Apprentice
- **Tool:** Web browser only
- **URL:** https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

The account page loads a user's data from an ID supplied in the request: `/my-account?id=wiener`. The server returns whatever account the `id` names, without checking that it belongs to the caller. That is a horizontal privilege escalation, the classic Insecure Direct Object Reference (IDOR): same privilege level, but access to another user's data. The goal is to read the API key of the user `carlos`.

## Reconnaissance

I logged in with the supplied account `wiener:peter`. The account page opened at:

```
/my-account?id=wiener
```

The page showed wiener's own username and API key. The identity being displayed is taken straight from the `id` query parameter, a value that sits in the URL and is fully under the client's control. The only question is whether the server checks that the requested `id` matches the logged-in session, or whether it simply returns whatever record is asked for.

## Exploitation

### Step 1: Change the identifier

I edited the `id` parameter in the URL from `wiener` to `carlos`:

```
/my-account?id=carlos
```

### Step 2: Read another user's data

The server returned carlos's account page in full, including his API key, with no check that my session belonged to carlos. Authentication confirmed I was a valid user; nothing confirmed I was allowed to see this particular record.

```
Your username is: carlos
Your API Key is: OEvolJeiCvcHpezTW1esk3HEb5Qys5zj
```

### Step 3: Solve

Submitting carlos's API key solved the lab. The key is from a disposable lab instance and is already dead.

## Impact

Any authenticated user can read any other user's account data by substituting an identifier in one request parameter. There is no authorisation on the record being requested, only authentication that the caller is some valid user. The leaked item here is an API key, which is a credential, so the exposure is not only a data leak: it lets the attacker act as that user against the API. When the identifiers are sequential or otherwise guessable, the same flaw scales to harvesting every account on the site in a simple loop.

## Remediation

- **Check ownership on every object access.** The server must confirm the requested record belongs to the authenticated session before returning it, not merely that the caller is logged in.
- **Derive the user's identity from the session, not from a request parameter.** For a user's own account page, the ID should come from the server-side session; it should never be read from the URL.
- **Unguessable identifiers (such as GUIDs) are defence in depth, not the fix.** An access control check is the fix. An unguessable ID still leaks through referrers, logs, browser history, and shared links.
- **Treat the exposed API key as compromised and rotate it.** Any secret that has been returned to an unauthorised party must be revoked.

## Key takeaway

This is IDOR, horizontal privilege escalation. Authentication proves who you are; it does not decide what you are allowed to see. When the server loads a record from an identifier in the request and hands it back without checking ownership, any logged-in user can read anyone's data by changing that one value. The fix is an ownership check derived from the session, enforced on every object reference, not a harder-to-guess identifier.
