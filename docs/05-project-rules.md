# MIP — Project Rules

Replaces `05-handover-and-playbook.md` and the operational content of
`06-standards-workflow-reflections.md` entirely.

**Audience: Sergio, Tony(Planner), and L.** This is the master reference they
curate from — Sergio's Briefs and Tony(Planner)'s task prompts draw on this
file, but **Tony(Builder) does not load this whole document.** His
`.clinerules` gets a condensed digest instead (see `INFRA-SETUP.md` Part 1,
Step 3) — dumping all of this into his context every task is exactly the
overload this file exists to prevent. For the narrative reasoning behind
these rules, see `07-lessons-learned.md` — optional reading, this file isn't.

**Read `00`–`04` for what the app is. Read this for how to build it correctly.**

---

## 0. The one standard everything below serves

Every rule here exists for one reason: **Tony(Builder) ships correct, working
code, verified, not claimed.** If a rule ever gets in the way of that instead
of serving it, that's a bug in the rule — flag it, don't just follow it
anyway. Full team workflow (who curates what, who writes prompts, who's the
second opinion) is in `AI-TEAM.md` — this file assumes that context and
doesn't repeat it.

---

## 1. Every task needs a contract — no exceptions

Tony(Planner) writes this, informed by Sergio's Brief, and it lands in
`06-current-sprint.md`'s "Next Task" section before Tony(Builder) touches a
file:

- **Objective** — one sentence, one concrete outcome
- **Allowed to edit** — a short, named list of files
- **Forbidden to edit** — explicitly named, especially anything already
  confirmed working
- **Acceptance criteria** — a literal command whose real output must be
  shown. "I confirmed X" is not acceptance criteria. The actual output,
  pasted verbatim, is.
- **Max fix attempts: 2, total, across the whole escalation loop** — his own
  initial attempt is 1; the retry after Tony(Planner) re-scopes the prompt is
  2. Still broken after that → straight to L, not a third local guess. See
  `AI-TEAM.md`'s Workflow section for the full loop.

**Size it right, not small.** The goal is the largest coherent, verifiable
slice of real work Tony(Builder) can reliably execute in one pass — not the
smallest possible fragment. Split a task only when keeping it together would
genuinely risk ambiguity, hidden dependencies, or unreliable execution. If a
task is big enough that you can't fill in all four fields *cleanly and
unambiguously*, that's the actual signal to split it — size alone isn't.

**The loop doesn't end at "done."** Tony(Builder) reports back using a fixed
structure — COMPLETED / BLOCKED / FAILED, with an optional DISCOVERY
addendum on any of them, never a fourth status — full format in
`.clinerules/reporting-format.md`. Tony(Planner) treats that report as
new ground truth, not a checkbox — incorporating what actually happened,
including any DISCOVERY worth keeping, before defining the next task. See
`AI-TEAM.md`'s Workflow section for the complete loop.

## 2. Verification discipline

A "complete" or "confirmed" report is a claim, not a fact. Every meaningful
milestone gets independently checked — a direct database query, the actual
terminal output, an actual browser test — before anyone treats it as done.
This isn't optional under time pressure; it's the thing that's cheaper than
the alternative every time it's been skipped in this project's history.

When asking for verification: ask for the actual output, not a description of
the output. "The table has 3 rows" is not verification. The pasted result of
`SELECT * FROM users;` is.

## 3. Authentication — Intended Design

How auth is *meant* to work once finished. Cross-check `06-current-sprint.md`
before assuming any piece below is actually built.

**Login flow**: username + password via Better Auth's client library
(`authClient.signIn.username(...)`) → catch-all route at
`app/api/auth/[...all]/route.ts` → Better Auth verifies against its own
`account` table → session row created, cookie set → redirect by role:
Administrator/VP/Developer → `(app)`, Employee(OM) → `(om)`.

**Username**: lowercase letters + digits, 6–12 chars, globally unique. The
lowercase-only rule is NOT confirmed as natively enforced by Better Auth's
username plugin — verify, likely needs an explicit Zod check at account
creation.

