---
name: sprint-workflow
description: Sprint extraction, tracking, write-back, and parallel wave planning for AI-assisted development. Use when working with SPRINTS.md, IMPLEMENTATION_PLAN.md, currentwork.md, serial sprint execution, parallel breakout sprints, wave plans, sub-agent coordination, implementation records, or converting a serial plan into batched parallel work. Also activates when the user mentions "extract sprint", "start sprint", "write back", "next sprint", "currentwork.md", "wave plan", "parallel sprints", or "breakout agents".
---

# Sprint Workflow

<skill_scope skill="sprint-workflow">
This skill provides the process and quality framework for implementing software in discrete, context-isolated plan sections — called **sprints** — using structured planning and working documents. It supports two modes:

- **Serial mode:** process one sprint at a time from `SPRINTS.md` using `currentwork.md`.
- **Wave-plan mode:** process staged waves from `IMPLEMENTATION_PLAN.md`, where each wave contains multiple non-overlapping sprint packets that can be assigned to parallel sub-agents.

A **sprint** is a self-contained unit of work: it has a goal, write ownership, required context, tasks, acceptance criteria, and an explicit out-of-scope boundary. A **wave** is an ordered integration stage containing one or more sprint packets that may run in parallel when their write sets do not overlap and their dependencies are satisfied.

**Related skills:**
- `opinionated-software-engineering:software-engineer` — Design principles that apply during implementation
- `opinionated-software-engineering:test-driven-development` — Testing philosophy for acceptance criteria
- `opinionated-software-engineering:git-version-control` — Commit standards for sprint completion
</skill_scope>

## SPRINTS.md Document Format

<document_format>
`SPRINTS.md` is a serial planning document where each numbered `##` section is one sprint. The workflow reads this file to find the next sprint to work on, and records completion when a sprint completes.

**Minimal sprint structure:**

```markdown
## Sprint N: Title

**Goal:** One sentence describing what this sprint produces.

**What changes:**
Detailed description of all changes.

### Task N.X — Task Title
Task description.

**Acceptance criteria:**
- [ ] criterion — `runnable command`

**Out of scope:**
What is explicitly excluded from this sprint.
```

**Completed sprint** — a sprint that has been written back has its heading updated:

```markdown
## Sprint N: Title ✓ Complete (YYYY-MM-DD)
```

The serial workflow finds the next sprint to implement by scanning `SPRINTS.md` top-to-bottom for the first `##` heading that does not contain `✓ Complete`.
</document_format>

## Wave Plan Document Format

<wave_plan_format>
`IMPLEMENTATION_PLAN.md` is a parallel-friendly planning document. It preserves planned intent and should not be rewritten into an implementation record. Completed reality belongs in `SOFTWARE_STATE.md` or `COMPLETED_SPRINTS.md`.

Load `references/wave-planning.md` when creating, converting, extracting, dispatching, or integrating wave plans. It contains the wave document format, packet templates, completion report template, sub-agent orchestration rules, role/model selection guidance, wave quality checks, and serial-to-wave conversion rules.

Core companion documents:

| File | Purpose |
|------|---------|
| `IMPLEMENTATION_PLAN.md` | Original plan: waves, sprint packets, dependencies, ownership, acceptance criteria |
| `currentwork.md` | Active coordination state for the current wave or serial sprint |
| `currentwork/sprint-NX.md` | Extracted work packet for one agent |
| `currentwork/results/NX.md` | Completion report written by one agent |
| `SOFTWARE_STATE.md` | Current implemented reality after each completed wave |
| `COMPLETED_SPRINTS.md` | Optional append-only implementation history |

Use `IMPLEMENTATION_PLAN.md` when the user wants staged parallel work, multiple sub-agents, or a plan designed for non-overlapping implementation batches.
</wave_plan_format>

## When to Use This Skill

<when_to_use>
Activate when any of these conditions hold:

