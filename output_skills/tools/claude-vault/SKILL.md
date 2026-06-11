---
name: claude-vault
description: Reads, creates, edits, and captures notes in the user's Claude config / Obsidian vault at ~/Documents/Claude, enforcing its two-tier (global/ vs wallapop/) layout and conventions (kebab-case filenames, no frontmatter, H1 titles). Use when the user mentions a note, plan, proposal, vault, or Obsidian document; asks to look something up in the vault; or wants to save a plan.
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - AskUserQuestion
---

STARTER_CHARACTER = 📓

# Claude Vault

Manages notes in the user's Claude configuration vault (also an Obsidian vault), enforcing its established conventions.

**Vault root**: `/Users/javier.sanchezdaza/Documents/Claude`

This vault is **not** a generic notebook — it is the curated source of truth for how Claude behaves. Files under `global/` are `@`-imported into every Claude Code session via `~/.claude/CLAUDE.md`, so editing them changes Claude's own instructions. Treat edits to rules/practices/memory as behavior changes, not documentation tweaks.

## When to use / Do NOT use

**Use:**
- Reading or searching notes (plans, proposals, rules, practices, memory, goals)
- Creating or editing notes in the vault
- Saving a plan-mode plan or a proposal

**Do NOT use:**
- Source code files or git repositories of actual software projects
- Any path outside `~/Documents/Claude`

---

## Vault map

Two-tier structure. The **more specific tier wins** for its scope; `wallapop/` files only carry what is genuinely Wallapop-specific (never restate global rules).

| Path | Purpose |
|---|---|
| `global/RULES.md` | Cross-project hard rules (git/commit/PR conventions, English-in-repo). Imported every session. |
| `global/MEMORY.md` | Cross-project memory **index** — one line per memory, never content. |
| `global/memory-policy.md` | Policy separating the curated vault from transient auto-memory. |
| `global/practices/engineering-practice.md` | Durable engineering practices (TDD, XP, naming, error handling). |
| `global/practices/testing-strategy.md` | Durable testing strategy (pyramid, isolation, naming). |
| `wallapop/RULES.md` | Wallapop overlay rules (branch naming `WPA-XXXXXX_...`, Jira-first flow, where plans go). |
| `wallapop/MEMORY.md` | Wallapop memory **index** — one line per memory, never content. |
| `wallapop/plans/` | Plan-mode artifacts — **primary area for new plans**. |
| `wallapop/proposals/` | Design docs meant to be shared with others. |
| `wallapop/personal/Goals.md` | Personal goals. |

Default for new notes: `wallapop/plans/`. Use `wallapop/proposals/` when the user says "proposal".

---

## Conventions (hard rules)

- **Filenames**: kebab-case, English, 2–5 words, action-oriented. No dates, no spaces. Fixed top-level notes (`MEMORY.md`, `RULES.md`, `Goals.md`, `memory-policy.md`) are the only exceptions.
- **First line**: always `# <Title>` (H1).
- **NO YAML frontmatter** — never open a note with a `---` YAML block.
- **NO tags** — no `#tag` in body, no `tags:` field.
- **NO checkboxes** (`- [ ]`) unless the user explicitly requests a task list.
- **Wikilinks**: `[[note-name]]` without `.md` extension — only when linking genuinely related sibling notes, never artificially.
- **MEMORY.md files are indexes only** — one line per memory, never the memory content itself.
- Separators `---`, Markdown tables, and code fences are fine.

---

## Operations

### Read

Search by filename:
```bash
find /Users/javier.sanchezdaza/Documents/Claude -iname "*<keyword>*" -type f -not -path '*/.obsidian/*'
```

Search by content:
```bash
grep -ril "<keyword>" /Users/javier.sanchezdaza/Documents/Claude --include='*.md'
```

Resolve a `[[wikilink]]` → look in `wallapop/plans/<wikilink>.md` first, then the rest of the vault.

### Create

1. Confirm folder (default: `wallapop/plans/`; `wallapop/proposals/` for proposals).
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

Read the note first with the Read tool. Preserve all existing conventions — never add frontmatter, tags, or checkboxes to a note that doesn't have them.

When appending to a `MEMORY.md` or `RULES.md`, add content under the relevant `## Section` header; never rewrite the whole file. Remember `MEMORY.md` files hold only one-line index entries, not memory bodies.

Editing anything under `global/` changes Claude's behavior in every future session — flag this to the user before doing it unless they explicitly asked.

---

## Anti-patterns

- Opening any note with a `---` YAML frontmatter block.
- Filenames with Title Case (`My-Plan.md`), date prefixes (`2026-05-22-plan.md`), or spaces.
- Adding `#tags`, `tags:`, or `- [ ]` checkboxes unprompted.
- Creating a plan outside `wallapop/plans/` without a clear reason.
- Restating a global rule inside a `wallapop/` file instead of letting the global tier own it.
- Putting memory content (not just an index line) into a `MEMORY.md`.
- Generating wikilinks artificially to notes that aren't related.
- Rewriting `MEMORY.md` or `RULES.md` in full instead of appending to the right section.
