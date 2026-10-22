# Blind SQL Injection with Time Delays

- **Lab:** Blind SQL injection with time delays
- **Platform:** PortSwigger Web Security Academy
- **Category:** SQL Injection (OWASP A03:2021, Injection)
- **Difficulty:** Practitioner
- **Tool:** Burp Suite (Repeater)
- **URL:** https://portswigger.net/web-security/sql-injection/blind/lab-time-delays

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

The `TrackingId` cookie is placed into a server-side SQL query, but the result is never shown and errors are handled, so there is no visible output and no error difference to read. This is blind SQL injection. The goal is to confirm the vulnerability by making the database pause for 10 seconds, using the response time as the only signal.

## Reconnaissance

The injection point is the `TrackingId` cookie, not a URL parameter. The application looks that value up in the database on every request. Because there is nothing to read on the page, I worked in Burp Suite: I found the `GET /` request in Proxy HTTP history, sent it to Repeater, and used the response time as my oracle.

## Exploitation

The payload, set as the `TrackingId` cookie value:

```
TrackingId=x'||pg_sleep(10)--
```

`'` closes the string the cookie value sits in, `||` (PostgreSQL string concatenation) appends `pg_sleep(10)` which tells the database to wait 10 seconds, and `--` comments out the rest of the original query. Sent from Repeater, the response came back in **10,048 ms** instead of the usual few milliseconds, and the lab registered as solved. The deliberate delay, with nothing shown on the page, is the proof that the injection ran.

## Impact

There is no visible output and no error difference, yet the injection is fully exploitable. Time is the channel: by making the database wait only when a chosen condition is true, an attacker can ask it yes/no questions and read any data one character at a time, with nothing ever appearing on the page. A blind injection is as serious as a visible one, only slower to exploit.

## Remediation

- **Parameterised queries (prepared statements)** for every value placed into a query, including values from cookies and headers, not only form fields and URL parameters. Bound input cannot add `pg_sleep` or a comment.
- Do not put raw request data into SQL. The cookie here was trusted straight into a query.

## Key takeaway

The injection point was a cookie, a reminder that any attacker-controlled input reaching a query is a candidate, not just visible form fields. With no output and no errors the attack still works through a side channel: response time. `pg_sleep(10)` turns "did my condition hold" into "did the page take 10 seconds", which is the foundation for extracting data blind, one yes/no answer at a time.
