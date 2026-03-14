# Protocol Reference

This document describes the shared dispatch and verification contract behind Codex Control Plane. It exists as a clean reference for humans. Each shipped skill also contains its own inline copy so the plugin remains self-contained after installation.

## Core Rule

Primary execution work goes to Codex through `codex exec`.

Use Claude for:

- intake
- scoping
- finding project conventions
- checking that required files exist
- verifying the returned artifact or code changes

Do not do the main research, review, or implementation work directly when a control-plane skill is invoked.

## Prompting Style

Build navigational prompts for Codex.

- Inline only the facts Codex cannot discover for itself.
- Point to files by absolute path instead of pasting their contents.
- Tell Codex where to work and what to produce.
- Tell Codex to look for repository-level instructions and follow local project conventions before making changes.

Every dispatch prompt should include:

- the objective
- the target repository path or explicit task scope
- relevant file paths
- the output path, if the role produces an artifact
- role-specific constraints and verification requirements

## Sandbox Policy

| Role | Sandbox |
| --- | --- |
| `research` | `read-only` |
| `review` | `read-only` |
| `coding` | `workspace-write` |
| `index` | `read-only` |
| `plan` | `read-only` |
| `general` | policy-dependent, usually manual-only |

Only three roles are public in v0.1, but the policy model is designed for the full architecture.

## Initial Dispatch

Template:

```bash
codex exec --sandbox <level> --full-auto --skip-git-repo-check \
  --config model_reasoning_effort="high" \
  -C "<target-repo-path>" \
  "<self-contained prompt>" 2>/dev/null
```

Always capture the returned `session_id`.

## Continue The Same Task

If verification finds gaps, continue the same task in the same Codex session:

```bash
echo "<what is missing or needs fixing>" | \
  codex exec --skip-git-repo-check resume <session-id> 2>/dev/null
```

v0.1 keeps continuation simple: same-task resume only.

## Verification Loop

### Document-producing roles

Examples: research, review, plan, index.

1. Check whether the artifact exists at the requested path.
2. If the file is missing but the content is present in stdout, write it to the target path.
3. Review the artifact against the role-specific quality criteria.
4. Resume the same Codex session if the artifact is incomplete.

### Code-changing roles

Examples: coding.

1. Check whether files changed with `git diff --stat` or `git status`.
2. Compare the changes against the plan or acceptance criteria.
3. Run verification commands from the plan when feasible.
4. Resume the same Codex session if the implementation is incomplete or verification fails.

## Artifact Paths

For public document skills:

- use an explicit path if the user gives one
- otherwise write to the current working directory
- if a base directory is provided, write underneath that base

Recommended default filenames:

- `YYYY-MM-DD__research__<topic>.md`
- `YYYY-MM-DD__review__<topic>.md`

## Result Block

Every skill should end its response with a machine-readable block:

```yaml
---
role: codex-{role}
cwd: /path/to/repo
session_id: <from codex exec>
artifact_path: <output file or "N/A">
diff_stat: <git diff --stat output or "N/A">
verification: pass | fail | skipped
---
```

## Final Response Expectations

A skill response should include:

- a short human-readable summary
- the Codex `session_id`
- the artifact path or diff summary
- verification status
- the result block