**Password**: min 8 chars, 1 uppercase, 1 number, no special characters. Not
a Better Auth default — needs an explicit validation hook. Check Better
Auth's current docs for the exact mechanism rather than assuming a remembered
API shape (see Rule 6 below).

**Password hashing**: entirely Better Auth's responsibility. No application
code ever hashes, verifies, or touches a password directly.

**Sessions**: server-side, database-backed (the `session` table), not JWT.
Deliberate: this is what makes "revoke access" take effect *immediately* — a
JWT keeps working until its own expiry unless a separate blocklist exists.
Idle timeout: 30 minutes (confirmed business requirement, implementation
status unverified).

**Middleware**: every `(app)` request verifies role in
[administrator, vp, developer]; every `(om)` request verifies role =
employee. Unauthenticated → `/login`. Lives in Next.js middleware or a shared
layout check, never duplicated per-page.

**Role enforcement**: never in the UI alone. Every Service Layer function's
first line is a `requirePermission(user, action, resource)` guard.

**Why this specific approach**: self-hosted, no third-party custody of
payroll-adjacent data, and immediate revocation were explicit requirements
from the actual client — not general best-practice defaults. If a future
shortcut would trade away any of the three, that's a conversation, not a
quiet substitution.

## 4. Coding Standards

- **Naming**: camelCase (vars/functions/JS keys), PascalCase (components/types),
  snake_case (DB columns — Drizzle maps this automatically, don't hand-convert).
- **File naming**: kebab-case (`payroll-entry-service.ts`).
- **Components**: one per file. Shared types go in a dedicated types location.
- **Error handling**: Service Layer throws typed, specific errors — never a
  raw DB error to the UI. Routes/Server Actions translate to user-facing messages.
- **Validation**: Zod schemas, single source of truth, shared between
  client-side and Service Layer. Never duplicate a rule in two places.
- **Logging**: every Service Layer mutation emits an `audit_logs` entry —
  consistently, not piecemeal per feature.
- **TypeScript**: strict mode always. No `any` — use `unknown` and narrow. No
  `as Type` — use `satisfies`. Export explicit return types on exported
  functions.
- **Database**: money/percentages always `NUMERIC`, never float/double.
  Every table gets `created_at`; mutable tables get `updated_at`. FK columns
  named `<referenced_table_singular>_id`.

## 5. Folder Structure — Where New Code Goes

- New report type → `lib/services/reports/`, `components/reports/`, `app/(app)/reports/`.
- New Service Layer function → `lib/services/<business-domain>.ts`, grouped
  by domain, not CRUD verb.
- New Zod schema → `lib/validation/<domain>.ts`, mirroring the service it validates.
- New DB table → `src/db/schema/`, split by domain (e.g. `auth-schema.ts`,
  `business-schema.ts`) — this project already started with a domain-split
  folder rather than one giant file, so follow that pattern rather than the
  old "one file until unwieldy" default.
- New shadcn primitive → `pnpm dlx shadcn@latest add <component>` into
  `components/ui/`. Never hand-copy source instead of the CLI.
- New route needing its own layout → `(app)` vs `(om)` by which roles need
  it. Employee(OM)-only functionality never goes in `(app)`.

## 6. Mistakes Already Made — Do Not Reintroduce

1. **A hand-built `users` table with its own `password_hash` column.** Wrong.
   Better Auth owns its own tables entirely; extend via `additionalFields`.
2. **Hand-rolled password hashing** (e.g. argon2 directly). Wrong — causes a
   hash-format mismatch with Better Auth's own verification. Never hash or
   verify a password anywhere except through Better Auth.
3. **Raw `db.insert()` to create a user + account row.** Always use
   `auth.api.signUpEmail({ body: {...} })` server-side. Never a raw insert,
   even as a "temporary" fix:
   ```
   await auth.api.signUpEmail({
     body: {
       email: "placeholder@internal.local", // required by Better Auth's core
                                             // schema even though login is by
                                             // username — never used otherwise
       name: "Display Name",
       password: "...",
       username: "...",
       role: "developer",
     }
   });
   ```
4. **`@/...` path aliases inside files `drizzle-kit` itself loads**
   (everything under `src/db/schema/`, `drizzle.config.ts`, and anything they
   import — this applies to *every* file in the schema folder now that it's
   split by domain, not just one). `drizzle-kit` cannot resolve these. Use
   relative imports there. App code elsewhere (pages, components, routes)
   can use `@/...` freely.
5. **`drizzle.config.ts` not loading `.env.local`.** `drizzle-kit` doesn't
   auto-load it. Needs explicit `dotenv.config({ path: '.env.local' })` at
   the top of the config, and in any standalone script run outside Next's
   dev server.
6. **Orphaned duplicate file variants** (`schema-new.ts`, `-backup`, `-temp`,
   `test-*.ts`, two different `auth.ts` in different folders). **Never
   create a `-new` / `-temp` / `-backup` / `-v2` / `test-` variant of an
   existing file. Edit in place only.**
7. **`psql` commands hanging.** Always include `--pset pager=off`.

## 7. Immutable Decisions

Do not silently change these. If a future need conflicts with one, that's an
explicit conversation, not a place to quietly diverge:

- Revenue and commission % are entered per employee, per period — never a
  shared company-wide figure.
- Commission % is stored as the plain percentage number (0.05 means 0.05%),
  never a decimal fraction.
- Approve is VP-only. Administrator does not get this action.
- Two distinct, non-destructive undo actions exist (undo-payment,
  undo-approval) — neither ever deletes a record.
- Employees are never hidden or archived. Revoking login access never
  affects record visibility or history.
- Login is by username only — no email login, no OAuth, no self-service sign-up.
- Developer can change Administrator's password; Administrator cannot change
  Developer's. Administrator cannot see Developer's account/activity at all;
  Developer can see everyone's.
- No Bulk Import feature. Historical data arrives via a one-time seed script.
- Self-hosted deployment only, at callboxmanagers.com — **confirmed as of
  this doc's writing.** A free-tier host (e.g. Vercel Hobby) was evaluated
  and rejected: its Terms of Service prohibit commercial use, its free tier
  permits training on submitted content, and either conflicts with the
  client's explicit no-third-party-custody requirement for payroll data. If
  hosting comes up again, that's a real conversation, not a quiet swap.
- Historical data (2025–present) imports as **final-total-only** — no
  reconstructed revenue/percentage/bonus breakdown, since the source data
  doesn't have one. Full per-field computation starts fresh with the next
  new payroll month processed inside the app. Still open: exact
  department/branch split for combined-label source rows, and confirming
  "Admin & Support" as a real department to add to the seed list.

## 8. Safe Refactoring Areas

Free to improve without asking: UI styling/component structure, internal
`lib/services` organization (public behavior unchanged), adding tests, adding
logging, improving error messages.

Not safe without explicit confirmation: anything in Rule 7, anything
touching the auth/schema layer, anything touching money or percentage
calculation logic.

## 9. Testing, Review & Git

- **Testing**: Vitest for Service Layer unit tests — plentiful, especially
  around commission math and permission checks. Playwright for a small
  number of critical E2E flows (login, compute→review→approve→pay). Not
  every screen needs E2E.
- **You are the reviewer.** This is a solo-developer-plus-AI project — there
  is no second human. Reading every diff yourself before approving Act mode,
  and independently verifying acceptance criteria before trusting
  "complete," is not a nice-to-have layered on top of everything else here —
  it *is* the review process. Skipping it under time pressure is exactly
  what cost real time earlier in this project.
- **Independent, automated check**: GitHub Actions runs `tsc --noEmit` +
  `vitest run` on every push (see `INFRA-SETUP.md` Part 5). This catches a
  different failure mode than your own review — compile errors and test
  regressions, not "is this actually the right logic." It's a supplement to
  reading the diff, not a replacement for it.
- **Git**: one branch per task/slice, commit immediately after it passes its
  acceptance criteria. Never leave more than a day's work uncommitted — every
  commit is a fallback point.