- A `SPRINTS.md` or equivalent planning document is present in the project
- An `IMPLEMENTATION_PLAN.md` wave plan is present in the project
- The user asks to "start", "extract", "begin", or "continue" a sprint
- The user asks to create, convert, or run a parallel wave plan
- A `currentwork.md` file exists and work is in progress
- The user asks about write-back, implementation records, or sprint completion
- The user asks about sprint quality, feasibility, or context budget

Do not activate for:
- General software architecture discussions not tied to a sprint
- Reviewing completed sprints in SPRINTS.md without intent to implement
</when_to_use>

## Mode Selection

<mode_selection>
Choose the workflow mode before extracting work:

| Condition | Mode |
|-----------|------|
| User asks for parallel agents, breakout sprints, batched stages, or wave planning | Wave-plan mode |
| `IMPLEMENTATION_PLAN.md` exists and has incomplete waves | Wave-plan mode |
| `SPRINTS.md` exists and no parallelization is requested | Serial mode |
| Existing `currentwork.md` references a serial sprint | Serial mode |
| Existing `currentwork.md` references an active wave | Wave-plan mode |

Serial mode remains the fallback when write ownership is unclear, sprint dependencies are tightly coupled, or parallel execution would create merge conflicts.
</mode_selection>

## Serial Core Process

<serial_core_process>
The serial sprint workflow has three phases. Each phase has a clear entry condition and exit condition.

### Phase 1: Extract

**Entry condition:** User wants to start a new sprint; `currentwork.md` does not exist or is from a previous completed sprint.

**Steps:**
1. Read `SPRINTS.md` completely. Identify the first sprint not marked `✓ Complete`.
2. Run quality checks on that sprint (see `<quality_checks>`). If any checks fail, fix the spec in `SPRINTS.md` before proceeding.
3. Create `currentwork.md` from the sprint spec using the template in `<currentwork_template>`. The extraction adds three structural elements not in the original spec: prerequisites block, files-to-read block, and definition-of-done block.
4. Read every file listed in the files-to-read block before writing any code.

**Exit condition:** `currentwork.md` created, prerequisite commands pass, all listed files have been read.

### Phase 2: Implement

**Entry condition:** `currentwork.md` exists with at least one incomplete task (`[ ]`).

**Steps:**
1. Find the first incomplete task (`[ ]`).
2. Read any files that task requires if not already read.
3. Implement the task.
4. Run the task's acceptance criteria commands. All must pass before marking the task complete.
5. Fill in the implementation notes subsection: actual files touched with line numbers, any deviation from the spec with the reason.
6. Mark `[ ]` → `[x]`.
7. Repeat for the next incomplete task.

**Exit condition:** All tasks marked `[x]`, Definition of Done commands all pass.

**On deviation from spec:** When reality requires a different approach than the spec describes, implement what is correct, document the deviation in the Deviations Log with the reason, and continue. Do not silently diverge — every deviation must be recorded.

### Phase 3: Write Back and Bootstrap

**Entry condition:** All tasks `[x]`, Definition of Done passes.

**Proceed without confirmation by default:** When all checks pass, write back and bootstrap the next sprint without asking. Ask the user before write-back only if there is a failed check, destructive action, unresolved deviation, ambiguous product decision, or user instruction requiring confirmation.

**Write-back steps:**
1. Read `currentwork.md` completely — this is the source of truth for what was built.
2. Identify the sprint section in `SPRINTS.md`: the full block from the sprint's `## Sprint N:` heading up to (but not including) the next `##` heading, or end of file.
3. Compose the complete replacement text using `<writeback_convention>`. Write the entire section out in full before making any edits.
4. Make a **single Edit call** to replace the sprint section in `SPRINTS.md`. This is one atomic substitution, not a series of incremental edits.

**Bootstrap next sprint (immediately after write-back):**
5. Scan `SPRINTS.md` for the next incomplete sprint (first `##` heading without `✓ Complete`).
6. **If a next sprint exists:**
   a. Run quality checks (see `<quality_checks>`) on that sprint.
   b. If quality checks pass: extract the sprint into `currentwork.md` using `<currentwork_template>`, replacing its current contents with a single Write call. Tell the user the next sprint is ready and they can clear context to begin.
   c. If quality checks fail: write a single `currentwork.md` that lists only the failures and the fixes needed. Tell the user what needs fixing before the next sprint can start.
