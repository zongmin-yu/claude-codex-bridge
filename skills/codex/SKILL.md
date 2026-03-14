---
name: codex
description: Dispatch any task to Codex CLI as a sub-agent
user-invocable: true
---

# /codex — Codex Sub-Agent Dispatch

Send a task to OpenAI Codex CLI and bring back the result.

## Arguments

- Remaining args — the task description (what you want Codex to do)

## The Pattern

Three steps: **build prompt → dispatch → verify**.

### Step 1: Build the Prompt

Turn the user's request into a self-contained prompt for Codex:
- Include absolute file paths to relevant code (don't paste contents — Codex can read files)
- Specify the expected output (a file, a commit, a report)
- Add any constraints (don't modify tests, use specific API, etc.)

### Step 2: Dispatch

```bash
codex exec --full-auto --sandbox read-only \
  -C "<working-directory>" \
  "<self-contained-prompt>" 2>/dev/null
```

Sandbox levels (pick one):
- `read-only` — can read files, no writes (research, review)
- `network-only` — can write files, no network (coding)
- `full` — full access (when needed)

Capture the **session ID** from stdout — it enables resume.

### Step 3: Verify

Check that Codex produced what was asked:
- File exists at the expected path? Read and spot-check.
- Commit landed? `git log --oneline -1`
- Report answers the question? Scan for gaps.

If something is missing, resume the same session:

```bash
echo "<what's missing>" | codex exec resume <session-id> 2>/dev/null
```

## After Completion

Report to the user:
- What Codex produced
- Session ID (so they can resume manually if needed)
- Any issues found during verification
