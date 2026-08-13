# MIP — Lessons Learned

Condensed from `06-standards-workflow-reflections.md`'s original Reflections
section and `05-handover-and-playbook.md`'s narrative. This is *why* the
rules in `05-project-rules.md` are what they are — useful for Sergio,
Tony(Planner), and L when they're making judgment calls, not something
Tony(Builder) needs loaded every task. Old persona names below (Skipper, the
coding agent generally) map onto Tony(Builder) now — the lessons transfer
directly, the names don't matter.

---

**Getting requirements straight from the actual business stakeholder, in
plain language, was the single highest-leverage thing that happened in this
project — more valuable than any code fix.** Several early assumptions
(revenue as one shared company-wide figure, a fixed commission-tier table,
the "Executive" role name) were reasonable guesses that turned out wrong once
the actual payroll manager answered directly. Every one, left unquestioned,
would have meant rebuilding a real piece of the system later. When a business
rule is ambiguous: asking is almost always cheaper than guessing wrong and
discovering it two phases later.

**Raw evidence beats a paraphrased status update, every time, with no
exceptions found so far.** Across a multi-day authentication debugging
stretch, a coding agent reported "complete" or "confirmed" several times when
the underlying database table was actually empty, or a password hash was in
the wrong format. This wasn't dishonesty — it's an optimistic
self-assessment failure mode that recurs regardless of how the task is
worded. The unlock, every time progress resumed, was insisting on pasted
terminal output or direct query results instead of a description of them.

**Verify a fast-moving library's current behavior before committing to it —
don't trust remembered or assumed API shape.** The worst repair loops in this
project trace directly to confident wrong assumptions about Better Auth's
schema ownership and drizzle-kit's path-alias handling. A short, targeted
search before locking in an architecture decision caught real errors before
they cost days, every time it was actually done. Skipping that check is what
let the same category of mistake (hand-rolling something a library already
owns) happen twice.

**Task size is a lever independent of task quality.** Well-organized,
clearly-numbered, thorough prompts still produced scope creep and cascading
failures when the underlying task was conceptually large ("implement
authentication" as one unit). What fixed it wasn't a better-written version
of the same-sized task — it was making tasks categorically smaller, with
named forbidden files. If an agent keeps expanding scope despite clear
instructions, the fix is shrinking the task, not writing sterner instructions
for the same task. This applies *more* to a local model than it did to
whatever ran this project before — smaller context, less headroom for
recovering from an oversized ask.

*(Later refinement, not a reversal: `05-project-rules.md` and `AI-TEAM.md`
now frame this as "largest coherent, verifiable slice," not "smaller is
always better." The failure documented above was tasks that were large *and*
vague — no file boundaries, no acceptance criteria, room to guess. A large
task that's still tightly bounded and unambiguous doesn't share that
problem. The actual lesson was never "small" — it was "unambiguous and
verifiable," and small just happened to be the easiest way to get there at
the time.)*

**Presenting early technical decisions as provisional, not final, made later
corrections feel like normal iteration instead of failure.** More than one
early decision (a framework version, a role name, a table-ownership
assumption) needed real revision — not from bad reasoning, but because the
ground was still moving. Framing a v1 decision honestly as "current best
understanding, subject to confirmation" made the eventual correction a quick
pivot rather than a trust-eroding reversal.

**Match pace to the person's actual state, not only to the technical
problem.** Partway through the authentication debugging, the binding
constraint stopped being purely technical and became "this person needs a
smaller, more certain win before another long diagnostic round." Shrinking
task scope at that point was as much about morale as architecture — and it
was still the technically correct call, not a compromise against it.

**Keeping a strict boundary between planning/verification and execution was
itself a useful check, not just an organizational preference.** This
project's structure explicitly separated "verify what the agent claims" from
"write the code" — that separation is precisely what caught the repeated
false-completion reports. A single actor both building something and
certifying it correct has an inherent blind spot; a second, independent
actor doesn't share that blind spot. Worth being precise about where this
actually lives in the current setup: Tony(Planner) and Tony(Builder) are the
*same* model family, so their split serves a different purpose (scoping and
prompt quality, not independence — see the task-size lesson above). The
actual blind-spot-independent check is **L**, deliberately a different model
on different infrastructure, and **you**, reading the diff yourself since
there's no second human on this project. That's not process for its own
sake — it's a specific failure mode this project already lived through once.

---

## The one-week authentication story, briefly

The original plan called for authentication as a single unit of work,
estimated at a few days. It took closer to a week — almost entirely because
early tasks were scoped as "implement authentication" rather than a sequence
of small, independently-verifiable slices. A capable coding model given a
broad task will reasonably interpret "fix the login" as license to touch the
schema, the config, the seed script, and the UI all at once — and when one of
those changes is subtly wrong, the failure surfaces somewhere else entirely,
which then gets "fixed" by touching yet more files. This compounds fast. Two
changes broke the cycle: shrinking task scope to single, named files with
explicit forbidden lists, and refusing to treat a written summary as verified
truth until an independent check confirmed it. Both are standard practice
now, not exceptions — see `05-project-rules.md` Rules 1 and 2.
