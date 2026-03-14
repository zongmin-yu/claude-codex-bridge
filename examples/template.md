# Template: Build Your Own Codex Skill

Copy this to `skills/<your-skill-name>/SKILL.md` and fill in the blanks.

```markdown
---
name: <your-skill-name>
description: <one line — what does this skill do?>
user-invocable: true
---

# /<your-skill-name>

## Arguments

- Remaining args — <what the user provides>

## Workflow

1. <Understand the request — what context do you need?>
2. <Build the prompt — what should Codex do? What files should it look at?>
3. Dispatch:

\```bash
codex exec --full-auto --sandbox <read-only|network-only|full> \
  -C "<working-directory>" \
  "<prompt>" 2>/dev/null
\```

4. <Verify — how do you check that Codex did the right thing?>
5. <Report — what do you tell the user?>
```

## Tips

- **Be specific in prompts**: "Refactor the auth middleware in `src/auth/middleware.ts`" beats "refactor auth"
- **Use absolute paths**: Codex needs to know exactly where to look
- **Capture session IDs**: Enables resume if verification finds gaps
- **Pick the right sandbox**: `read-only` for investigation, `network-only` for coding, `full` when needed
