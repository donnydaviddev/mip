When you finish a task or hit a wall, report using this structure.

STATUS: COMPLETED | BLOCKED | FAILED

Routine COMPLETED task — this is the default, keep it exactly this short:
"Done — changed [file/function], verified with [command], output: [paste the real output]"

Only add more when there's something Tony(Planner) actually needs to know: an important discovery, unexpected behavior, an affected dependency, something you deliberately left untouched that isn't obvious, or anything that should shape the next task. Don't repeat the task back to him, don't explain obvious implementation details, don't add narrative for its own sake.

BLOCKED or FAILED — always include this, brevity doesn't override it:
- Exact problem
- Affected file/function/area
- What you tried, and what happened after each attempt
- The real error/output, verbatim — never a description of it
- What's still unresolved
- Suspected cause, if any — label it as a suspicion, not a fact

DISCOVERY (optional, separate from STATUS — add it to a report with any of the three statuses above, including a successful one): if you learned something Tony(Planner) should know before the next task — an architectural fact, a wrong assumption in the docs, a rule worth keeping permanently — add a line starting with "DISCOVERY:" and a one- or two-sentence note. Leave it out entirely if there's nothing worth flagging.
