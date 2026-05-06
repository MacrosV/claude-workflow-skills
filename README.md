# claude-workflow-skills

Personal Claude Code skill collection.

## Skills

### sprint-workflow
This skill was created with Claude using the skill creation skill from Pyroxin's Opinionated Claude Skills (https://github.com/Pyroxin/opinionated-claude-skills).

Sprint extraction, implementation tracking, write-back, and parallel wave planning for AI-assisted development against structured serial plans (`SPRINTS.md`) or staged parallel plans (`IMPLEMENTATION_PLAN.md`).

**Activates when:** Working with a `SPRINTS.md` or `IMPLEMENTATION_PLAN.md` planning document; user mentions "extract sprint", "start sprint", "write back", "currentwork.md", "wave plan", "parallel sprints", or "breakout agents".

**Provides:**
- Sprint extraction process (spec → self-contained `currentwork.md`)
- Implementation tracking (task checkboxes, implementation notes, deviations log)
- Quality checks (feasibility, completeness, self-containment) before each sprint starts
- Write-back convention (completed `currentwork.md` → implementation record in `SPRINTS.md`)
- Parallel wave planning for batched non-overlapping sprint packets
- Serial-to-wave conversion guidance for turning `SPRINTS.md` into `IMPLEMENTATION_PLAN.md`
- Sub-agent coordination conventions using packet files and completion reports
- Guidance for resuming after context clears mid-sprint

## Installation

### Claude Code

```bash
# In Claude Code, run:
/plugin marketplace add MacrosV/claude-workflow-skills
/plugin install sprint-workflow@claude-workflow-skills
```

### Codex

This repo also includes a Codex-native plugin manifest at `sprint-workflow/.codex-plugin/plugin.json` and a repo-local marketplace at `.agents/plugins/marketplace.json`.

Use the local marketplace from this repo, or install the plugin directly from:

```text
./sprint-workflow
```

## Setup on a New Machine

```bash
# Clone this repo somewhere convenient:
git clone https://github.com/MacrosV/claude-workflow-skills.git ~/projects/claude-workflow-skills

# In Claude Code:
/plugin marketplace add MacrosV/claude-workflow-skills
/plugin install sprint-workflow@claude-workflow-skills

# In Codex:
# Use .agents/plugins/marketplace.json or install ./sprint-workflow as a local plugin.
```

## Adding New Skills

1. Create `new-skill-name/plugin.json` and `new-skill-name/skills/new-skill-name/SKILL.md`
2. Add the plugin entry to `.claude-plugin/marketplace.json`
3. Commit and push
4. Run `/plugin marketplace update claude-workflow-skills` in Claude Code to pick up the new skill
