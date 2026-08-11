# MIP — API Design

All routes versioned under `/api/v1/`. The web UI uses Server Actions for its own interactions;
these Route Handlers exist for any future external client. Both call the same Service Layer
functions — permission rules live in exactly one place regardless of entry point.

---

## Auth & credentials

| Method | Path | Description | Roles |
|---|---|---|---|
| POST | /api/v1/auth/login | Authenticate with username + password | Public (rate-limited) |
| POST | /api/v1/auth/logout | Destroy session | Authenticated |
| GET | /api/v1/me | Current user profile | Authenticated |
| PATCH | /api/v1/me/credentials | Change own username/password | Authenticated |

No email-based login, no self-service password reset — neither exists anywhere in this app.

## Account Management (Accounts tab — VP and OM sections)

| Method | Path | Description | Roles |
|---|---|---|---|
| GET | /api/v1/accounts | List accounts. **Administrator's results never include Developer accounts; Developer's results include everyone's** | Admin, Developer |
| POST | /api/v1/accounts | Create a VP or Employee account (system-generates the initial password, returned once in the response) | Admin, Developer |
| PATCH | /api/v1/accounts/:id/credentials | Change another user's username/password. **Administrator cannot target a Developer account; Developer can target anyone, including Administrator** | Admin, Developer |
| POST | /api/v1/accounts/:id/revoke-access | Deactivate a login (`users.is_active = false`) — does not touch the linked employee record | Admin, Developer |
| POST | /api/v1/accounts/:id/restore-access | Reactivate a login | Admin, Developer |

## Employees

| Method | Path | Description | Roles |
|---|---|---|---|
| GET | /api/v1/employees | List, search, filter, paginate — always includes every employee, active or not | Admin, VP |
| POST | /api/v1/employees | Create an employee record (without a login — use Account Management for that) | Admin |
| GET | /api/v1/employees/:id | Full profile + complete payroll history | Admin, VP |
| PATCH | /api/v1/employees/:id | Edit employee | Admin |

There is no archive/restore endpoint — employees are never hidden. Revoking access lives under
Account Management above, not here.

## Employee (OM) self-service

| Method | Path | Description | Roles |
|---|---|---|---|
| GET | /api/v1/me/employee | The logged-in OM's own employee record | Employee |
| GET | /api/v1/me/payroll-history | The logged-in OM's own payroll entries across all periods — resolved server-side from the session, never from a client-supplied id | Employee |

## Payroll periods & entries

| Method | Path | Description | Roles |
|---|---|---|---|
| GET / POST | /api/v1/payroll-periods | List / create a period (month, year, exchange rate) | Admin, VP (read) / Admin (write) |
| GET | /api/v1/payroll-periods/:id/entries | Employee queue for the period | Admin, VP |
| POST | /api/v1/payroll-entries | Compute an entry — revenue + commission % entered per employee | Admin |
| PATCH | /api/v1/payroll-entries/:id | Update a draft computation | Admin |
| POST | /api/v1/payroll-entries/:id/submit-review | Move to under_review | Admin |
| PATCH | /api/v1/payroll-entries/:id/bonus | Edit Additional Bonus (writes bonus_change_log) | VP |
| POST | /api/v1/payroll-entries/:id/approve | Approve | VP (Developer, for testing only) |
| POST | /api/v1/payroll-entries/:id/undo-approval | Approved for Release → For Review | Admin (Developer, for testing only) |
| POST | /api/v1/payroll-entries/:id/mark-paid | Record payment | Admin (Developer, for testing only) |
| POST | /api/v1/payments/:id/undo | Paid → Approved for Release, flag not delete | Admin (Developer, for testing only) |

## Reports & exports

| Method | Path | Description | Roles |
|---|---|---|---|
| GET | /api/v1/reports/monthly, /employee/:id, /year-to-date | Read-only reports | Admin, VP |
| GET | /api/v1/exports/:type | Generate xlsx/csv | Admin, VP |

## Dashboard

| Method | Path | Description | Roles |
|---|---|---|---|
| GET | /api/v1/dashboard | Aggregated widgets + chart data. PHP conversion always uses each period's own stored rate | Admin, VP |

## Audit logs

| Method | Path | Description | Roles |
|---|---|---|---|
| GET | /api/v1/audit-logs | Query the trail, filterable, paginated. **Administrator's results exclude any action performed by, or on, a Developer account; Developer's results include everything** | Admin, VP (read), Developer (full) |
