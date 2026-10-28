---
name: branch-test-review
description: "Review the tests changed on the current branch, classify each as low, medium, or high value, list them, delete tests that provide no unique protection, and reject tests that only assert a removed path stays removed. Use when the user asks to review branch tests, audit test value, prune redundant tests, or classify tests by priority."
---

# Branch Test Review

Review the test changes on the current branch, present a value-ranked inventory, and remove the tests that are safe to delete.

## Scope

- Resolve the base branch from the user's request; otherwise use `main`.
- Run `git diff --stat <base>...HEAD -- '*_test.go'` and `git diff <base>...HEAD -- '*_test.go'`.
- Inventory added, modified, and renamed test functions, not just files. Include new cases added to an existing test as part of that test.
- Record a change type for every test: `Added`, `Modified`, `Renamed`, or `Deleted`. A test function with new cases is `Modified`; for a renamed test, include the old name (for example, `old name → new name`). A test removed during this review is `Deleted`.
- Account for helpers and fixtures that exist only to support a test being considered for deletion.
- For each added or modified test, trace every environment variable, flag, config key, endpoint, and command path it sets to a current production read. If the test's subject has no current production read, it is a removed-path test; see below.
- If the branch changes no tests, say so and stop.

## Removed-path tests

- Do not add a test whose subject is an input that current production code no longer reads, or whose only purpose is to prove that a deliberately removed capability stays removed. It documents history rather than current behavior, and a future reader sees an input with no production use and cannot tell why it matters.
- For example, a test that sets a deleted endpoint or API-key environment variable only to assert that it is ignored is a removed-path test.
- Record the removal in `CHANGELOG.md`, `AGENTS.md`, or the relevant documentation instead. If production still reads the input to reject or warn about it, test that rejection or warning.
- A one-time check that a removed input is ignored is not a durable regression test. If it already exists, classify it Low and remove it.

## Classify

Assign one level to each test:

- **High**: protects user-visible behavior, billing, security, data integrity, or a protocol or contract that crosses components. End-to-end tests and tests that would catch a plausible production regression in a behavior the product still supports are high.
- **Medium**: protects configuration validation, migrations, boundaries, or a layer that higher-level tests do not exercise deterministically. Keep it when it guards a real failure mode.
- **Low**: duplicates a named higher-level test, asserts only log field names or payload shape that is not a documented contract, covers a legacy path the user no longer requires, is a removed-path test, asserts the absence of behavior that never existed, or tests an implementation detail instead of behavior.

When unsure, prefer medium over low. Do not classify a test as low only because it is short, slow, or inconvenient. Do not classify a removed-path test as high solely because reverting the removal would make it fail; the behavior it would protect no longer exists.

## Report

Present a table with these columns:

| Level | Change | Test | Location | Why | Action |
| ----- | ------ | ---- | -------- | --- | ------ |

`Change` must be one of `Added`, `Modified`, `Renamed`, or `Deleted`; localize the labels when the user's language differs. Order high to low. For every low test, name the higher-level test that makes it redundant. For a removed-path test, name the changelog, AGENTS, or documentation entry that records the removal instead. Follow the user's language.

## Delete

- Delete only low tests whose coverage is genuinely duplicated or explicitly not required.
- Never delete the last test for a behavior, a concurrency or atomicity test, a migration test, or a test that guards a previously observed regression in behavior that still exists.
- A test that only asserts a deliberately removed behavior stays removed is a one-time check, not a regression guard; remove it. When a branch removes a capability, delete its old tests and do not add a replacement negative test that sets the removed input only to assert it is ignored.
- Remove helpers and fixtures used only by the deleted tests.
- Report removed tests with Change `Deleted`, and name the remaining test that retains the coverage, or for a removed-path test the changelog, AGENTS, or documentation entry that records the removal.
- Do not change production code in this workflow unless a deletion exposes an unused helper.

## Verify

- Run the narrowest test command for the affected package.
- Run the repository's check or lint task when one exists.
- Report what was deleted, which remaining test covers each behavior, and any low-value tests intentionally left in place with the reason.
