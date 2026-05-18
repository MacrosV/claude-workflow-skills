# Work Queue Reference

Load this reference when a sprint is blocked, when switching between unrelated sprints, or when multiple agents/sessions need to coordinate active work.

## Core Model

`currentwork.md` can be either a single active sprint document or a work queue index. Use queue mode when more than one sprint packet may be active, blocked, suspended, ready, or assigned.

```text
currentwork.md                  # coordinator-owned work queue index
currentwork/sprint-N.md          # resumable sprint packet
currentwork/results/sprint-N.md  # optional completion/checkpoint report
```

The coordinator owns `currentwork.md`. Individual agents or sessions own only their assigned packet files and result reports.

## Work Queue Template

```markdown
# currentwork.md - Work Queue

**Status:** Active
**Coordinator:** [main agent/session]
**Updated:** [YYYY-MM-DD]
**Plan source:** [SPRINTS.md or IMPLEMENTATION_PLAN.md]

---

## Active Work Queue

| Sprint | Status | Owner | Writes | Packet File | Result File | Blocker/Reason | Resume Condition | Next Action |
|--------|--------|-------|--------|-------------|-------------|----------------|------------------|-------------|
| Sprint A | Blocked | local | `path/**` | `currentwork/sprint-A.md` | `currentwork/results/sprint-A.md` | Awaiting API review | Feedback received | Resume task A.3 |
| Sprint B | Ready | local | `other/**` | `currentwork/sprint-B.md` | `currentwork/results/sprint-B.md` | none | n/a | Start task B.1 |

---

## Coordination Rules

- [ ] Only one owner edits each write set at a time.
- [ ] Blocked/suspended packets have resume conditions.
- [ ] Concurrent packets have non-overlapping write sets or isolated worktrees/branches.
- [ ] Shared files are assigned to one owner or reserved for integration.
- [ ] Integration owner is identified when concurrent work is active.

---

## Progress and Next Steps

**Plan progress:** [Sprint/Wave count and approximate percent]
**Blocked:** [blocked packets and reasons]
**Ready:** [ready packets]
**In progress:** [active packets]
**Suggested next steps:**
- [next action]
```

## Suspend and Switch

Use this when the current sprint cannot advance because of external feedback, product decisions, review, blocked validation, or unavailable dependencies.

1. Update the active packet with current task status, files touched, validation run, deviations, and exact resume condition.
2. Mark the packet `Blocked` or `Suspended`.
3. Record blocker owner when known, such as "awaiting API review from Devon".
4. Record whether local edits are committed, stashed, branched, or left in the working tree.
5. Update `currentwork.md` work queue with status, blocker, resume condition, and next action.
6. Select the next ready sprint only if its write set is safe relative to pending edits.
7. If write sets overlap, either wait, commit/stash/shelve the blocked work, or use a separate branch/worktree.

## Resume Blocked Work

1. Read `currentwork.md` and the blocked packet.
2. Confirm the resume condition is satisfied.
3. Inspect current git status and files touched since suspension.
4. Re-read files listed in the packet that may have changed while blocked.
5. Continue from the first incomplete task.
6. Update the queue status from `Blocked` or `Suspended` to `In Progress`.

## Single Agent vs Concurrent Agents

The same queue document supports both cases, but apply different safety rules.

| Situation | Primary concern | Required checks |
|-----------|-----------------|-----------------|
| Single agent switching away from blocked work | Resume quality | blocker, resume condition, validation state, files touched |
| Multiple agents/sessions working at once | Coordination safety | non-overlapping write sets, owner per packet, integration owner, branch/worktree strategy |

For single-agent switching, overlapping write sets can be acceptable if no one else is editing them and local changes are controlled. For concurrent agents or developers, overlapping write sets should be avoided unless work is isolated by branches/worktrees and integrated deliberately.

## Packet Checkpoint

Add or update this section in any blocked/suspended packet:

```markdown
## Checkpoint

**Status:** Blocked | Suspended
**Blocked by:** [person/system/decision]
**Resume condition:** [specific event required]
**Local change state:** [committed/stashed/uncommitted/worktree/branch]
**Last validation:** [command and result]
**Files touched:** [paths]
**Next task:** [exact next action]
**Risk on resume:** [merge risk, stale context, changed dependency, none]
```
