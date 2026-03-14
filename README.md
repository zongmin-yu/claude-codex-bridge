# claude-codex-bridge

Use [Codex CLI](https://github.com/openai/codex) as a sub-agent inside [Claude Code](https://claude.com/claude-code) skills.

Claude excels at conversation and alignment. Codex excels at deep reasoning and code generation. This bridge lets you combine both — Claude orchestrates, Codex executes.

## Install

```bash
claude plugin install github:zongmin-yu/claude-codex-bridge
```

Then use `/codex` in any Claude Code conversation:

```
/codex investigate why the auth tests are flaky in src/auth/
```

## The Pattern

Every Codex-powered skill follows three steps:

### 1. Build the prompt

Turn the user's request into a self-contained instruction for Codex. Point to files by absolute path — don't paste contents.

### 2. Dispatch

```bash
codex exec --full-auto --sandbox read-only \
  -C "/path/to/repo" \
  "Your prompt here" 2>/dev/null
```

Pick a sandbox level:
| Level | Can read | Can write | Has network | Use for |
|-------|----------|-----------|-------------|---------|
| `read-only` | Yes | No | No | Research, review |
| `network-only` | Yes | Yes | No | Coding |
| `full` | Yes | Yes | Yes | Tasks needing network |

### 3. Verify

Check that Codex produced what was asked. If something is missing, resume:

```bash
echo "Also check the edge case for empty input" | codex exec resume <session-id> 2>/dev/null
```

That's it. The `/codex` skill included in this plugin implements exactly this pattern.

## Build Your Own

The real power is creating specialized skills for your workflow. Copy `examples/template.md` to `skills/<name>/SKILL.md` and fill in:

- What arguments your skill takes
- How to build the prompt for Codex
- What sandbox level to use
- How to verify the result

See `examples/` for reference implementations:
- [`research.md`](examples/research.md) — investigate codebases
- [`coding.md`](examples/coding.md) — implement features
- [`review.md`](examples/review.md) — code review

## Advanced

### Session Resume

Every `codex exec` prints a session ID. Save it to continue a conversation:

```bash
echo "Follow-up question" | codex exec resume <session-id> 2>/dev/null
```

### Reasoning Effort

For complex tasks, increase reasoning:

```bash
codex exec --full-auto --sandbox read-only \
  --config model_reasoning_effort="high" \
  -C "/path/to/repo" "Your prompt" 2>/dev/null
```

### Cross-Repo Work

Point Codex at any directory. One skill can work across multiple repos by dispatching separate `codex exec` calls with different `-C` paths.

## Requirements

- [Claude Code](https://claude.com/claude-code) installed
- [Codex CLI](https://github.com/openai/codex) installed (`npm install -g @openai/codex`)
- `OPENAI_API_KEY` set in your environment

## License

MIT