7. **If no next sprint exists:** clear `currentwork.md` with a single Write call containing only a "All sprints complete" note. Tell the user the plan is finished.

**Exit condition:** completion is recorded; `currentwork.md` contains either the next sprint (ready to implement), a quality-check failure report, or an "all complete" note.
</serial_core_process>

## Wave-Plan Core Process

<wave_plan_core_process>
The wave-plan workflow has four phases: prepare, dispatch, integrate, and advance.

### Phase 1: Prepare Wave

**Entry condition:** User asks for parallel work, `IMPLEMENTATION_PLAN.md` exists, or a serial plan should be converted to waves.

**Steps:**
1. Read `IMPLEMENTATION_PLAN.md` completely, or convert `SPRINTS.md` using `<serial_to_wave_conversion>`.
2. Identify the first incomplete wave whose dependencies are satisfied.
3. Run wave quality checks (see `<wave_quality_checks>`).
4. Create or update `currentwork.md` as the wave coordination index using `<wave_currentwork_template>`.
5. Extract each ready sprint packet into `currentwork/sprint-NX.md`.

**Exit condition:** `currentwork.md` lists the active wave, sprint packets, owners, dependencies, write sets, dispatch status, and integration gate.

### Phase 2: Dispatch Parallel Work

**Entry condition:** Active wave has two or more ready sprint packets with non-overlapping write sets.

**Steps:**
1. Assign each sprint packet to one sub-agent only when the user or runtime permits sub-agent use.
2. Select the least expensive capable agent role/model tier using `references/wave-planning.md`.
3. Give each sub-agent its packet file, owned write paths, read-only paths, acceptance criteria, and completion-report path.
4. Tell sub-agents they are not alone in the codebase, must not revert others' work, and must stay inside their write ownership.
5. If sub-agents are unavailable, process packets serially while preserving packet boundaries.

**Exit condition:** each dispatched packet has a completion report in `currentwork/results/NX.md`, or a blocker is recorded in `currentwork.md`.

### Phase 3: Integrate Wave

**Entry condition:** All dispatched sprint packets are complete or blocked.

**Steps:**
1. Read every `currentwork/results/NX.md` report.
2. Verify files changed match each packet's declared write ownership.
3. Run each packet's acceptance criteria, then the wave integration gate.
4. Resolve integration failures locally if they are within the coordinator's scope; otherwise record the blocker.
5. Update `SOFTWARE_STATE.md` with what the software now does, important implementation decisions, public contracts, and known limitations.
6. Append implementation details to `COMPLETED_SPRINTS.md` if that file is used.
7. Mark the wave complete in `currentwork.md`; optionally add a concise completion marker in `IMPLEMENTATION_PLAN.md`, but do not replace planned text with implementation text.

**Exit condition:** integration gate passes and implemented reality is recorded in `SOFTWARE_STATE.md`.

### Phase 4: Advance

**Entry condition:** Active wave is complete.

**Steps:**
1. Scan `IMPLEMENTATION_PLAN.md` for the next incomplete wave whose dependencies are satisfied.
2. If a next wave exists, prepare it immediately.
3. If no next wave exists, update `currentwork.md` with an "all waves complete" note.

**Exit condition:** next wave is ready, blocked with specific reasons, or all waves are complete.
</wave_plan_core_process>

## currentwork.md Template

<currentwork_template>
When creating `currentwork.md` for serial mode, use this structure. Sections in `[brackets]` are instructions; replace them with actual content.

