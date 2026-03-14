# Example: Coding Skill

A skill that uses Codex to implement features and write code.

Copy this file to `skills/code/SKILL.md` and customize.

```markdown
---
name: code
description: Implement features using Codex as coding agent
user-invocable: true
---

# /code — Code with Codex

## Arguments

- Remaining args — what to implement

## Workflow

1. Read the relevant code to understand the current state
2. Build a prompt with:
   - What to implement (specific, actionable)
   - File paths to modify (absolute paths)
   - Constraints (style, patterns to follow, tests to pass)
3. Dispatch:

\```bash
codex exec --full-auto --sandbox network-only \
  -C "<repo-path>" \
  "<implementation-prompt>" 2>/dev/null
\```

4. Verify: check `git diff`, run tests if applicable
5. Report: what changed + session ID
```

## What to Customize

- **Sandbox**: `network-only` (default) or `full` for tasks needing network
- **Verification**: Run tests? Lint? Type check?
- **Commit policy**: Should Codex commit, or leave changes staged?
