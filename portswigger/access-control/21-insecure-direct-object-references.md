# Insecure Direct Object References (IDOR)

- **Lab:** Insecure direct object references
- **Platform:** PortSwigger Web Security Academy
- **Category:** Access Control (OWASP A01:2021, Broken Access Control), IDOR against a static file
- **Difficulty:** Apprentice
- **Tool:** Web browser
- **URL:** https://portswigger.net/web-security/access-control/lab-insecure-direct-object-references

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

The live chat feature lets a user download their own chat transcript. The transcript is served from a URL that names the file by a sequential number, with no check that the file belongs to the caller. That is an Insecure Direct Object Reference against a static file: change the number, read someone else's transcript. The other user's transcript contains a password. The goal is to read it and log in as `carlos`.

## Reconnaissance

Logged in as `wiener`, the chat page has a **View transcript** button that downloads a file. Clicking it does not change the address bar, because it is a file download rather than a navigation, so the request is easy to miss. The saved file, however, is named `2.txt`. That number is the direct object reference: the transcript is served from `/download-transcript/2.txt`, keyed only by a small sequential id, with no token or ownership check in the path.

## Exploitation

### Step 1: Identify the reference

The downloaded file name, `2.txt`, revealed both the endpoint and the id. `/download-transcript/2.txt` returns my own transcript, and the id is a low sequential number.

### Step 2: Change the reference

I requested the previous id directly in the address bar:

```
/download-transcript/1.txt
```

The server returned another user's transcript with no check that it was mine. It was carlos's conversation.

### Step 3: Harvest the credential

In that transcript, carlos had read his password out to the chatbot to have it confirmed:

```
You: Ok so my password is lv4nq76az4ovpmpvwtw3. Is that right?
Hal Pline: Yes it is!
```

### Step 4: Take over the account

I logged out and logged back in as `carlos` with that password. The lab registered as solved. The credentials are from a disposable lab instance and are already dead.

## Impact

Any user can read any other user's chat transcript by changing one number in a URL, because the files are named by a sequential id and served with no ownership check. The transcripts hold whatever users typed, here a plaintext password, so the flaw is not only a privacy breach but a direct account takeover. Sequential, guessable identifiers make the whole set trivially enumerable: a short loop downloads every transcript on the server.

## Remediation

- **Enforce an ownership check on every download.** The server must confirm the requested transcript belongs to the authenticated session before returning it, and never serve a file purely because the client named it.
- **Do not expose direct, guessable references to server-side objects.** Resolve a user's files through their session, or use unguessable identifiers. The access check is the real fix; the id format is not.
- **Never store or transmit passwords in plaintext,** including in chat logs. A support bot inviting a user to type their password is its own failure.
- **Treat the exposed credential as compromised and rotate it.**

## Key takeaway

An insecure direct object reference is the server handing over an object because the client named it, without checking the client is allowed to have it. Here the transcripts were numbered 1, 2, 3 and served with no ownership check, so changing 2 to 1 read another user's chat, and that chat held a password. The fix is an authorisation check on every object access, tied to the session. An unguessable file name would slow enumeration down, but it still would not decide who is allowed to read the file.
