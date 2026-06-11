---
name: testing-strategy
description: "Covers how to structure a test suite and write clear tests: the testing pyramid, isolation, intention-revealing naming, Arrange-Act-Assert, and common test smells. Use when writing or organizing tests, or deciding what to test."
---

STARTER_CHARACTER = 🔺🧪

# Testing Strategy

A test suite earns its keep by giving fast, trustworthy feedback. Bias toward many small, isolated tests of behaviour; reserve slow, wired tests for the few journeys that justify them.

## The shape of the suite

Three layers, roughly 70% unit / 20% integration / 10% E2E:

- **Unit** — fast, isolated, many. Business logic, domain rules, algorithms. No I/O, no wiring.
- **Integration** — real I/O. Persistence, external APIs, message queues. Not business rules; those belong in unit tests.
- **E2E / Acceptance** — slow, few. Critical user journeys only, not every edge case.

An inverted pyramid (mostly E2E, few unit) is the anti-pattern: slow suites, flaky signal, and failures that point everywhere at once.

## Isolation

- Unit tests carry no I/O, shared state, randomness, or clock. Same input, same result, every run.
- Integration tests may share infrastructure (a test database) but clean up after themselves.
- No test depends on another running first. Each sets up its own state.

A test that only passes in a certain order is leaking shared mutable state — fix the leak, don't reorder.

## Naming

Names read as a specification of behaviour, not a label for a method:

```
should_<expected_behaviour>_when_<condition>
```

`should_reject_bid_when_price_is_below_minimum` tells you what broke before you open the file. `test_bid_2` or `testPlaceBid` does not.

## Structure: Arrange – Act – Assert

Every test has three sections, in order: set up the context, trigger the behaviour, verify the outcome. Keep them separate — never interleave assertions with more setup or actions. When the Arrange block dwarfs the Act and Assert, treat it as a design smell in the code under test, not just the test.

## What not to test

Spending tests here adds maintenance cost without adding signal:

- Trivial getters/setters with no logic.
- Framework behaviour — that the DI container wires beans, that the ORM saves an entity.
- Third-party libraries themselves — test your *usage* of them at the integration level if it matters.
- Private methods directly — exercise them through the public interface. If one genuinely needs its own test, that is a signal to extract it into its own unit.

## Test smells

Catch and reverse these:

- A refactor that changes no behaviour breaks the test — the test is coupled to implementation, not behaviour.
- One test asserting many unrelated things — split into focused tests so a failure points to one cause.
- Setup longer than the test itself — the code under test is too coupled.
- Tests that pass only in a specific order — shared mutable state leaking between them.
- Everything mocked — the test stays green even when the real logic is broken. See the `nullables` skill for testing I/O without that trap.

## Fast feedback first

Unit and integration tests must be fast enough to run locally on every change. Reserve the slow E2E suite for CI rather than the inner loop.

## Related skills

- For the red-green-refactor process of writing code test-first, see the `tdd` skill.
- For analysing existing tests against Kent Beck's 12 properties, see the `test-desiderata` skill.
- For testing I/O (HTTP, files, DB, clocks) without mocks, see the `nullables` skill.
