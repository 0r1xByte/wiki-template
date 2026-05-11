---
name: write-a-skill
description: Create new agent skills with proper structure, progressive disclosure, and bundled resources. Use when user wants to create, write, or build a new skill.
---

# Writing Skills

## Process

1. **Gather requirements** - ask user about:
   - What task/domain does the skill cover?
   - What specific use cases should it handle?
   - Does it need executable scripts or just instructions?
   - Any reference materials to include?

2. **Draft the skill** - create:
   - SKILL.md with concise instructions
   - Additional reference files if content exceeds 500 lines
   - Utility scripts if deterministic operations needed

3. **Review with user** - present draft and ask:
   - Does this cover your use cases?
   - Anything missing or unclear?

## Skill Structure

```
.agent/skills/skill-name/
├── SKILL.md           # Main instructions (required)
└── REFERENCE.md       # Detailed docs (if needed)
```

Add a matching command file at `.agent/commands/skill-name.md`:
```
Read and follow `.agent/skills/skill-name/SKILL.md` exactly.

$ARGUMENTS
```

## SKILL.md Template

```md
---
name: skill-name
description: Brief description. Use when [specific triggers].
---

# Skill Name

[Instructions]
```

## Description Requirements

- Max 1024 chars
- First sentence: what it does
- Second sentence: "Use when [specific triggers]"
