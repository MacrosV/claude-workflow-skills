# personal-claude-skills

Personal Claude Code skill collection.

## Skills

### sprint-workflow

Sprint extraction, implementation tracking, and write-back for AI-assisted development against structured sprint plans (`SPRINTS.md`).

**Activates when:** Working with a `SPRINTS.md` planning document; user mentions "extract sprint", "start sprint", "write back", or "currentwork.md".

**Provides:**
- Sprint extraction process (spec → self-contained `currentwork.md`)
- Implementation tracking (task checkboxes, implementation notes, deviations log)
- Quality checks (feasibility, completeness, self-containment) before each sprint starts
- Write-back convention (completed `currentwork.md` → implementation record in `SPRINTS.md`)
- Guidance for resuming after context clears mid-sprint

## Installation

```bash
# In Claude Code, run:
/plugin marketplace add MacrosV/personal-claude-skills
/plugin install sprint-workflow@personal-claude-skills
```

## Setup on a New Machine

```bash
# Clone this repo somewhere convenient:
git clone https://github.com/MacrosV/personal-claude-skills.git ~/projects/personal-claude-skills

# In Claude Code:
/plugin marketplace add MacrosV/personal-claude-skills
/plugin install sprint-workflow@personal-claude-skills
```

## Adding New Skills

1. Create `new-skill-name/plugin.json` and `new-skill-name/skills/new-skill-name/SKILL.md`
2. Add the plugin entry to `.claude-plugin/marketplace.json`
3. Commit and push
4. Run `/plugin marketplace update personal-claude-skills` in Claude Code to pick up the new skill
