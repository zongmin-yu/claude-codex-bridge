# Example: Code Review Skill

A skill that uses Codex to review code for bugs and design issues.

Copy this file to `skills/review/SKILL.md` and customize.

```markdown
---
name: review
description: Code review powered by Codex
user-invocable: true
---

# /review — Review with Codex

## Arguments

- Remaining args — what to review (branch, file, or diff description)

## Workflow

1. Identify the scope: specific files, a branch diff, or recent commits
2. Build a prompt:
   - Point to the files/diff to review
   - Specify what to look for (bugs, design, performance, security)
   - Specify output format (markdown report at a given path)
3. Dispatch:

\```bash
codex exec --full-auto --sandbox read-only \
  -C "<repo-path>" \
  "Review <scope>. Write report to <output-path>." 2>/dev/null
\```

4. Verify the report exists and covers the requested scope
5. Report findings + session ID
```

## What to Customize

- **Review focus**: Security? Performance? API design? All of the above?
- **Output**: Inline comments? Summary report? Severity ratings?
- **Scope**: Single file, PR diff, or entire module?
