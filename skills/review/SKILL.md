---
description: >
  Review existing code, diffs, branches, or modules by delegating the primary analysis to
  Codex CLI. Use this for bugs, regressions, design risks, and missing tests. Not for making
  the fixes directly and not for broad exploratory research.
user-invocable: true
disable-model-invocation: false
allowed-tools: Bash, Read, Write, Glob, Grep
argument-hint: "<target-repo-path> <review-scope>"
---

# Codex Control Plane: Review

Use this skill when the user is asking:

- "Is this code good?"
- "Review this diff."
- "Find bugs or risks in this module."
- "Check for regressions or missing tests."

This skill is fully self-contained. Do not depend on any external protocol file.

## Core Rule

Primary review work MUST go to Codex through `codex exec`.

You may use Claude's tools only for:

- intake and scope clarification
- locating the repository and review scope
- finding repository conventions or instructions
- verifying that the review artifact exists and is complete

Do not perform the main review yourself.

## Intake

From `$ARGUMENTS`, determine:

- the target repository path
- the review scope: files, module, branch, commit range, or latest diff
- any requested focus areas such as correctness, performance, security, or design
- the desired artifact path, if the user specified one

Resolve the repository to an absolute path before dispatching.

If the user did not give an output path:

- default to the current working directory
- write a file named `YYYY-MM-DD__review__<topic>.md`

If the user gave an output base directory, write the file under that base.

## Repository Conventions

Before dispatching, look for repository-level instructions and local project conventions. Tell Codex to follow them too.

Do not assume any specific internal naming scheme or documentation layout.

## Prompt Style

Build a navigational prompt for Codex.

- Inline only the facts Codex cannot discover itself.
- Point to files by absolute path instead of pasting their contents.
- Tell Codex exactly what scope to review and what quality bar to use.

The prompt MUST include:

- the review objective
- the repository path
- the review scope
- any focus areas
- the artifact path
- a reminder to check for local project instructions before proceeding

## Required Artifact Format

Codex should write markdown with:

1. A header: `> Produced by Codex on YYYY-MM-DD. Source: <repo>. Scope: <scope>`
2. `Summary`
3. `Findings`, where each finding includes:
   - severity: `critical`, `major`, `minor`, or `nit`
   - file and line reference
   - issue description
   - suggested fix when useful
4. `Positive Observations`
5. `Priority Actions`

All file references in the artifact should be relative to the repository root.

## Dispatch Command

Use `read-only` sandboxing:

```bash
codex exec --sandbox read-only --full-auto --skip-git-repo-check \
  --config model_reasoning_effort="high" \
  -C "<target-repo-path>" \
  "<self-contained prompt>" 2>/dev/null
```

Always capture the returned `session_id`.

## Verification Loop

After Codex returns:

1. Check whether the artifact exists at the target path.
2. If it does not exist but the content is present in stdout, save it to the target path.
3. Review the artifact for:
   - coverage of the requested scope
   - specific findings with file and line references
   - sensible severity levels
   - a clear priority list
4. If anything is missing, resume the same Codex session:

```bash
echo "<what is missing or needs fixing>" | \
  codex exec --skip-git-repo-check resume <session-id> 2>/dev/null
```

Keep the quality loop inside the same session. Do not restart the task with a fresh session unless the original session is unusable.

## Final Response

Return:

- a short review summary
- the Codex `session_id`
- the artifact path
- the result block

Use this YAML result block:

```yaml
---
role: codex-review
cwd: /path/to/repo
session_id: <from codex exec>
artifact_path: <output file>
diff_stat: N/A
verification: pass | fail | skipped
---
```
