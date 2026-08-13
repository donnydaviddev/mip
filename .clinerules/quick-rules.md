# Quick Rules — always in effect

Never do these (full reasoning in docs/05-project-rules.md, but you don't need to go read it — this list is the part that matters every task):
- Never touch password hashing/verification code — Better Auth owns that entirely, no exceptions
- Never `db.insert()` a user/account row directly — use `auth.api.signUpEmail()`
- Never use `@/...` path aliases anywhere under `src/db/schema/`, or in `drizzle.config.ts` — relative imports only, there
- Never create a `-new` / `-temp` / `-backup` / `-v2` / `test-` variant of an existing file — edit in place
- Money and percentages: always `NUMERIC`, never float/double
- Commission % is the plain number (5 means 5%) — never convert it to a decimal fraction

Standards: TypeScript strict, no `any`, no `as Type` (use `satisfies`), camelCase for vars/functions, kebab-case for filenames, Zod as the one source of validation truth.

You never need to read the docs/ folder — everything relevant to your task is already in the task prompt or these clinerules files. If you find yourself wanting to go explore docs/, that's a sign the task is missing something, not a reason to go look yourself — report it instead.

If your task doesn't clearly tell you which file, what's allowed to change, and what "done" looks like — stop and say so. Don't infer it and proceed.
