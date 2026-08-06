---
name: creating-hooks
description: Creates lifecycle hooks for Claude Code and Codex. Use when setting up automated behaviors triggered by agent lifecycle events (pre/post tool, permission requests, session start, stop, notifications), or when debugging a hook that does not fire.
---

STARTER_CHARACTER = 🪝

## Setup

First, update the reference docs to get the latest from Anthropic:
```bash
python ~/.claude/skills/creating-hooks/scripts/update-docs.py
```

## What Hooks Are

Shell commands that execute at lifecycle points in the agent. Unlike prompts, hooks are deterministic—they always run when triggered.

## Client support

Claude Code and Codex share this hook model: the same event names, the same `matcher` + `hooks[]` shape, the same exit-code semantics, and the same `hookSpecificOutput` JSON protocol. Everything in this skill applies to both. Only where the configuration lives and how it is trusted differ:

| | Claude Code | Codex |
|---|---|---|
| User scope | `~/.claude/settings.json` | `~/.codex/hooks.json` |
| Project scope | `.claude/settings.json` (shared), `.claude/settings.local.json` (not committed) | `.codex/hooks.json`, in a trusted project |
| Alternate format | — | inline in `config.toml`; never define the same layer in both JSON and TOML |
| Trust step | none | review or retrust with `/hooks` after changes, then restart if needed |
| Project-root variable | `$CLAUDE_PROJECT_DIR` | resolve paths absolutely |

**OpenCode does not have this hook system.** Its configuration has no `hooks` key; it extends behavior through JS plugins declared in the `plugin` array. Do not port a hook here — write a plugin instead.

### Codex workflow

1. Inspect active hook sources and `/hooks` trust state before editing.
2. Choose user scope (`~/.codex`) or trusted project scope (`.codex`).
3. Prefer `hooks.json` over inline TOML.
4. Use an external script for non-trivial logic, with absolute executable paths when PATH may be restricted.
5. Validate the JSON, run the handler against representative stdin, and assert the intended exit code and output shape.
6. Retrust with `/hooks`, then restart the session when necessary.

Codex matchers are regex strings: shell and unified-exec tools match `Bash`; file patches match `apply_patch`, `Edit`, or `Write`.

## Configuration

Hooks live in settings files:
- `~/.claude/settings.json` - User settings (all projects)
- `.claude/settings.json` - Project settings (shared via git)
- `.claude/settings.local.json` - Local project settings (not committed)

Codex uses the same block under the `hooks` key of `~/.codex/hooks.json`.

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolPattern",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here"
          }
        ]
      }
    ]
  }
}
```

**Matcher**: Pattern to match tool names (case-sensitive)
- Exact match: `Write`
- Regex: `Edit|Write`
- All tools: `*` or omit

**Environment variables**:
- `$CLAUDE_PROJECT_DIR` - Absolute path to project root
- `$CLAUDE_ENV_FILE` - File path for persisting env vars (SessionStart only)

## Hook Events

**Tool events** (matcher applies):
- `PreToolUse` - Before tool executes
- `PostToolUse` - After tool completes
- `PermissionRequest` - Permission dialog shown

**Session events**:
- `SessionStart` - Session begins/resumes (matcher: startup/resume/clear/compact)
- `SessionEnd` - Session ends
- `PreCompact` - Before compaction (matcher: manual/auto)

**Other events**:
- `UserPromptSubmit` - User submits prompt
- `Stop` - Agent finishes
- `SubagentStop` - Subagent finishes
- `Notification` - Alerts sent (matcher: notification type)

**Event coverage differs slightly**: Codex also fires `PostCompact` and `SubagentStart`; `Notification` is Claude Code only. Check the client before relying on an event at the edge of this list.

## Exit Codes

- **0**: Success. stdout shown in verbose mode. For `UserPromptSubmit`/`SessionStart`, stdout added to context.
- **2**: Block. stderr fed to Claude as error message. Blocks the action.
- **Other**: Non-blocking error. stderr shown to user.

## JSON Output

For advanced control, return JSON to stdout with exit code 0:

```json
{
  "continue": false,
  "stopReason": "Message shown when stopping"
}
```

### PreToolUse Control

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "allow",
    "permissionDecisionReason": "Auto-approved",
    "updatedInput": { "field": "modified value" }
  }
}
```

Decisions: `"allow"` (bypass permission), `"deny"` (block), `"ask"` (prompt user)

### PostToolUse Feedback

```json
{
  "decision": "block",
  "reason": "Explanation fed to Claude"
}
```

### Stop/SubagentStop Control

```json
{
  "decision": "block",
  "reason": "Must fix X before stopping"
}
```

## Hook Input

Hooks receive JSON via stdin:

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.jsonl",
  "cwd": "/current/dir",
  "permission_mode": "default",
  "hook_event_name": "PreToolUse",
  "tool_name": "Write",
  "tool_input": { "file_path": "/path", "content": "..." }
}
```

## Common Patterns

**Auto-format after edit**:
```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_input.file_path' | xargs -I{} sh -c 'echo {} | grep -q \"\\.ts$\" && npx prettier --write {}'"
      }]
    }]
  }
}
```

**Block dangerous commands**:
```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/validate-bash.py"
      }]
    }]
  }
}
```

**Inject context on prompt**:
```json
{
  "hooks": {
    "UserPromptSubmit": [{
      "hooks": [{
        "type": "command",
        "command": "echo '[REMINDER: Follow TDD]'"
      }]
    }]
  }
}
```

**Desktop notification**:
```json
{
  "hooks": {
    "Notification": [{
      "hooks": [{
        "type": "command",
        "command": "osascript -e 'display notification \"Claude needs input\" with title \"Claude Code\"'"
      }]
    }]
  }
}
```

## Hook Scripts

For complex logic, use external scripts. UV single-file format works well:

```python
#!/usr/bin/env -S uv run --script
# /// script
# dependencies = []
# requires-python = ">=3.11"
# ///

import json
import sys

data = json.load(sys.stdin)
tool_input = data.get("tool_input", {})

# Validation logic here

if should_block:
    print("Error message", file=sys.stderr)
    sys.exit(2)

sys.exit(0)
```

## Anti-Patterns

- Using exit code 2 without stderr message (Claude gets no feedback)
- Forgetting to handle JSON parsing errors in scripts
- Blocking without explaining why (Claude will retry the same thing)
- Long-running hooks without timeout (default is 60s)
- Modifying files in PreToolUse (use PostToolUse for modifications)

## Reference

- [references/anthropic-hooks.md](references/anthropic-hooks.md) - Complete reference (input schemas, prompt hooks, MCP tools)
- [references/anthropic-hooks-guide.md](references/anthropic-hooks-guide.md) - Quickstart and examples
