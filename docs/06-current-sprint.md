# MIP — Current Sprint

This is the one doc that changes constantly. Before each task, Sergio curates
a Brief and Tony(Planner) turns it into the "Next Task" section below, using
the contract format in `05-project-rules.md` Rule 1. Everything else on this
page only needs updating when it actually changes.

---

## Known State (last confirmed, not just claimed)

**Reset on [fill in today's date] — the project was rebuilt from a clean
slate.** Everything below reflects the fresh scaffold, not the earlier
in-progress auth work described in prior sessions — that work no longer
exists, it was intentionally wiped.

**Phase 0 (environment): complete, freshly verified.**
- Next.js 16 (App Router), TypeScript, Tailwind CSS v4, ESLint — `pnpm lint`
  and `pnpm build` both pass
- shadcn/ui initialized with the Nova preset, on **Radix UI** — switched from
  the Base UI default deliberately, since Tony(Builder)'s model has far more
  training exposure to Radix. Switch confirmed and build-verified.
- Docker Desktop + PostgreSQL 17 running via Docker Compose, confirmed
  healthy via `docker compose ps` and a direct `psql` connection — zero
  tables, which is expected
- `better-auth`, `drizzle-orm`, `postgres`, `zod`, `dotenv` installed;
  `drizzle-kit`, `tsx` installed as dev deps
- `drizzle.config.ts` and a `src/db/` structure exist; no schema written yet

**Phase 1 (auth): not started.** Nothing from the earlier session's partial
auth work survived the reset — no Better Auth configuration, no schema, no
routes, no login page. This is a real fresh start on Phase 1, not a resume.

**Known open item before real work starts**: the project uses `src/db/`,
`src/app/`, etc. (a `src/` root) — `01-architecture.md`'s folder tree didn't
originally show that prefix. Docs updated to match reality; flagging here so
it's not a silent surprise if you're comparing against an older read of that
file.

*(Update this section yourself the moment anything below changes — don't
leave it stale once you've actually verified something, the way this
section itself just was.)*

---

## Next Task

*Fill this in before every task, using the exact contract format from
`05-project-rules.md` Rule 1. Template below — delete the placeholder text,
don't leave it half-filled.*

**Objective**: [one sentence, one concrete outcome]

**Allowed to edit**: [named files only]

**Forbidden to edit**: [named files, especially anything in "Known State" above marked confirmed]

**Acceptance criteria**: [a literal command; its real output is what gets pasted back, not a description]

**Max fix attempts**: 2

---

## How Tony(Builder) Reports Back

Every task ends with one of three statuses — full format in
`.clinerules/reporting-format.md`, this is the short version:

- **COMPLETED** — a short report by default ("changed X, verified with Y,
  output: Z"). Tony(Planner) incorporates what actually happened before
  writing the next task — not just clearing a checkbox.
- **BLOCKED / FAILED**:
  1. Tony(Builder) tries one initial fix, or narrows the task to something
     he can finish.
  2. Still broken → he reports the *actual* error output to you.
  3. You relay it to Tony(Planner), who re-scopes the task above — smaller,
     clearer — preserving the original objective unless he explicitly
     decides the objective itself needs to change.
  4. Still broken after that → escalate to L with the real output.
  5. L hands back a "Summary for Tony(Builder)" — paste it straight into Cline.
  6. **Whatever L or Tony(Planner) changed that Tony(Builder) didn't make
     himself — state it explicitly in your next message to him.** He has no
     memory of edits he didn't personally make.

**DISCOVERY isn't a fourth status** — it's an optional addendum Tony(Builder)
can add to a COMPLETED, BLOCKED, or FAILED report when something's worth
flagging. See the log below.

Full reasoning behind this loop is in `AI-TEAM.md`.

---

## Discoveries Log

Findings Tony(Builder) or Tony(Planner) flagged as worth keeping
permanently, not yet promoted into `05-project-rules.md`. Tony(Planner)
proposes an addition in its own copy-paste block when this happens — you
review it, and once you've actually added it to the rules doc, move the
entry from here into a `✅ Promoted` note (or just delete the line) so this
list only ever shows what's still pending.

*(Empty — nothing logged yet. Add a line here the first time it happens
rather than waiting to remember it later.)*
