# User ID Controlled by Request Parameter with Data Leakage in Redirect (IDOR)

- **Lab:** User ID controlled by request parameter with data leakage in redirect
- **Platform:** PortSwigger Web Security Academy
- **Category:** Access Control (OWASP A01:2021, Broken Access Control), IDOR / horizontal privilege escalation
- **Difficulty:** Apprentice
- **Tool:** Burp Suite (Repeater)
- **URL:** https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-data-leakage-in-redirect

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

The account page loads a user's data from an `id` in the request, the same IDOR pattern as before. This time the application does notice that the caller is not allowed to view another user's page and responds with a redirect to the login page. The catch is that it builds the full response body first and only then redirects, so the data is still in the body of the redirect response. The goal is to read it and obtain carlos's API key.

## Reconnaissance

I logged in with `wiener:peter`. My account page was at `/my-account?id=wiener`, with the `id` being the predictable username again, not a GUID. Requesting another user's account in the browser appears to do nothing useful: the page just bounces to the login screen. That bounce is a `302` redirect, and a browser follows it automatically, so the response that triggered it is never shown. To see what the server actually returned before the redirect, the request has to be inspected in a tool that does not follow redirects on its own. That is Burp.

## Exploitation

### Step 1: Send the account request to Repeater

With the traffic proxied through Burp, I found the `GET /my-account?id=wiener` request in the HTTP history and sent it to Repeater.

### Step 2: Request another user's account

I changed `id=wiener` to `id=carlos` and sent the request. The response was a redirect:

```
HTTP/2 302 Found
Location: /login
Content-Length: 3759
```

### Step 3: Read the leaked body

A `302` with a `Location` header is an instruction to the browser to go elsewhere, but it can still carry a body, and this one did. The 3759 byte body was carlos's complete account page, including his API key:

```html
<h1>My Account</h1>
<div id=account-content>
    <p>Your username is: carlos</p>
    <div>Your API Key is: 0k6PUWqa44C52lKROeFWkHO0pEF4CvKo</div>
    ...
</div>
```

In the browser this is invisible: it follows the `302` to `/login` and the account data never reaches the screen. In Repeater the raw response is shown as-is, so the leaked body is right there.

### Step 4: Solve

Submitting carlos's API key `0k6PUWqa44C52lKROeFWkHO0pEF4CvKo` solved the lab. The key is from a disposable lab instance and is already dead.

## Impact

The underlying flaw is the familiar IDOR: there is no ownership check on the requested `id`, so one user can ask for another user's account. The redirect makes it look handled, but it is cosmetic. The server computed and returned the victim's data, including a credential, and merely told the browser to navigate away afterwards. Any attacker using an intercepting proxy, which is a trivial and standard tool, reads the data straight out of the response. On a real application this leaks every user's data and credentials despite an apparent access control being in place.

## Remediation

- **Enforce the access control before building the response.** If the caller is not authorised to see the record, the server must not put that record in the response at all. The ownership check has to happen first, and on failure the response carries no victim data.
- **A redirect is not a security control.** `302 Location` is a client-side instruction. It does not protect anything the server has already placed in the body, and it is trivially ignored.
- **Derive identity from the session, not from a request parameter,** and check ownership on every object access, the same root fix as every IDOR.
- **Treat the exposed API key as compromised and rotate it.**

## Key takeaway

A redirect only tells the browser where to go next; it does nothing to the data the server already wrote into the response. Here the application checked access late, built carlos's page, put his API key in it, and then bolted a `302` on top, so the browser hid the leak while the server had already handed it over. Access control has to be enforced before the response is composed, not patched on afterwards with a redirect, and a browser quietly following that redirect is exactly why the raw response has to be read in a proxy.
