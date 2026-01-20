# agent-skill
Shared agent skills for EFE

## Structure

This repository contains agent skills organized in the following structure:

```
skills/
  ├── <skill-name>/
  │   ├── SKILL.md        # Required: Skill metadata and description
  │   └── ...             # Other skill-related files
  └── ...
```

### Skill Format

Each skill directory must contain at least one `SKILL.md` file with the following format:

```markdown
---
name: skill-name
description: Brief description of the skill
---
Detailed description and usage instructions.
```

### Available Skills

- **hello**: A template skill example
