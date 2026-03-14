# Codex Control Plane

Reviewable Codex delegation for Claude Code.

**Philosophy:** `先卖 loop，再卖 lattice`  
Sell the repeatable delegation loop first. Expand to the wider role lattice later.

Codex Control Plane is a Claude Code plugin for people who already like both tools and want cleaner orchestration between them. Claude stays in the alignment and interaction seat. Codex does the primary execution work. The plugin adds role routing, least-privilege sandboxes, verification, and consistent result reporting.

## Three Questions

v0.1 ships three public skills:

- `codex-control-plane:research` answers "What's here?" and "What is true?" for a codebase or technical question.
- `codex-control-plane:review` answers "Is it good?" for code, diffs, branches, or modules.
- `codex-control-plane:coding` answers "Build this." from a plan file with verification against acceptance criteria.

The larger architecture is documented in [docs/architecture.md](docs/architecture.md). The dispatch contract is documented in [docs/protocol.md](docs/protocol.md).

## Why This Exists

This project is not trying to outcompete the capability layer. `codex exec` is the capability layer. The value here is the control plane on top:

- role-specialized routing
- least-privilege sandboxes by role
- explicit verify loops
- resumable Codex sessions
- predictable artifacts and result blocks

That makes delegation easier to inspect, repeat, and trust.

## Prerequisites

- Claude Code installed and authenticated
- Claude Code 1.0.33 or later
- Codex CLI installed, authenticated, and available as `codex` on your `PATH`

## Install

For local development or testing, run Claude Code with the plugin directory directly:

```bash
claude --plugin-dir /absolute/path/to/codex-control-plane
```

Then use the namespaced skills:

```text
/codex-control-plane:research ...
/codex-control-plane:review ...
/codex-control-plane:coding ...
```

When you publish this repo through a marketplace, users can install it with:

```bash
claude plugin install codex-control-plane@<marketplace-name>
```

## Quick Start

### 1. What's in this codebase?

```text
/codex-control-plane:research /absolute/path/to/repo Trace how authentication tokens are refreshed.
```

By default the skill writes a markdown artifact in the current working directory. If you want a different location, tell Claude the exact output path or base directory.

### 2. Is this code good?

```text
/codex-control-plane:review /absolute/path/to/repo Review the latest diff for correctness, regressions, and missing tests.
```

The review skill produces a report with scoped findings, severity, evidence, and priority actions.

### 3. Build this.

```text
/codex-control-plane:coding /absolute/path/to/plan.md
```

The coding skill reads the plan first, dispatches implementation to Codex with `workspace-write`, and verifies the result against the plan before reporting back.

## How The Loop Works

The public workflow is a simple, reviewable loop:

1. `research` clarifies what is true.
2. `coding` changes the code from a plan.
3. `review` checks the result for bugs, risks, and missing coverage.

That loop is the product in v0.1. The wider role lattice comes later.

## What This Is Not

- Not a generic one-shot `/codex` wrapper.
- Not a hidden adapter for one person's private repo layout.
- Not an autopilot system or danger-mode release.
- Not a replacement for human review or project-specific conventions.

## Architecture

See [docs/architecture.md](docs/architecture.md) for the full six-role vision and which roles are public in v0.1.

## Protocol

See [docs/protocol.md](docs/protocol.md) for the clean reference version of the dispatch, verify, and result contract. Each shipped skill also inlines the protocol so the plugin remains fully self-contained after installation.

## Sources

The install and plugin-layout guidance used in this repo comes from the official Claude Code plugin docs:

- https://code.claude.com/docs/en/plugins
- https://code.claude.com/docs/en/plugins-reference
