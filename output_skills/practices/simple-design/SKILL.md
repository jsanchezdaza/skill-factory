---
name: simple-design
description: Favors the simplest design that works—YAGNI and KISS over speculative abstraction. Use when designing a solution, deciding whether to add an abstraction, or when code feels over-engineered.
---

STARTER_CHARACTER = 🪒

# Simple Design

The simplest design that makes the tests pass and reveals intent is the target. Complexity is a cost paid on every future read and change — add it only when a concrete, present need forces it.

## Simplicity

- Write the least code that satisfies the current behaviour. Nothing speculative.
- If a solution feels clever, treat that as a smell, not an achievement. Clever reads as complex to the next person.
- Prefer a few extra lines of obvious code over an abstraction that hides what happens.
- Reveal intent in names and structure so the design needs no commentary to follow.

## YAGNI

- Build for the requirement in front of you, not the one you imagine arriving later.
- Defer every reversible decision to the last responsible moment — the point past which deciding gets meaningfully harder. Until then, keep options open by not deciding.
- Duplication is cheaper than the wrong abstraction. Three similar lines beat a premature framework that bends the moment the third case differs.
- Abstract on demonstrated repetition, not anticipated repetition. Two occurrences are a coincidence; wait for the pattern to actually recur before extracting it.

## Signals you have over-built

These are anti-patterns to catch and reverse, not a checklist to satisfy:

- An interface or base class with a single implementation, added "so we can swap it later".
- Configuration flags, strategy hooks, or parameters that no current caller sets to anything but the default.
- A generic layer (`*Manager`, `*Helper`, `*Service` of last resort) absorbing unrelated responsibilities.
- A plugin or extension mechanism built for one plugin.
- Parameterising a function for variation that no caller needs yet.
- Abstracting two call sites that are similar today but have no reason to stay identical.

When you spot one, collapse it back to the concrete case until a real second need appears.

## When the call is genuinely unclear

Bias toward the concrete, reversible choice. A duplicated block can be merged later in seconds; a wrong abstraction threaded through the codebase is expensive to unwind. Defer rather than guess.

## Related skills

- The simplicity discipline is paired with TDD — write the simplest code that makes the failing test pass. See the `tdd` skill.
- "Keep each change small and shippable" is its own discipline. See the `small-safe-steps` skill rather than duplicating it here.
- For pulling complexity out of code that already exists, see the `refactoring` and `code-simplifier` skills.
