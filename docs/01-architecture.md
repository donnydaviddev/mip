# MIP — Architecture

---

## Technology Stack

| Layer | Choice | Why |
|---|---|---|
| Language | TypeScript, strict mode | A wrong field type here is a payroll bug, not a cosmetic one |
| Framework | Next.js 16 (App Router) | Unifies UI + backend; Server Actions remove CRUD boilerplate; Server Components suit data-heavy dashboards. Node 20.9+ required, Turbopack is the default bundler |
| Database | PostgreSQL 17 | ACID guarantees and exact `NUMERIC` currency math, both non-negotiable for payroll data |
| ORM | Drizzle ORM | SQL-first TypeScript schema — plays well with AI coding agents, and gives fine-grained control over `NUMERIC`/`CHECK` constraints |
| Auth | Better Auth, with its official `username` plugin | Self-hosted, sessions in your own Postgres, immediate revocation, and — verified directly — supports username-only login with email fully optional. No per-user cost, no third-party custody of payroll-adjacent data |
| Styling | Tailwind CSS v4 + shadcn/ui (Nova preset, Lucide icons), on **Radix UI** as the underlying primitive library — deliberately chosen over shadcn's July 2026 Base UI default, specifically because Tony(Builder)'s model has far more training exposure to Radix's well-established API. Confirmed switched and build-verified. | Matches the Stripe/Linear/Vercel design direction; already scaffolded |
| Charts | Recharts | Sufficient for the confirmed dashboard chart set |
| Animation | Framer Motion | Used to support usability, not decorate |
| Exports | ExcelJS | Styled, branded `.xlsx`/`.csv` report exports (company name, period, generated-by) |
| Validation | Zod | One schema shared between form validation and Service Layer input validation |
| Testing | Vitest + Playwright | Unit/integration and end-to-end |
| Local DB | PostgreSQL via Docker Compose | Already running from Phase 0 |
| Hosting | Self-hosted, production domain callboxmanagers.com | A Docker Compose stack (app + Postgres) behind a reverse proxy (Caddy is the simplest option for automatic HTTPS) on a VPS or existing company server. This is a deployment-phase decision, not needed until the app is ready to ship. **Confirmed, not just default** — see `05-project-rules.md` Rule 7 for why a free-tier host was evaluated and rejected |

## Development Machine

Local development and the AI coding workflow (see `AI-TEAM.md` and
`INFRA-SETUP.md`) run on an M5 MacBook Pro, 32GB unified memory, 1TB storage.
This is separate from the production hosting decision above — nothing about
the dev machine constrains or implies anything about where MIP is deployed.

## Auth Configuration Shape

Confirmed and verified feasible — Better Auth's username plugin, roughly:

```
betterAuth({
  emailAndPassword: { enabled: true },   // required by the plugin; email itself stays optional
  plugins: [
    username({ minUsernameLength: 6, maxUsernameLength: 12 })
  ]
})
```

No transactional email service anywhere in this application — no password-reset email, no
email-based login, no verification email.

## High-Level Architecture

Four layers, strictly one-directional dependency:

```
Presentation           Next.js App Router pages, Server Components, Server Actions
      |
Service Layer          Framework-agnostic business logic — lib/services/*
      |
Data Access Layer       Drizzle queries — src/db/*
      |
PostgreSQL
```

Both Server Actions (the web UI) and versioned Route Handlers (`/api/v1/*`, for any future
external client) call into the same Service Layer functions — business logic is never duplicated
or re-implemented per entry point. Every permission rule in this project — VP-only Approve,
Developer's full access, Administrator's blindness to Developer's data, OM scoped to only their
own record — is enforced inside the Service Layer function itself, never in a route or the UI.
Hidden buttons are not security.

## Folder Structure

**Confirmed against the actual scaffold**: this project uses a `src/` root
(`src/app/`, `src/db/`, etc.), not a bare `app/`/`lib/` root at the project
top level. If you're reading an older copy of this doc that shows no `src/`
prefix, this version is the corrected one.

```
mip/
  src/
    app/
      (auth)/login/
      (app)/                          Administrator, VP, Developer
        dashboard/ employees/[id]/ commission/ review/ approval/ payment/
        reports/ accounts/            NEW — VP & OM account management, two sections
        profile/ audit-logs/
      (om)/                           Employee(OM) — separate, smaller shell
        my-commissions/
        profile/
      api/v1/
        auth/ employees/ payroll-periods/ payroll-entries/
        payments/ reports/ exports/ accounts/ audit-logs/ dashboard/
    components/
      ui/  layout/  employees/ commission/ review/ reports/ dashboard/ accounts/
      om/                             OM's restricted view components
    db/
      schema/                        Split by domain, not one giant schema.ts —
                                      e.g. auth-schema.ts, business-schema.ts
      migrations/
      index.ts                       Drizzle client
    lib/
      services/  auth/  validation/  utils/
  docs/
  tests/
  .clinerules/
  .env.example
  docker-compose.yml
```

Note the Data Access Layer lives at `src/db/`, not `lib/db/` as an earlier
version of this doc showed — `lib/` holds the framework-agnostic business
logic (services, validation, utils), `src/db/` holds the Drizzle schema and
client specifically, matching what's actually on disk.

The `(om)` route group is deliberately separate from `(app)` — a different, much smaller
navigation shell, not a cut-down version of the admin sidebar. This makes "OM cannot even
navigate toward anything beyond their own data" true at the routing level, not just a permission
check that could be missed somewhere.

## Environment Variables

```
DATABASE_URL=
BETTER_AUTH_SECRET=
BETTER_AUTH_URL=
NODE_ENV=
ENABLE_DEVELOPER_ROLE=          # false in production
SESSION_IDLE_TIMEOUT_MINUTES=30
LOGIN_MAX_ATTEMPTS=             # proposed default 5
LOGIN_LOCKOUT_MINUTES=          # proposed default 15
```

Real values live only in your local `.env.local` (already gitignored) and, later, your server's
own environment — never in chat, never committed.