```markdown
# currentwork.md — Sprint [N]: [Title from SPRINTS.md]

**Status:** In Progress  
**Started:** [YYYY-MM-DD]  
**Source:** Sprint [N] in SPRINTS.md

---

## Prerequisites

Run these before writing any code. All must pass.

- [ ] `[test command — e.g., uv run pytest backend/tests/ -x]`
- [ ] `[any sprint-specific verification command]`

---

## Files to Read Before Starting

Read every file on this list completely before touching anything.

- `[/absolute/path/file.py]` — [what to understand from it]
- `[/absolute/path/component.jsx]` — [what to understand from it]

---

## Tasks

### Task [N.X] — [Title]

**Status:** [ ]

[Paste task spec from SPRINTS.md verbatim]

**Implementation notes:**
*(fill in after completing)*

**Acceptance criteria:**
- [ ] [criterion] — `[runnable command or [manual check] label]`

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

## Write-back Status

- [ ] All tasks complete  
- [ ] Definition of Done passes  
- [ ] Implementation notes filled in for every task  
- [ ] Deviations log complete  
- [ ] Written back to SPRINTS.md  
- [ ] Next sprint bootstrapped (or all sprints complete)  
```
</currentwork_template>

## Wave currentwork.md Template

<wave_currentwork_template>
Use the template in `references/wave-planning.md`. Keep `currentwork.md` as a coordination index, not as the detailed packet spec.
</wave_currentwork_template>

## Sprint Packet Template

<sprint_packet_template>
Use the template in `references/wave-planning.md`. Each `currentwork/sprint-NX.md` file must be self-contained enough for one agent to execute without reading unrelated plan sections.
</sprint_packet_template>

## Completion Report Template

<completion_report_template>
Use the template in `references/wave-planning.md`. Sub-agents write one completion report each; the coordinator is the only writer to shared state documents.
</completion_report_template>

## Quality Checks

<quality_checks>
Run these before extracting any sprint. A sprint that fails quality checks is not ready to implement — fix the spec first.

### Self-containment check

| Check | How to verify |
|-------|---------------|
| No cross-sprint references | Grep the sprint text for "see Sprint", "as described above", "from Sprint N" — must be zero hits |
| Prerequisites are runnable | Every item in the prerequisites block is a shell command, not prose |
| All file paths are absolute | File paths start with `/` or are clearly relative to a known root |

### Completeness check

| Check | Minimum |
|-------|---------|
| Files listed | Every file to be touched is listed with path |
| New files have skeletons | Every new file shows at minimum the function/class signatures |
| Acceptance criteria are testable | Each criterion is either a runnable command or explicitly `[manual check: description]` |
| Spec tokens per task | `(sprint_spec_chars / 4) / task_count ≥ 500 tokens` — below this, tasks are underspecified |

### Feasibility check

Estimate whether the sprint fits in the target model's context window. Use this formula:

```
session_budget   = context_window_tokens × 0.5

file_tokens      = files_to_read_count × avg_file_tokens
                 = files_to_read_count × (avg_file_lines × 5)

spec_tokens      = sprint_spec_chars ÷ 4

feasible         = (file_tokens + spec_tokens) ≤ session_budget
```

Reference context windows and session budgets:

| Model | Context window | Session budget |
|-------|---------------|----------------|
| claude-sonnet-4-6 | 200,000 tokens | 100,000 tokens |
| claude-opus-4-7 | 200,000 tokens | 100,000 tokens |
| claude-haiku-4-5 | 200,000 tokens | 100,000 tokens |
| gpt-4o | 128,000 tokens | 64,000 tokens |

If the sprint exceeds the session budget, split it before extracting.

### Granularity check

| Bound | Recommendation |
|-------|----------------|
| Files touched | ≤ 15 |
| Tasks | ≤ 8 |
| New lines of code (estimated) | ≤ 500 |

Exceeding any bound is a signal (not a hard rule) to consider splitting.
</quality_checks>

## Wave Quality Checks

<wave_quality_checks>
Run the checks in `references/wave-planning.md` before dispatching a wave. A wave that fails those checks is not ready for parallel execution; either fix the plan or run the affected packets serially.
</wave_quality_checks>

## Serial to Wave Conversion

<serial_to_wave_conversion>
Use this when the user asks to convert a serial `SPRINTS.md` plan into a parallel wave plan. Load `references/wave-planning.md`, build dependency and ownership metadata, batch independent packets into waves, and write the result to `IMPLEMENTATION_PLAN.md`. Preserve `SPRINTS.md` as the original serial source unless the user asks to replace it.
</serial_to_wave_conversion>

## Write-back Convention

