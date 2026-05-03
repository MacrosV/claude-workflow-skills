---
name: sprint-workflow
description: Sprint extraction, tracking, and write-back for AI-assisted development. Use when working with a SPRINTS.md planning document: extracting a sprint into currentwork.md before starting, tracking implementation progress, enforcing quality checks, and writing back the implementation record when done. Also activates when the user mentions "extract sprint", "start sprint", "write back", or "currentwork.md".
---

# Sprint Workflow

<skill_scope skill="sprint-workflow">
This skill provides the process and quality framework for implementing software in discrete, context-isolated sprints using a structured planning document. It is designed to work with `SPRINTS.md` planning documents and `currentwork.md` working files.

**Related skills:**
- `opinionated-software-engineering:software-engineer` — Design principles that apply during implementation
- `opinionated-software-engineering:test-driven-development` — Testing philosophy for acceptance criteria
- `opinionated-software-engineering:git-version-control` — Commit standards for sprint completion
</skill_scope>

## When to Use This Skill

<when_to_use>
Activate when any of these conditions hold:

- A `SPRINTS.md` or equivalent planning document is present in the project
- The user asks to "start", "extract", "begin", or "continue" a sprint
- A `currentwork.md` file exists and work is in progress
- The user asks about write-back, implementation records, or sprint completion
- The user asks about sprint quality, feasibility, or context budget

Do not activate for:
- General software architecture discussions not tied to a sprint
- Reviewing completed sprints in SPRINTS.md without intent to implement
</when_to_use>

## Core Process

<core_process>
The sprint workflow has three phases. Each phase has a clear entry condition and exit condition.

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

### Phase 3: Write Back

**Entry condition:** All tasks `[x]`, Definition of Done passes.

**Steps:**
1. Convert `currentwork.md` to an implementation record (see `<writeback_convention>`).
2. In `SPRINTS.md`, replace the sprint's spec section with the implementation record.
3. Clear or archive `currentwork.md`.

**Exit condition:** `SPRINTS.md` updated, `currentwork.md` cleared.
</core_process>

## currentwork.md Template

<currentwork_template>
When creating `currentwork.md`, use this structure. Sections in `[brackets]` are instructions; replace them with actual content.

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
```
</currentwork_template>

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

## Write-back Convention

<writeback_convention>
When writing a completed sprint back to `SPRINTS.md`, convert the spec into an implementation record using these substitutions:

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
</common_mistakes>
