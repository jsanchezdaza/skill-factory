---
name: claude-vault
description: Reads, creates, edits, and captures notes in the user's Wallapop Obsidian vault at ~/Documents/Wallapop, enforcing its layout (plans/, proposals/, spikes/, personal/) and conventions (kebab-case filenames, no frontmatter, H1 titles). Use when the user mentions a note, plan, proposal, spike, vault, or Obsidian document; asks to look something up in the vault; or wants to save a plan.
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - AskUserQuestion
---

STARTER_CHARACTER = 📓

# Wallapop Vault

Manages notes in the user's **Wallapop** Obsidian vault, enforcing its established conventions.

**Vault root**: `/Users/javier.sanchezdaza/Documents/Wallapop`

This is a personal **Obsidian vault of work notes** — plans, proposals, spikes, and personal goals. It is **not** a software project (no code to build/run) and **not** the source of truth for the agent's behavior: hard rules and practices live in the agent instruction files — `~/.claude/CLAUDE.md` and per-project `CLAUDE.md` for Claude Code, `AGENTS.md` for Codex and OpenCode — not here. Treat this vault as documentation.

The vault is outside normal repository workspaces. Keep every operation scoped to the vault root.

## When to use / Do NOT use

**Use:**
- Reading or searching notes (plans, proposals, spikes, personal goals)
- Creating or editing notes in the vault
- Saving a plan-mode plan, a proposal, or a spike

**Do NOT use:**
- Source code files or git repositories of actual software projects
- Any path outside `~/Documents/Wallapop`

---

## Vault map

Flat, single-tier structure:

| Path | Purpose |
|---|---|
| `plans/` | Technical implementation plans (often with embedded Kotlin snippets) — **primary area for new plans**. |
| `proposals/` | Design docs / drafts meant to be shared with the team for review. |
| `spikes/` | Time-boxed exploratory investigations and findings. |
| `personal/Goals.md` | Personal goals and private notes. |
| `bug-fixes/` | Notes on investigated bugs. Read and search it, but don't send new notes here unless the user asks. |

Default for new notes: `plans/`. Use `proposals/` when the user says "proposal", `spikes/` when the user says "spike".

---

## Conventions (hard rules)

- **Filenames**: kebab-case, English, 2–5 words, action-oriented. No dates, no spaces. `Goals.md` is the only fixed-name exception.
- **First line**: always `# <Title>` (H1).
- **NO YAML frontmatter** — never open a note with a `---` YAML block.
- **NO tags** — no `#tag` in body, no `tags:` field.
- **NO checkboxes** (`- [ ]`) unless the user explicitly requests a task list.
- **Wikilinks**: `[[note-name]]` without `.md` extension — only when linking genuinely related sibling notes, never artificially. A `> Related:` line near the top is common for sequencing dependent plans; preserve it.
- Separators `---`, Markdown tables, and code fences are fine.

---

## Operations

### Read

Search by filename:
```bash
find /Users/javier.sanchezdaza/Documents/Wallapop -iname "*<keyword>*" -type f -not -path '*/.obsidian/*'
```

Search by content:
```bash
grep -ril "<keyword>" /Users/javier.sanchezdaza/Documents/Wallapop --include='*.md'
```

Resolve a `[[wikilink]]` → look in `plans/<wikilink>.md` first, then the rest of the vault.

### Create

1. Confirm folder (default: `plans/`; `proposals/` for proposals; `spikes/` for spikes).
2. Validate filename: kebab-case, 2–5 words, no date prefix.
3. Scaffold a plan with this skeleton:

```markdown
# <Title>

## Context

## Changes

## Verification
```

Add `## Risks` or `## Open questions` only when the user's content warrants them.

### Edit

Read the note first with the Read tool. Preserve all existing conventions — never add frontmatter, tags, or checkboxes to a note that doesn't have them. Preserve and reuse existing `[[wikilinks]]` and `> Related:` lines when editing or adding notes.

---

## Anti-patterns

- Opening any user-authored note with a `---` YAML frontmatter block.
- Filenames with Title Case (`My-Plan.md`), date prefixes (`2026-05-22-plan.md`), or spaces.
- Adding `#tags`, `tags:`, or `- [ ]` checkboxes unprompted.
- Creating a plan outside `plans/` without a clear reason.
- Generating wikilinks artificially to notes that aren't related.
- Looking for rules/practices/memory here — those live in the agent instruction files (`CLAUDE.md` / `AGENTS.md`).
