---
description: >
  Research a codebase or technical question by delegating the primary investigation to Codex CLI.
  Use this when the user wants grounded answers with evidence, call-chain tracing, dependency
  mapping, or root-cause investigation. Not for implementation work or code review.
user-invocable: true
disable-model-invocation: false
allowed-tools: Bash, Read, Write, Glob, Grep
argument-hint: "<target-repo-path-or-topic> <question>"
---

# Codex Control Plane: Research

Use this skill when the user is asking:

- "What's in this codebase?"
- "How does this part work?"
- "What is true here?"
- "Trace this behavior or dependency."

This skill is fully self-contained. Do not depend on any external protocol file.

## Core Rule

Primary investigation work MUST go to Codex through `codex exec`.

You may use Claude's tools only for:

- intake and scope clarification
- finding repository conventions or instructions
- checking which files exist
- verifying the returned artifact

Do not perform the main investigation yourself.

## Intake

From `$ARGUMENTS`, determine:

- the target repository path or explicit non-repo topic scope
- the question to investigate
- the desired artifact path, if the user specified one

If the target is a local repository, resolve it to an absolute path before dispatching.

If the user did not give an output path:

- default to the current working directory
- write a file named `YYYY-MM-DD__research__<topic>.md`

If the user gave an output base directory, write the file under that base.

## Repository Conventions

Before dispatching, look for repository-level instructions and local project conventions. Tell Codex to follow them too.

Do not assume any specific house style or documentation layout.

## Prompt Style

Build a navigational prompt for Codex.

- Inline only the facts Codex cannot discover itself.
- Point to files by absolute path instead of pasting their contents.
- Tell Codex what question to answer, what scope to inspect, and where to write the artifact.

The prompt MUST include:

- the investigation objective
- the repository path or task scope
- any specific files, modules, or directories to inspect
- the artifact path
- a reminder to check for local project instructions before proceeding

## Required Artifact Format

Codex should write markdown with:

1. A header: `> Produced by Codex on YYYY-MM-DD. Source: <scope>`
2. One section per question with:
   - `Question`
   - `Short Answer`
   - `Evidence`
3. `Uncertainty / Open Questions`
4. `Next Actions`

Evidence should point to concrete files, symbols, commands, or URLs rather than vague assertions.

## Dispatch Command

Use `read-only` sandboxing:

```bash
codex exec --sandbox read-only --full-auto --skip-git-repo-check \
  --config model_reasoning_effort="high" \
  -C "<target-repo-path>" \
  "<self-contained prompt>" 2>/dev/null
```

Always capture the returned `session_id`.

If the task is not tied to a repository, use the most relevant working directory you have while still giving Codex an explicit scope in the prompt.

## Verification Loop

After Codex returns:

1. Check whether the artifact exists at the target path.
2. If it does not exist but the content is present in stdout, save it to the target path.
3. Review the artifact for:
   - complete coverage of the user's questions
   - concrete evidence for each answer
   - explicit uncertainty where needed
4. If anything is missing, resume the same Codex session:

```bash
echo "<what is missing or needs fixing>" | \
  codex exec --skip-git-repo-check resume <session-id> 2>/dev/null
```

Keep the quality loop inside the same session. Do not restart the task with a fresh session unless the original session is unusable.

## Final Response

Return:

- a short summary of findings
- the Codex `session_id`
- the artifact path
- the result block

Use this YAML result block:

```yaml
---
role: codex-research
cwd: /path/to/repo-or-scope
session_id: <from codex exec>
artifact_path: <output file>
diff_stat: N/A
verification: pass | fail | skipped
---
```
