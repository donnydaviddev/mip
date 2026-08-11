# MIP — Development Roadmap

Presented at the phase level — each phase gets detailed in full at the start of its own first
sprint, not all at once here. This is a current best estimate, not a fixed commitment; it will flex
based on how fast the real prompt → build → test → report cycle actually goes once we're in it.

| Phase | Est. days | Scope |
|---|---|---|
| 0 | 1–3 | Environment finalization + walking skeleton + dummy data seed |
| 1 | 4–8 | Auth (username-based, 4 roles) + Accounts tab (VP and OM sections, manual account creation only, Developer/Administrator credential hierarchy, Developer invisible to Administrator) + Profile + Sidebar shell |
| 2 | 9–13 | Employee Management: CRUD, search, sort, filter, profile page. No archive — every employee always visible |
| 3 | 14–19 | Commission Computation: per-employee revenue + manual commission %, entered and stored as a plain percentage number |
| 4 | 20–24 | Review workflow: Additional Bonus editing, progress tracking, change log |
| 5 | 25–28 | Approval + Payment: VP-only Approve, Administrator-only Mark Paid, both Undo Payment and Undo Approval |
| 6 | 29–32 | Reports + exports, plus the one-time historical data seed once the completed spreadsheet comes back |
| 7 | 33–36 | Dashboard: widgets, charts, currency display (always using each period's own stored rate) |
| 8 | 37–41 | Employee (OM) Self-Service Portal: separate navigation shell, own-payroll-history view only |
| 9 | 42–44 | Audit logs UI + hardening pass — the two boundaries worth testing hardest are VP-only Approve and Administrator's blindness to Developer's data |
| 10 | 45–46 | Deployment prep (self-hosted, callboxmanagers.com), documentation finalization, acceptance testing |

This table is scope and estimate only — it doesn't track what's actually
done. For real, currently-confirmed status, `docs/06-current-sprint.md` is
the one source of truth; keeping status in two places is exactly how a doc
goes stale without anyone noticing.

**Current estimate: ~45-46 days** of focused, most-days effort — not idle calendar time. The
biggest lever if this needs to compress is the Employee Self-Service Portal (Phase 8): it's the one
clean, self-contained piece that could move to a fast-follow release without touching anything
else, if needed.

**This estimate predates the current setup and hasn't been re-validated against it.** It was made
before Tony(Builder) was locked in as a 27B dense local model doing 100% of the implementation —
which is slower per-token than whatever this was originally paced against, in exchange for higher
per-task accuracy. Whether that nets out faster or slower overall isn't something to guess at; it
depends on real throughput once actual sprints are running. Re-check this table honestly after
Phase 1 actually finishes — if the real pace is meaningfully different, update it then rather than
carrying this number forward on faith.

## Notes
- The future OM calculator tab (self-verify their own commission) is not in any phase above —
  deliberately excluded, noted for later.
- No background job infrastructure is needed anywhere in this build at the confirmed scale.
