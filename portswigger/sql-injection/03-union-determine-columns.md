# SQL Injection UNION Attack: Determining the Number of Columns

- **Lab:** SQL injection UNION attack, determining the number of columns returned by the query
- **Platform:** PortSwigger Web Security Academy
- **Category:** SQL Injection (OWASP A03:2021, Injection)
- **Difficulty:** Practitioner
- **URL:** https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

A UNION-based SQL injection appends the results of a second SELECT to the original query, which lets an attacker read data from other tables. The first requirement for UNION is that both queries return the same number of columns. This lab's goal is to determine how many columns the original query behind the product category filter returns.

## Reconnaissance

Selecting a category loads a URL of the form `?category=Pets`. The value is placed into the query's `WHERE` clause with no sanitisation, so I can close the string and append my own SQL after it. This is the same injection point used by the other SQL injection labs on this site.

## Exploitation

Two independent methods, each converging on the same count.

### Method 1: ORDER BY

Increment the column index until the database errors:

```
?category=Pets' ORDER BY 1--
?category=Pets' ORDER BY 2--
?category=Pets' ORDER BY 3--
?category=Pets' ORDER BY 4--
```

`ORDER BY 1`, `2` and `3` returned the page normally. `ORDER BY 4` produced an "Internal Server Error". The database can only order by a column that exists, so the first index that errors is one past the real count. The query returns 3 columns.

A side signal worth noting: each `ORDER BY` re-sorted the product list by a different column. That re-sorting confirms the injected clause is actually being applied to the live query rather than ignored.

### Method 2: UNION SELECT NULL

Add NULLs until the UNION stops erroring:

```
?category=Pets' UNION SELECT NULL--
?category=Pets' UNION SELECT NULL,NULL--
?category=Pets' UNION SELECT NULL,NULL,NULL--
```

One and two NULLs errored. Three NULLs returned the page normally. A UNION requires matching column counts, so the number of NULLs that works equals the column count. `NULL` is used because it is compatible with any data type, which isolates the column-count question from column types. The count is 3, confirming Method 1.

## Impact

On its own, determining the column count is reconnaissance, not a breach. It is the first required measurement for a UNION attack. Once the count is known, the next steps become possible: finding a column that can hold text, then injecting a `UNION SELECT` against another table. That chain is how UNION-based SQL injection ends in reading arbitrary data such as usernames and passwords.

## Remediation

- **Parameterised queries (prepared statements).** Input is bound as a value and can never add an `ORDER BY`, a `UNION`, or a comment, so this probing cannot alter the query structure.
- **Do not expose database errors to the client.** The "Internal Server Error" is exactly what made `ORDER BY` enumeration easy. Generic error pages remove that oracle.
- Apply least privilege to the database account so that even a successful UNION reads as little as possible.

## Key takeaway

UNION needs both queries to return the same number of columns, so the attack begins by measuring that number. `ORDER BY` finds it through errors, and `UNION SELECT NULL` finds it through type-independent matching. Two methods, one answer, and the foundation for the rest of the UNION chain.
