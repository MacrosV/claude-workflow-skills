# Serial Workflow Reference

Load this reference when extracting or resuming a serial sprint from `SPRINTS.md`.

## currentwork.md Template

When creating `currentwork.md` for serial mode, use this structure. Sections in `[brackets]` are instructions; replace them with actual content.

```markdown
# currentwork.md - Sprint [N]: [Title from SPRINTS.md]

**Status:** In Progress
**Started:** [YYYY-MM-DD]
**Source:** Sprint [N] in SPRINTS.md

---

## Prerequisites

Run these before writing any code. All must pass.

- [ ] `[test command - e.g., uv run pytest backend/tests/ -x]`
- [ ] `[any sprint-specific verification command]`

---

## Files to Read Before Starting

Read every file on this list completely before touching anything.

- `[/absolute/path/file.py]` - [what to understand from it]
- `[/absolute/path/component.jsx]` - [what to understand from it]

---

## Tasks

### Task [N.X] - [Title]

**Status:** [ ]

[Paste task spec from SPRINTS.md verbatim]

**Implementation notes:**
*(fill in after completing)*

**Acceptance criteria:**
- [ ] [criterion] - `[runnable command or [manual check] label]`

---

## Deviations Log

| Task | Planned | Actual | Reason |
|------|---------|--------|--------|

---

## Files Created

| Path | Purpose |
|------|---------|

---

## Files Modified

| Path | Lines | What changed |
|------|-------|--------------|

---

## Definition of Done

All must pass before write-back:

- [ ] `[primary test command]`
- [ ] `[build command]`
- [ ] `[any manual check with specific instructions]`

---

## Commit Readiness

- [ ] Generated files reviewed
- [ ] Ignored and untracked files checked
- [ ] Validation run and results recorded
- [ ] Docs/current-work/state files updated
- [ ] Release notes needed/not needed recorded
- [ ] Vendor/build/cache/local files excluded unless intentionally required

---

## Progress and Next Steps

**Plan progress:** Sprint [N] of [total] [complete/in progress] ([percent] by sprint count)
**Completed:** [sprints/tasks complete]
**Remaining:** [remaining sprints/tasks]
**Suggested next steps:**
- [next step]

---

## Write-back Status

- [ ] All tasks complete
- [ ] Definition of Done passes
- [ ] Implementation notes filled in for every task
- [ ] Deviations log complete
- [ ] Written back to SPRINTS.md
- [ ] Next sprint bootstrapped (or all sprints complete)
```
