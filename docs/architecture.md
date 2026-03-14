# Architecture

Codex Control Plane packages a role-specialized delegation system for Claude Code.

The idea is simple:

- Claude handles interaction, task framing, and orchestration.
- Codex handles the primary execution work through `codex exec`.
- The plugin adds routing, sandbox policy, verification, and reporting.

## Control Plane Layers

```text
capability layer   : codex exec
control plane      : route + sandbox + verify + result schema
distribution layer : Claude Code plugin
```

The capability layer will keep changing as model behavior improves. The control plane is the durable part: how work is routed, constrained, checked, and surfaced back to a human.

## Public v0.1

v0.1 exposes the smallest useful loop:

```text
research -> coding -> review
```

These three roles cover the most common high-value workflow:

- understand the codebase
- make a change from a plan
- inspect the result

## Full Six-Role Vision

```text
index    : where should I look?
research : what is true?
review   : what is wrong?
plan     : what should change?
coding   : do the change
general  : broad, manual-only expert delegation
```

### Role Matrix

| Role | Question | Primary output | Sandbox | Public in v0.1 |
| --- | --- | --- | --- | --- |
| `index` | Where should I look? | file map / navigational artifact | read-only | no |
| `research` | What is true? | research artifact | read-only | yes |
| `review` | Is it good? / What is wrong? | review artifact | read-only | yes |
| `plan` | What should change? | implementation plan | read-only | no |
| `coding` | Build this. | code changes + verification | workspace-write | yes |
| `general` | Handle this expert task. | task-dependent | danger-full-access or manual policy | no |

## Why Three First

This repo follows the product principle:

> 先卖 loop，再卖 lattice

Shipping the loop first keeps the surface area small and the story clear. Users do not need to learn six entry points before the system becomes useful.

## Verification-First Design

Every role follows the same control-plane shape:

1. intake and scope the request
2. dispatch the primary work to Codex
3. capture the `session_id`
4. verify the artifact or code change
5. resume the same session if gaps remain
6. return a structured result block

This is what turns delegation into an auditable workflow instead of a one-off prompt.

## Portability Rules

The public plugin intentionally strips private assumptions:

- no hardcoded repo aliases
- no personal absolute paths
- no dependency on a shared external protocol file
- no requirement for a specific documentation layout

Document skills default to writing artifacts in the current working directory unless the user gives an explicit output path or base directory.
