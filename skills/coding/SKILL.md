---
description: >
  Implement planned code changes by delegating the primary execution to Codex CLI. Use this for
  features, refactors, fixes, and tests when you already have a plan file. Not for open-ended
  research and not for code review.
user-invocable: true
disable-model-invocation: false
allowed-tools: Bash, Read, Glob, Grep
argument-hint: "<absolute-path-to-plan-file>"
---

# Codex Control Plane: Coding

Use this skill when the user is asking:

- "Build this."
- "Implement the plan."
- "Make the planned refactor."
- "Write the code and run the checks."

This skill is fully self-contained. Do not depend on any external protocol file.

## Core Rule

Primary implementation work MUST go to Codex through `codex exec`.

You may use Claude's tools only for:

- reading the plan and scoping the target repository
- locating repository conventions or instructions
- checking that files changed
- running or checking verification commands from the plan when feasible

Do not perform the main implementation yourself while this skill is active.

## Plan Requirement

A plan file MUST exist before dispatching.

The plan may live:

- inside the target repository
- next to the repository
- anywhere else on disk, as long as it clearly points to the target repository

If no plan file is provided, stop and ask for one.

## Intake

Read the plan first. Determine:

- the target repository path
- the goals and acceptance criteria
- the files or modules likely involved
- any verification commands the plan requires

Resolve the repository to an absolute path before dispatching.

## Repository Conventions

Before dispatching, look for repository-level instructions and local project conventions. Tell Codex to follow them too.

Do not assume any specific personal workflow, commit policy, or directory layout.

## Prompt Style

Build a navigational prompt for Codex.

- Inline only the facts Codex cannot discover itself.
- Point to the plan file and important source files by absolute path.
- Tell Codex to read the plan first.

The prompt MUST include:

- the absolute path to the plan file
- the target repository path
- the implementation objective
- the plan's constraints and acceptance criteria
- verification commands from the plan, when present
- a reminder to check for local project instructions before proceeding

## Output Expectations

This skill produces code changes in the target repository, not a document artifact.

Leave changes uncommitted unless the user explicitly asked for a commit as part of the task.

## Dispatch Command

Use `workspace-write` sandboxing:

```bash
codex exec --sandbox workspace-write --full-auto --skip-git-repo-check \
  --config model_reasoning_effort="high" \
  -C "<target-repo-path>" \
  "<self-contained prompt>" 2>/dev/null
```

Always capture the returned `session_id`.

## Verification Loop

After Codex returns:

1. Check whether files changed with `git -C "<target-repo-path>" diff --stat` or `git -C "<target-repo-path>" status --short`.
2. Compare the change set against the plan's goals and acceptance criteria.
3. Run verification commands from the plan when feasible.
4. If implementation is incomplete or verification fails, resume the same Codex session:

```bash
echo "<what is missing or needs fixing>" | \
  codex exec --skip-git-repo-check resume <session-id> 2>/dev/null
```

Keep the quality loop inside the same session. Do not restart the task with a fresh session unless the original session is unusable.

## Final Response

Return:

- a short implementation summary
- the Codex `session_id`
- the `git diff --stat` summary
- verification results
- the result block

Use this YAML result block:

```yaml
---
role: codex-coding
cwd: /path/to/repo
session_id: <from codex exec>
artifact_path: N/A
diff_stat: <git diff --stat output or N/A>
verification: pass | fail | skipped
---
```
