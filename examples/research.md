# Example: Research Skill

A skill that uses Codex to investigate codebases and answer questions.

Copy this file to `skills/research/SKILL.md` and customize.

```markdown
---
name: research
description: Deep investigation powered by Codex
user-invocable: true
---

# /research — Investigate with Codex

## Arguments

- Remaining args — the question to investigate

## Workflow

1. Identify which repo/files are relevant to the question
2. Build a prompt that points Codex to specific file paths
3. Dispatch:

\```bash
codex exec --full-auto --sandbox read-only \
  -C "<repo-path>" \
  "Investigate: <question>. Write findings to <output-path>." 2>/dev/null
\```

4. Verify the output file exists and answers the question
5. If gaps remain, resume: `echo "<what's missing>" | codex exec resume <session-id>`
6. Report findings + session ID to the user
```

## What to Customize

- **Output format**: Markdown report? JSON? Structured checklist?
- **Scope**: Single repo or cross-repo investigation?
- **Depth**: Quick scan or exhaustive trace?