<writeback_convention>
When writing a completed serial sprint back to `SPRINTS.md`, convert the spec into an implementation record using these substitutions. In wave-plan mode, preserve `IMPLEMENTATION_PLAN.md` as planned intent and write implemented reality to `SOFTWARE_STATE.md` or `COMPLETED_SPRINTS.md` instead.

| Spec element | Implementation record element |
|--------------|-------------------------------|
| `## Sprint N: Title` | `## Sprint N: Title ✓ Complete (YYYY-MM-DD)` |
| `**Goal:**` | Keep as-is |
| `### What changes:` or `**What changes:**` | `### What was done:` |
| Planned function signatures / code snippets | Actual code at `file.py:line` or short excerpt |
| `**Acceptance criteria:**` list | `**Results:**` list with `[PASS]` or `[FAIL: notes]` per item |
| `**Out of scope:**` | Keep as-is |
| *(not present)* | `**Deviations:**` section if any deviations occurred |
| *(not present)* | `**Files created:**` and `**Files modified:**` tables |

The goal is accuracy: someone reading the record should understand what the software does and why it was built that way, without needing to read the code.

**What NOT to do:**
- Do not summarize vaguely ("updated the file to handle X") — cite actual line numbers
- Do not omit deviations — they are the most useful part of the record for future sessions
- Do not remove the acceptance criteria — their pass/fail status is diagnostic
</writeback_convention>

## Handling Incomplete Sessions

<incomplete_sessions>
If context is cleared or the session ends mid-sprint, the next session should:

1. Read `WORKFLOW.md` (or this skill) to understand the process.
2. Read `currentwork.md` to find current state: checked tasks `[x]` are done, unchecked `[ ]` are not.
3. Read every file in the Files to Read block that is relevant to the next incomplete task.
4. Continue from the first `[ ]` task.

Do not re-run completed tasks. Do not re-read the entire codebase. Trust the implementation notes in `currentwork.md` for what was already done.

If context is cleared mid-wave, the next session should read `currentwork.md`, any active `currentwork/sprint-NX.md` packets, and existing `currentwork/results/NX.md` reports. Continue from the first ready or blocked packet, then run the wave integration phase when all packet reports are present.
</incomplete_sessions>

## Common Mistakes

<common_mistakes>
### Starting without reading files

Implementing based on training data assumptions about what a file contains, rather than reading the actual current file. The Files to Read block exists precisely to prevent this. Read every listed file before writing a single line.

### Marking tasks complete before acceptance criteria pass

A task is not done when the code is written — it is done when its acceptance criteria commands pass. Running `pytest` after every task is not optional overhead; it is the signal that the task is actually complete.

### Skipping the write-back

The most common failure mode. Completing implementation without writing back means `SPRINTS.md` still shows the sprint as "Planned" even though the code is done. The next session (or the next human reading the doc) has no record of what was built. Write-back is not optional.

### Silently deviating from spec

When the spec says "add field X to model Y" and you discover Y already has X, or Y doesn't exist at all — that is a deviation. Record it. The Deviations Log is how future sessions know the spec and the code diverged and why.

### Over-reading before starting

Reading every file in the entire codebase "to get context" before starting the first task burns the context budget. Read only the files in the Files to Read block. If a task requires additional context, read those specific files when you reach that task.

### Incremental write-back

Making multiple small Edit calls to SPRINTS.md during write-back — changing the heading, then the goal, then each task result individually — leaves the document in an inconsistent state between calls and generates unnecessary tool noise. Compose the entire replacement section first, then replace it in one Edit call.

### Skipping the bootstrap

Clearing `currentwork.md` after write-back without checking for a next sprint forces the user to manually trigger sprint extraction in their next session. After every write-back, always scan for the next sprint and populate `currentwork.md` before the session ends.

### Letting sub-agents edit shared state

In wave-plan mode, sub-agents should write only their owned code paths and their own completion report. The coordinator updates `currentwork.md`, `SOFTWARE_STATE.md`, `COMPLETED_SPRINTS.md`, and plan status after validating packet results.
</common_mistakes>
