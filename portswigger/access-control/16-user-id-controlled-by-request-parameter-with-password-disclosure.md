# User ID Controlled by Request Parameter with Password Disclosure (IDOR)

- **Lab:** User ID controlled by request parameter with password disclosure
- **Platform:** PortSwigger Web Security Academy
- **Category:** Access Control (OWASP A01:2021, Broken Access Control), IDOR leading to account takeover
- **Difficulty:** Apprentice
- **Tool:** Web browser (DevTools)
- **URL:** https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-password-disclosure

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

The account page loads a user's data from an `id` in the request, the same IDOR pattern as the earlier labs. On top of that, the page renders the account's current password into the change-password form, masked on screen but present in the HTML. Chained together, the two let a normal user read the administrator's password. The goal is to recover it, log in as `administrator`, and delete the user `carlos`.

## Reconnaissance

I logged in with `wiener:peter`. My account page at `/my-account?id=wiener` had a change-password form with the current password already filled in, shown as dots because the field is `type="password"`. Masking is a display behaviour only: the real value is still delivered to the browser in the field's `value` attribute. Combined with the `id` parameter having no ownership check, that means I can request another user's account page and read their password straight out of the markup.

## Exploitation

### Step 1: Load the administrator's account page

I changed the `id` parameter from `wiener` to `administrator`:

```
/my-account?id=administrator
```

The server returned the administrator's account page, with no check that my session belonged to that account.

### Step 2: Read the password from the source

The change-password field showed dots on screen, but inspecting the element (DevTools) revealed the cleartext password sitting in its `value` attribute:

```html
<input required type="password" name="password" value="6cpc5dh3eozgbjdu014g">
```

The administrator's password was `6cpc5dh3eozgbjdu014g`. The masking hid nothing; it was in the HTML the whole time.

### Step 3: Take over the account and act

I logged out, logged back in as `administrator` with that password, opened the admin panel, and deleted `carlos`. The lab registered as solved. The credentials are from a disposable lab instance and are already dead.

## Impact

Two weaknesses chain into a full account takeover. The IDOR lets any authenticated user load another user's account page, and that page returns the account's password to the client. `type="password"` only hides the value visually, so the IDOR does not merely leak data, it leaks a credential, and the leaked credential belongs to an administrator. One normal user goes from reading a page to full administrative control of the application.

## Remediation

- **Never send a password back to the client, in any field or attribute.** Stored passwords should be salted hashes that cannot be reversed, and a change-password form should start empty, never pre-filled with the current password.
- **Enforce an ownership check on every record access,** derived from the session, so one user cannot load another user's account page. This is the root IDOR fix.
- **Do not treat `type="password"` as a security control.** Masking is presentation. Anything delivered to the browser is readable in the source.
- **Rotate the exposed administrator credential immediately,** and review for any use of it.

## Key takeaway

Masking is not protection. `type="password"` turns the characters into dots on screen, but the value travels to the browser in plain text and is visible in the source. Here that display trick sat on top of an IDOR, so a normal user could open the administrator's account page and read the administrator's password out of a form field. The data should never have reached the client: a password has no business being returned to the browser at all, and the account page should have been gated by an ownership check before it was ever built.
