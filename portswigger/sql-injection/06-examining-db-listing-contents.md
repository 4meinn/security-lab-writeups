# SQL Injection: Listing the Database Contents (information_schema)

- **Lab:** SQL injection attack, listing the database contents on non-Oracle databases
- **Platform:** PortSwigger Web Security Academy
- **Category:** SQL Injection (OWASP A03:2021, Injection)
- **Difficulty:** Practitioner
- **URL:** https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-non-oracle

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

The product category filter is injectable and the query results are reflected, so a UNION attack can read other data. Unlike the earlier UNION labs, the table and column names are not given. The goal is to discover them from the database catalog, extract the administrator's password, and log in.

## Reconnaissance

Same injection point as the other labs, the category GET parameter. This is a PostgreSQL (non-Oracle) database, so its catalog lives in `information_schema`.

## Exploitation

### Step 1: column count and text columns

```
?category=Gifts' UNION SELECT NULL,NULL--
?category=Gifts' UNION SELECT 'a','b'--
```

Two NULLs returned the page, so the query returns 2 columns, and both displayed a string. This step is about the query's output columns, not about tables. A UNION must return the same number of columns as the original query, and the data I later pull has to land in a column that renders text. It is the same prerequisite as any UNION attack.

### Step 2: list the tables

```
?category=Gifts' UNION SELECT table_name, NULL FROM information_schema.tables--
```

`information_schema.tables` returns every table the database account can see. The list is long and mixes two very different things: the database's own built-in catalog (dozens of `pg_*` tables and `information_schema` views) and the application's own tables. The application table stood out by its randomized name: `users_zwymxi`.

### Step 3 (wrong turn): the system catalog is a trap

Filtering for "user" first pointed me at `pg_user`, a built-in PostgreSQL view of database roles with columns `usename` and `passwd`. It looked perfect:

```
?category=Gifts' UNION SELECT usename, passwd FROM pg_user--
```

The result was:

```
peter        ********
postgres     ********
```

Those are real database roles, but PostgreSQL masks their passwords as `********` in `pg_user` by design, so this is a dead end. The lesson: `information_schema.tables` lists both the DBMS's own system catalog and the application's data. The credentials I wanted live in the application's table, not in the database's role catalog.

### Step 4: columns of the real table

Back to the application table from Step 2:

```
?category=Gifts' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users_zwymxi'--
```

The columns were also randomized: `username_ahlwfd` and `password_zfpfxe`.

### Step 5: extract and log in

```
?category=Gifts' UNION SELECT username_ahlwfd, password_zfpfxe FROM users_zwymxi--
```

Returned:

```
administrator    dovcwvqi2s9os6h0z1ic
```

Logging in as `administrator` with that password solved the lab. The value is from a disposable lab instance and is already dead.

## Impact

With no prior knowledge of the schema, an unauthenticated visitor mapped the database from its own catalog, located the application's user table, and extracted the administrator credentials. Discovery plus extraction is the full real-world chain. In practice you are never handed the table and column names, so reading them from `information_schema` is the first move.

## Remediation

- **Parameterised queries (prepared statements).** Bound input cannot run a UNION against the catalog or any table.
- **Least privilege.** The web application's database account should not be able to read the whole schema or tables it never needs. A tightly scoped account cannot enumerate `information_schema` freely.
- **Store passwords as salted hashes, never in plaintext.**

## Key takeaway

When the schema is unknown, the database describes itself: `information_schema.tables` lists the tables and `information_schema.columns` lists their columns. The trap is that this catalog also exposes the DBMS's own system tables (`pg_user` and the other `pg_*` views), which look like the target but hold masked or irrelevant data. The real credentials are in the application's own table, identifiable here by its randomized name. Map first, then extract.
