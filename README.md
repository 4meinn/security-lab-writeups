# Security Lab Write-ups

Documented hands-on web security labs. For each finding I record my notes, the exploitation steps, and the fix. All work is performed in isolated, disposable training environments. No third-party or production systems are involved.

**GitHub:** [@4meinn](https://github.com/4meinn)

## Write-ups

| # | Vulnerability | Platform | Difficulty |
|---|---------------|----------|------------|
| 1 | [SQL Injection: WHERE clause, hidden data](portswigger/sql-injection/01-where-clause-retrieve-hidden-data.md) | PortSwigger Web Security Academy | ![Apprentice](assets/badges/apprentice.svg) |
| 2 | [SQL Injection: login bypass](portswigger/sql-injection/02-login-bypass.md) | PortSwigger Web Security Academy | ![Apprentice](assets/badges/apprentice.svg) |
| 3 | [SQL Injection: UNION attack, column count](portswigger/sql-injection/03-union-determine-columns.md) | PortSwigger Web Security Academy | ![Practitioner](assets/badges/practitioner.svg) |
| 4 | [SQL Injection: UNION attack, text column](portswigger/sql-injection/04-union-find-text-column.md) | PortSwigger Web Security Academy | ![Practitioner](assets/badges/practitioner.svg) |
| 5 | [SQL Injection: UNION attack, read another table](portswigger/sql-injection/05-union-retrieve-data.md) | PortSwigger Web Security Academy | ![Practitioner](assets/badges/practitioner.svg) |
| 6 | [SQL Injection: examining the database, listing contents](portswigger/sql-injection/06-examining-db-listing-contents.md) | PortSwigger Web Security Academy | ![Practitioner](assets/badges/practitioner.svg) |
| 7 | [SQL Injection: blind, time delays](portswigger/sql-injection/07-blind-time-delays.md) | PortSwigger Web Security Academy | ![Practitioner](assets/badges/practitioner.svg) |
| 8 | [SQL Injection: blind, conditional responses](portswigger/sql-injection/08-blind-conditional-responses.md) | PortSwigger Web Security Academy | ![Practitioner](assets/badges/practitioner.svg) |
| 9 | [SQL Injection: filter bypass via XML encoding](portswigger/sql-injection/09-filter-bypass-xml-encoding.md) | PortSwigger Web Security Academy | ![Practitioner](assets/badges/practitioner.svg) |
| 10 | [Access Control: unprotected admin functionality](portswigger/access-control/10-unprotected-admin-functionality.md) | PortSwigger Web Security Academy | ![Apprentice](assets/badges/apprentice.svg) |
| 11 | [Access Control: unprotected admin functionality with unpredictable URL](portswigger/access-control/11-unprotected-admin-functionality-unpredictable-url.md) | PortSwigger Web Security Academy | ![Apprentice](assets/badges/apprentice.svg) |
| 12 | [Access Control: user role controlled by request parameter](portswigger/access-control/12-user-role-controlled-by-request-parameter.md) | PortSwigger Web Security Academy | ![Apprentice](assets/badges/apprentice.svg) |
| 13 | [Access Control: user ID controlled by request parameter (IDOR)](portswigger/access-control/13-user-id-controlled-by-request-parameter.md) | PortSwigger Web Security Academy | ![Apprentice](assets/badges/apprentice.svg) |
| 14 | [Access Control: user ID controlled by request parameter, with unpredictable user IDs](portswigger/access-control/14-user-id-controlled-by-request-parameter-with-unpredictable-user-ids.md) | PortSwigger Web Security Academy | ![Apprentice](assets/badges/apprentice.svg) |
| 15 | [Access Control: user ID controlled by request parameter with data leakage in redirect](portswigger/access-control/15-user-id-controlled-by-request-parameter-with-data-leakage-in-redirect.md) | PortSwigger Web Security Academy | ![Apprentice](assets/badges/apprentice.svg) |
| 16 | [Access Control: user ID controlled by request parameter with password disclosure](portswigger/access-control/16-user-id-controlled-by-request-parameter-with-password-disclosure.md) | PortSwigger Web Security Academy | ![Apprentice](assets/badges/apprentice.svg) |
| 17 | [Access Control: URL-based access control can be circumvented](portswigger/access-control/17-url-based-access-control-can-be-circumvented.md) | PortSwigger Web Security Academy | ![Apprentice](assets/badges/apprentice.svg) |
| 18 | [Access Control: method-based access control can be circumvented](portswigger/access-control/18-method-based-access-control-can-be-circumvented.md) | PortSwigger Web Security Academy | ![Practitioner](assets/badges/practitioner.svg) |
| 19 | [Access Control: multi-step process with no access control on one step](portswigger/access-control/19-multi-step-process-with-no-access-control-on-one-step.md) | PortSwigger Web Security Academy | ![Practitioner](assets/badges/practitioner.svg) |
| 20 | [Access Control: user role can be modified in user profile](portswigger/access-control/20-user-role-can-be-modified-in-user-profile.md) | PortSwigger Web Security Academy | ![Apprentice](assets/badges/apprentice.svg) |
| 21 | [Access Control: insecure direct object references (IDOR)](portswigger/access-control/21-insecure-direct-object-references.md) | PortSwigger Web Security Academy | ![Apprentice](assets/badges/apprentice.svg) |
| 22 | [Access Control: Referer-based access control](portswigger/access-control/22-referer-based-access-control.md) | PortSwigger Web Security Academy | ![Practitioner](assets/badges/practitioner.svg) |
| 23 | [Cross-Site Scripting: reflected XSS into HTML context with nothing encoded](portswigger/cross-site-scripting/23-reflected-xss-html-context-nothing-encoded.md) | PortSwigger Web Security Academy | ![Apprentice](assets/badges/apprentice.svg) |
| 24 | [Cross-Site Scripting: stored XSS into HTML context with nothing encoded](portswigger/cross-site-scripting/24-stored-xss-html-context-nothing-encoded.md) | PortSwigger Web Security Academy | ![Apprentice](assets/badges/apprentice.svg) |
| 25 | [Cross-Site Scripting: DOM XSS in document.write sink using source location.search](portswigger/cross-site-scripting/25-dom-xss-document-write-sink-location-search.md) | PortSwigger Web Security Academy | ![Apprentice](assets/badges/apprentice.svg) |
| 26 | [Cross-Site Scripting: DOM XSS in innerHTML sink using source location.search](portswigger/cross-site-scripting/26-dom-xss-innerhtml-sink-location-search.md) | PortSwigger Web Security Academy | ![Apprentice](assets/badges/apprentice.svg) |
| 27 | [Cross-Site Scripting: DOM XSS in jQuery anchor href attribute sink using location.search source](portswigger/cross-site-scripting/27-dom-xss-jquery-href-attribute-sink-location-search.md) | PortSwigger Web Security Academy | ![Apprentice](assets/badges/apprentice.svg) |
| 28 | [Cross-Site Scripting: reflected XSS into attribute with angle brackets HTML-encoded](portswigger/cross-site-scripting/28-reflected-xss-attribute-angle-brackets-html-encoded.md) | PortSwigger Web Security Academy | ![Practitioner](assets/badges/practitioner.svg) |
| 29 | [Cross-Site Scripting: reflected XSS into a JavaScript string with angle brackets HTML encoded](portswigger/cross-site-scripting/29-reflected-xss-javascript-string-angle-brackets-html-encoded.md) | PortSwigger Web Security Academy | ![Practitioner](assets/badges/practitioner.svg) |
| 30 | [Cross-Site Scripting: reflected XSS into a JavaScript string with single quote and backslash escaped](portswigger/cross-site-scripting/30-reflected-xss-javascript-string-single-quote-backslash-escaped.md) | PortSwigger Web Security Academy | ![Practitioner](assets/badges/practitioner.svg) |

## Structure

Write-ups are grouped by platform and vulnerability class. Each one follows the same format: objective, reconnaissance, exploitation, impact, and remediation.

Difficulty badges follow the PortSwigger scale: ![Apprentice](assets/badges/apprentice.svg) ![Practitioner](assets/badges/practitioner.svg) ![Expert](assets/badges/expert.svg)
