---
name: writing-statuslines
description: Configures the agent status line and terminal title. Use when creating, customizing, or debugging a statusline — Claude Code status line scripts, or the Codex TUI footer via /statusline, /title and tui.status_line.
---

STARTER_CHARACTER = 📊

## Setup

Update the reference docs to get the latest from Anthropic:
```bash
python ~/.claude/skills/writing-statuslines/scripts/update-docs.py
```

## Client support

The three clients use fundamentally different models here. Pick the section that matches the client before writing anything.

| Client | Model | Section |
|---|---|---|
| Claude Code | arbitrary script; receives session JSON on stdin, first stdout line is the status line | most of this skill |
| Codex | built-in footer items only; **no arbitrary scripts** | [Codex](#codex) below |
| OpenCode | no status line configuration exists in its schema | not supported |

What carries across all of them is the design advice, not the mechanics: pick three to five fields, keep it glanceable in under a second, and don't duplicate information already visible elsewhere.

## What Status Lines Are

Custom scripts that display contextual information at the bottom of Claude Code's interface. Updated when conversation messages change, at most every 300ms.

## Configuration

Add to `~/.claude/settings.json` (user-level) or `.claude/settings.json` (project-level):

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 0
  }
}
```

`padding` is optional. Set to 0 to let the status line reach the terminal edge.

## How It Works

- Claude Code passes session context as JSON via stdin to the script
- First line of stdout becomes the status line text
- ANSI color codes supported
- Script must be executable (`chmod +x`)
- Only stdout is used (not stderr)

## JSON Input Schema

The script receives this via stdin:

```json
{
  "hook_event_name": "Status",
  "session_id": "abc123...",
  "transcript_path": "/path/to/transcript.json",
  "cwd": "/current/working/directory",
  "model": {
    "id": "claude-opus-5",
    "display_name": "Opus"
  },
  "workspace": {
    "current_dir": "/current/working/directory",
    "project_dir": "/original/project/directory"
  },
  "version": "1.0.80",
  "output_style": {
    "name": "default"
  },
  "cost": {
    "total_cost_usd": 0.01234,
    "total_duration_ms": 45000,
    "total_api_duration_ms": 2300,
    "total_lines_added": 156,
    "total_lines_removed": 23
  },
  "context_window": {
    "total_input_tokens": 15234,
    "total_output_tokens": 4521,
    "context_window_size": 200000,
    "used_percentage": 42.5,
    "remaining_percentage": 57.5,
    "current_usage": {
      "input_tokens": 8500,
      "output_tokens": 1200,
      "cache_creation_input_tokens": 5000,
      "cache_read_input_tokens": 2000
    }
  }
}
```

`context_window.current_usage` may be `null` if no messages have been sent yet.

## Key Fields

- `model.display_name` — short model name ("Opus", "Sonnet")
- `workspace.current_dir` / `workspace.project_dir` — may differ when working in subdirectories
- `cost.total_cost_usd` — cumulative session cost
- `context_window.used_percentage` / `remaining_percentage` — pre-calculated, ready to display
- `context_window.current_usage` — raw token counts from the last API call

## Constraints

- Output exactly one line
- Runs every 300ms at most — expensive operations must be cached
- Keep it scannable: glanceable in under a second
- Script must exit cleanly and quickly

## Anti-Patterns

- Cramming too much info — pick 3-4 data points max
- Not consuming stdin (script must read it even if it doesn't use all fields)
- Expensive uncached operations (git commands, API calls) on every invocation
- Multiple output lines (only first line is used)
- Forgetting `chmod +x`
- Writing to stderr instead of stdout

## Testing

Test scripts manually with mock JSON:
```bash
echo '{"model":{"display_name":"Sonnet"},"workspace":{"current_dir":"/test"},"cost":{"total_cost_usd":0.05},"context_window":{"used_percentage":42.5}}' | ./statusline.sh
```

## Codex

Codex uses built-in footer items rather than arbitrary status-line scripts. Nothing above this section applies — there is no script, no stdin JSON, and no stdout contract.

**Preferred workflow**

1. Run `/statusline` in the Codex TUI.
2. Select and reorder the desired footer items.
3. Confirm Codex persisted the result to `tui.status_line` in `~/.codex/config.toml`.
4. Use `/title` separately for terminal window or tab title items.

**Direct configuration**

```toml
[tui]
status_line = ["model-with-reasoning", "context-remaining", "current-dir", "git-branch"]
status_line_use_colors = true
```

Set `status_line = []` to hide the footer. Available items vary by Codex version and can include model, reasoning, context statistics, rate limits, git branch, token counters, session id, current directory, project root, and Codex version. Prefer the interactive picker when unsure of the current identifiers.

For the terminal title:

```toml
[tui]
terminal_title = ["spinner", "project", "git-branch"]
```

Use `/debug-config` when another configuration layer overrides the expected value.

## OpenCode

Not supported. OpenCode's configuration schema has no status line key. Don't port a Claude Code statusline script here.

## Reference

- [references/anthropic-statusline.md](references/anthropic-statusline.md) - Complete reference with examples in bash, python, and node
