# Wave Planning Reference

Load this reference when creating, converting, extracting, dispatching, or integrating parallel wave plans.

## Wave Plan Format

`IMPLEMENTATION_PLAN.md` is a parallel-friendly planning document. It preserves planned intent and should not be rewritten into an implementation record. Completed reality belongs in `SOFTWARE_STATE.md` or `COMPLETED_SPRINTS.md`.

```markdown
## Wave N: Title

**Goal:** One sentence describing the integrated milestone this wave produces.
**Parallelizable:** yes
**Integration gate:** `runnable command`

### Sprint NA: Packet Title

**Goal:** One sentence describing what this packet produces.

**Writes:**
- `path/or/glob/**` - ownership boundary

**Reads:**
- `path/file.ext` - context needed

**Depends on:** none
**Blocks:** Sprint NB

**Tasks:**
- [ ] task description

**Acceptance criteria:**
- [ ] criterion - `runnable command`

**Out of scope:**
What this packet must not change.
```

## Companion Documents

| File | Purpose |
|------|---------|
| `IMPLEMENTATION_PLAN.md` | Original plan: waves, sprint packets, dependencies, ownership, acceptance criteria |
| `currentwork.md` | Coordinator-owned active wave index or work queue |
| `currentwork/sprint-NX.md` | Extracted work packet for one agent |
| `currentwork/results/NX.md` | Completion report written by one agent |
| `SOFTWARE_STATE.md` | Current implemented reality after each completed wave |
| `COMPLETED_SPRINTS.md` | Optional append-only implementation history |

For blocked/suspended work or multiple unrelated active sprints, use `references/work-queue.md` alongside this reference.

## Wave currentwork.md Template

```markdown
# currentwork.md - Wave [N]: [Title]

**Status:** In Progress
**Started:** [YYYY-MM-DD]
**Source:** Wave [N] in IMPLEMENTATION_PLAN.md
**Software state source:** SOFTWARE_STATE.md

---

## Wave Goal

[Goal copied from IMPLEMENTATION_PLAN.md]

---

## Workflow Mode

**Mode:** Lightweight serial | Parallel dispatch
**Reason:** [why this wave is serial or parallel]
**Coordinator:** [main agent/session]

---

## Active Sprint Packets

| Packet | Owner | Role | Tier | Selection reason | Status | Writes | Packet File | Result File |
|--------|-------|------|------|------------------|--------|--------|-------------|-------------|
| Sprint [NA] | [agent/local] | worker | medium | Clear owned implementation with tests | Ready | `[path/**]` | `currentwork/sprint-[NA].md` | `currentwork/results/[NA].md` |

---

## Dependency Order

| Packet | Depends on | Blocks |
|--------|------------|--------|
| Sprint [NA] | none | Sprint [NB] |

---

## Ownership Conflicts

| Paths | Packets | Resolution |
|-------|---------|------------|

---

## Integration Gate

All must pass after packet completion:

- [ ] `[wave-level test command]`
- [ ] `[build command]`
- [ ] `[manual check: specific instructions]`

---

## Wave Status

- [ ] Wave quality checks passed
- [ ] Packets extracted
- [ ] Packets dispatched or selected for local serial execution
- [ ] Packet completion reports received
- [ ] Packet acceptance criteria pass
- [ ] Integration gate passes
- [ ] SOFTWARE_STATE.md updated
- [ ] Next wave prepared or all waves complete

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

**Plan progress:** Wave [N] of [total] [complete/in progress] ([percent] by wave count)
**Completed:** [waves/sprints/tasks complete]
**Remaining:** [remaining waves/sprints/tasks]
**Suggested next steps:**
- [next step]
```

For lightweight serial waves, use the same template but set `Mode: Lightweight serial`, keep `Owner` as `local`, and omit `currentwork/sprint-NX.md` packet extraction unless the work grows enough to need packet files.

## Sprint Packet Template

Each `currentwork/sprint-NX.md` file must be self-contained enough for one agent to execute without reading unrelated plan sections.

```markdown
# Sprint [NX]: [Title]

**Status:** Ready
**Source:** Wave [N] in IMPLEMENTATION_PLAN.md
**Result file:** currentwork/results/[NX].md

## Goal

[Goal copied from plan]

## Write Ownership

Only modify these paths:

- `[path/**]` - [reason]

## Read-Only Context

Read these before editing:

- `[path/file.ext]` - [what to understand]

## Dependencies

**Depends on:** [none or packet ids]
**Blocks:** [none or packet ids]

## Tasks

- [ ] [task]

## Acceptance Criteria

- [ ] [criterion] - `[runnable command or manual check]`

## Out of Scope

[Explicit exclusions]

## Completion Report Requirements

Write `currentwork/results/[NX].md` with:

- Summary of implemented behavior
- Files created
- Files modified with line references
- Acceptance commands run and results
- Deviations from plan
- Integration notes or blockers
```

## Completion Report Template

Sub-agents write one completion report each. The coordinator is the only writer to shared state documents.

```markdown
# Sprint [NX] Completion Report

**Status:** Complete | Blocked
**Agent:** [agent id/name if available]
**Completed:** [YYYY-MM-DD]

## Implemented Behavior

[What now works]

## Files Created

| Path | Purpose |
|------|---------|

## Files Modified

| Path | Lines | What changed |
|------|-------|--------------|

## Acceptance Results

| Criterion | Command | Result |
|-----------|---------|--------|

## Deviations

| Planned | Actual | Reason |
|---------|--------|--------|

## Integration Notes

[Contracts, assumptions, follow-up integration needs, or blockers]
```

## Sub-Agent Orchestration

The coordinator owns planning, dispatch, integration, and shared state updates. Sub-agents own only their assigned packet's write paths and completion report.

There is one coordinator-owned `currentwork.md` per active wave. Sub-agents do not create separate current-work documents. They receive packet files (`currentwork/sprint-NX.md`) and write completion reports (`currentwork/results/NX.md`); the coordinator merges those reports into `currentwork.md`, `SOFTWARE_STATE.md`, and optional history documents.

If a packet blocks while other unrelated packets remain ready, suspend the blocked packet using `references/work-queue.md`, then continue with another ready packet only after checking write ownership and local change state.

### Dispatch Preconditions

Dispatch a packet only when all are true:

- Dependencies are complete or explicitly mocked by a stable contract.
- `Writes:` is explicit and does not overlap another active packet.
- Shared files are owned by one packet or reserved for coordinator integration.
- Acceptance criteria are runnable or have a specific manual check.
- The packet has a unique result file.

If any condition fails, keep the packet local, split it, move it to a later wave, or mark it serial within the wave.

### Agent Role Selection

Choose the least expensive capable agent for each packet. Do not assign a frontier model to routine or mechanical work unless the packet has genuine ambiguity, broad architectural impact, or high integration risk.

| Packet type | Preferred role | Reasoning effort/model tier |
|-------------|----------------|-----------------------------|
| Codebase reconnaissance, dependency mapping, file ownership audit | explorer | low or medium |
| Isolated documentation, config, fixture, snapshot, or simple test edits | worker | low |
| Narrow implementation with clear acceptance criteria and owned files | worker | medium |
| Cross-module implementation, migration work, public API design, concurrency, data model changes | worker | high or frontier-capable |
| Integration failure diagnosis after multiple packets complete | worker or coordinator-local | high |
| Final architecture/state synthesis | coordinator-local | medium or high |

When runtime-specific roles are available, prefer:

- `explorer` for read-only codebase questions.
- `worker` for bounded implementation packets.
- coordinator-local work for plan conversion, dispatch decisions, integration gates, and shared document updates.

### Model Selection Rules

Use these rules when the runtime allows choosing models or reasoning effort:

1. Start with the cheapest/fastest model tier that can satisfy the packet.
2. Increase capability only for ambiguity, large context requirements, cross-module design, brittle migrations, security-sensitive work, or repeated failure.
3. Keep trivial packets on low reasoning effort.
4. Use medium effort for normal implementation.
5. Use high effort for public contracts, hard debugging, wave integration, and plan conversion.
6. Avoid overriding the runtime's inherited/default model unless the packet clearly needs a different tier or the user explicitly requests one.

Record the selected role and tier in `currentwork.md` before dispatch:

```markdown
| Packet | Owner | Role | Tier | Selection reason |
|--------|-------|------|------|------------------|
| Sprint 2B | agent | worker | medium | Clear owned implementation with tests |
```

### Claude Code Dispatch Mapping

When dispatching from Claude Code, translate the generic role and tier to `Agent` tool parameters:

| Role | `subagent_type` | Notes |
|------|----------------|-------|
| explorer | `"Explore"` | Read-only tools only; use for reconnaissance, dependency mapping, file ownership audits |
| worker | `"general-purpose"` | Full tool access; use for all implementation packets regardless of tier |
| coordinator-local | *(no spawn)* | Main agent continues; plan conversion, dispatch decisions, integration gates, shared state updates |

Map tier to the `model` parameter on the `Agent` tool call:

| Tier | `model` value | Typical packets |
|------|--------------|-----------------|
| low | `"haiku"` | Mechanical edits, config changes, simple fixtures, snapshot updates |
| medium | `"sonnet"` | Normal bounded implementation with clear acceptance criteria |
| high / frontier | `"opus"` | Cross-module design, hard debugging, public API decisions, security-sensitive work |

Omit `model` to inherit the runtime default. Override only when the packet clearly needs a different tier than the default.

**User-specified model cap:** Users can constrain the maximum model tier the coordinator assigns to sub-agents. Accept this as a session instruction (e.g., "use at most Sonnet for all sub-agents") or via `CLAUDE.md`. When a cap is set, record it in `currentwork.md` before the first dispatch and apply it to every packet selection. This cap is enforced by the coordinator following the guidance — Claude Code has no runtime mechanism that prevents the `Agent` tool from being called with a higher-tier `model` value.

### Sub-Agent Prompt Requirements

Each dispatched sub-agent must receive:

- Packet file path.
- Owned write paths.
- Read-only paths.
- Result file path.
- Acceptance criteria.
- Explicit instruction that other agents may be editing the codebase.
- Explicit instruction not to revert unrelated changes or edit shared state documents.

Minimal dispatch prompt:

```text
You own currentwork/sprint-[NX].md. Read it completely, then implement only the declared Write Ownership paths. Other agents may be editing the codebase; do not revert their work. Do not edit currentwork.md, SOFTWARE_STATE.md, COMPLETED_SPRINTS.md, or IMPLEMENTATION_PLAN.md. Run the packet acceptance criteria and write your completion report to currentwork/results/[NX].md.
```

## Wave Quality Checks

### Parallel safety check

| Check | How to verify |
|-------|---------------|
| Write sets are explicit | Every packet has a `Writes:` block |
| Write sets do not overlap | No two packets own the same path, glob, generated file, migration, route, schema, or shared config |
| Shared files are isolated | Shared files such as lockfiles, route registries, generated indexes, and config are assigned to one packet or reserved for coordinator integration |
| Read/write boundaries are clear | Paths in `Reads:` are read-only unless also listed under that packet's `Writes:` |
| Dependencies are acyclic | No packet depends on a packet that directly or indirectly depends on it |

### Packet completeness check

| Check | Minimum |
|-------|---------|
| Goal | One concrete behavior or artifact |
| Tasks | Enough detail for an isolated agent to implement without reading other wave sections |
| Acceptance criteria | Runnable command or explicit `[manual check: description]` |
| Result file | Unique `currentwork/results/NX.md` path |
| Out of scope | Explicitly states what the packet must not touch |

### Integration check

| Check | Minimum |
|-------|---------|
| Wave gate | At least one command or manual check validates cross-packet integration |
| State update target | `SOFTWARE_STATE.md` or `COMPLETED_SPRINTS.md` identified |
| Coordinator ownership | Shared state documents are updated only by the coordinator |

If any parallel safety check fails, split the conflicting packet, move shared-file edits into a dedicated integration packet, mark conflicting packets as serial, or move dependent work to a later wave.

## Lightweight Serial Wave Checklist

Use this instead of parallel dispatch for small or coupled waves:

- [ ] Current-work document identifies the active wave and mode.
- [ ] Tasks are ordered by dependency.
- [ ] Write ownership notes identify shared or coupled files.
- [ ] Validation commands are listed.
- [ ] Non-blocking validation issues are explicitly labeled.
- [ ] Deferred work is recorded.
- [ ] Commit readiness is recorded before handoff.
- [ ] Progress and suggested next steps are included in the completion summary.

Do not create sub-agent packets for routine one-agent work unless the wave grows or write ownership becomes separable.

## Serial to Wave Conversion

Use this when the user asks to convert a serial `SPRINTS.md` plan into a parallel wave plan. This can be done in planning mode without implementing code.

1. Read the full serial plan and identify each sprint's goal, expected files, dependencies, and acceptance criteria.
2. Build a dependency graph from explicit references, likely API/data dependencies, test dependencies, and shared-file ownership.
3. Assign each sprint or task to a packet with explicit `Writes:` and `Reads:` blocks.
4. Batch packets into waves:
   - Wave 1 establishes shared contracts, project skeletons, schemas, interfaces, and test harnesses.
   - Later waves contain feature work that depends only on completed prior waves.
   - Packets in the same wave must have non-overlapping write sets.
5. Add an integration gate to every wave.
6. Write the result to `IMPLEMENTATION_PLAN.md`.
7. Preserve `SPRINTS.md` as the original serial source unless the user asks to replace it.

## Batching Rules

| Situation | Placement |
|-----------|-----------|
| Packet defines public contracts used by others | Earlier wave |
| Packet consumes public contracts but owns separate implementation files | Later wave, parallel with other consumers |
| Packets touch the same files | Same wave only if one is read-only; otherwise split or serialize |
| Packet requires generated code, migrations, or lockfile changes | Give ownership to one packet or coordinator integration |
| Acceptance requires end-to-end behavior across packets | Wave integration gate, not individual packet only |

Do not invent false independence. If the serial plan is tightly coupled, produce fewer packets or explicitly mark a wave as serial.
