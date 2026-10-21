---
name: branch-test-review
description: "Review the tests changed on the current branch, classify each as low, medium, or high value, list them, and delete tests that provide no unique protection. Use when the user asks to review branch tests, audit test value, prune redundant tests, or classify tests by priority."
---

# Branch Test Review

Review the test changes on the current branch, present a value-ranked inventory, and remove the tests that are safe to delete.

## Scope

- Resolve the base branch from the user's request; otherwise use `main`.
- Run `git diff --stat <base>...HEAD -- '*_test.go'` and `git diff <base>...HEAD -- '*_test.go'`.
- Inventory added, modified, and renamed test functions, not just files. Include new cases added to an existing test as part of that test.
- Account for helpers and fixtures that exist only to support a test being considered for deletion.
- If the branch changes no tests, say so and stop.

## Classify

Assign one level to each test:

- **High**: protects user-visible behavior, billing, security, data integrity, or a protocol or contract that crosses components. End-to-end tests and tests that would catch a plausible production regression are high.
- **Medium**: protects configuration validation, migrations, boundaries, or a layer that higher-level tests do not exercise deterministically. Keep it when it guards a real failure mode.
- **Low**: duplicates a named higher-level test, asserts only log field names or payload shape that is not a documented contract, covers a legacy path the user no longer requires, asserts the absence of behavior that never existed, or tests an implementation detail instead of behavior.

When unsure, prefer medium over low. Do not classify a test as low only because it is short, slow, or inconvenient.

## Report

Present a table with these columns:

| Level | Test | Location | Why | Action |
| ----- | ---- | -------- | --- | ------ |

Order high to low. For every low test, name the higher-level test that makes it redundant. Follow the user's language.

## Delete

- Delete only low tests whose coverage is genuinely duplicated or explicitly not required.
- Never delete the last test for a behavior, a concurrency or atomicity test, a migration test, or a test that guards a previously observed regression.
- Remove helpers and fixtures used only by the deleted tests.
- Do not change production code in this workflow unless a deletion exposes an unused helper.

## Verify

- Run the narrowest test command for the affected package.
- Run the repository's check or lint task when one exists.
- Report what was deleted, which remaining test covers each behavior, and any low-value tests intentionally left in place with the reason.
