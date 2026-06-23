# Decisions

Decision memory for star-charter-pro (ADR-lite). Status: **Accepted · Open · Proposed · Superseded**.
Dates are absolute (YYYY-MM-DD).

---

## 0001 — Own standalone repo, split from the star-charter monorepo
**Status:** Accepted (2026-06-23)

**Context.** Star Charter Pro lived in a subfolder of the `star-charter` repo.

**Decision.** Promote it to its own repo (`star-charter-pro`) with the doc set. The
existing `DESIGN.md` is carried in as `docs/DESIGN.md` and remains the design source of truth.

**Consequence.** Clean one-game repo. `star-charter` retains the combined history.
