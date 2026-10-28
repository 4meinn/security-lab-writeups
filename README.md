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

## Structure

Write-ups are grouped by platform and vulnerability class. Each one follows the same format: objective, reconnaissance, exploitation, impact, and remediation.

Difficulty badges follow the PortSwigger scale: ![Apprentice](assets/badges/apprentice.svg) ![Practitioner](assets/badges/practitioner.svg) ![Expert](assets/badges/expert.svg)
