# SQL Injection Login Bypass: Logging In Without a Password

- **Lab:** SQL injection vulnerability allowing login bypass
- **Platform:** PortSwigger Web Security Academy
- **Category:** SQL Injection (OWASP A03:2021, Injection)
- **Difficulty:** Apprentice
- **URL:** https://portswigger.net/web-security/sql-injection/lab-login-bypass

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

The login form authenticates a user by looking them up in the database. For a submitted username and password the back end builds a query like:

```sql
SELECT * FROM users WHERE username = '<username>' AND password = '<password>'
```

If the query returns a matching row, the user is logged in. The goal is to log in as `administrator` without knowing the password.

## Reconnaissance

The form sends a username and a password, and both are placed directly into the query. The username field is the injection point. Closing the string there lets me change what the query actually checks.

## Exploitation

### Attempt 1: bypass with an always-true condition

Username: `' OR 1=1 --`
Password: any non-empty value (see the note below).

The query becomes:

```sql
SELECT * FROM users WHERE username = '' OR 1=1 --' AND password = '...'
```

`OR 1=1` is true for every row, and `--` comments out the password condition. Every row matches, and the application logs in as the first row returned, which here was `wiener`. This proves authentication can be bypassed with no valid credentials, but it gives no control over which account I land on, and I need the administrator.

Note on the password field: leaving it empty was blocked by the browser's client-side "required field" check, which stops the form from submitting. That is validation in the browser, not on the server. Any value such as `asdasdasd` satisfies it. The value is irrelevant because `--` already discarded the password condition from the query.

### Attempt 2: guessing the admin username

Username: `admin'--`

The query becomes:

```sql
SELECT * FROM users WHERE username = 'admin'--' AND password = '...'
```

After the comment it reduces to `... WHERE username = 'admin'`. No user named `admin` exists, so zero rows are returned and login is refused with "Invalid username or password". This shows that `--` only removes the password check. The `username = '...'` condition stays and must match a real row, so I still need the correct username.

### Attempt 3: the real username (success)

Username: `administrator'--`

The query becomes:

```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = '...'
```

The `'` closes the username string and `--` comments out the rest, removing `AND password = '...'`. The query reduces to a lookup of the row where username is `administrator`. That row exists, so it is returned and I am logged in as administrator. **Lab solved.**

## Impact

An attacker can log in as any account without its password. With `administrator'--` they take over the admin account directly. With `wiener'--` or `carlos'--` they take over a specific customer account whose username they know, guessed, or extracted. With `' OR 1=1 --` they log in as whichever account comes first. The weakness is in how the query is built, not in any single account's credentials, so every account is exposed.

## Remediation

- **Parameterised queries (prepared statements).** Username and password are bound as values, so input can never close the string or add a comment. This removes the injection at its root.
- **Verify the password properly.** Store password hashes and compare the submitted password against the stored hash in application code. Authentication must not be "did the query return a row", which is exactly what the comment bypassed here.
- **Do not rely on hidden usernames.** Making the admin username hard to guess is only obscurity. It does not stop `' OR 1=1 --`, and usernames leak through registration, profiles, error messages, and data extraction. It is a small speed bump, not a fix.
- Keep a single generic "invalid username or password" message, so the response does not reveal which part was wrong.

## Key takeaway

`--` removes the password check, but the `username = '...'` condition stays, so this technique still needs a valid username. `' OR 1=1 --` removes that requirement but gives up control over which account you get. Different payloads trade precision against prerequisites. The single fix for all of them is the same: stop building the query from raw input and use parameterised queries.
