---
name: trim-tests
description: Prune low-signal tests, optionally scoped to a time box.
disable-model-invocation: true
---

# Trim tests

Keep tests that protect meaningful public behavior and product risk. Remove false confidence and maintenance noise. Coverage is diagnostic, not a quota.

## Steps

1. Determine scope. If the user gave a time box, identify current tests touched by commits in it (`git log --since="<time box>"`) and stop if there are none. With no time box, scope to the entire test suite.
2. Read the repo's testing guidelines (`AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `docs/`) and inspect its test commands, fixtures, and setup. Use the project's existing tools and lifecycle.
3. Review every test in scope using the criteria below. Name the public behavior or risk, a plausible defect it should catch, and why its setup and assertions would catch it. Decide: keep, rewrite, combine, or delete. Record the reason for each decision; inspect related coverage before calling a test redundant.
4. Apply the decisions. Preserve meaningful protection when rewriting or combining. If sensitivity to a defect is uncertain, introduce a small reversible fault and check that the test fails for the claimed reason; restore the fault before continuing.
5. Run the repo's formatter, then its test suite. Report the decisions, validation results, and any unresolved gaps or failures. Account for every test reviewed.

## Review criteria

**Tests do not lie:** each test fails when the behavior it claims to protect breaks, and stays green through behavior-preserving changes. Remove or rewrite tests that lie:

- **Tautological:** the expected value restates or is derived from the implementation under test. Ground expectations in an independent contract, example, or invariant.
- **Structure-sensitive:** fails on a rename, reorder, or reformat without a behavior change. This includes reading application source files to assert implementation shape, or asserting private helper calls. Assert public results, returned errors/responses, and persisted outcomes.
- **Cannot-fail:** stubs replace the dependency whose failure modes the test claims to cover, so the claimed defect cannot reach the assertion. Exercise the relevant boundary; mocks are useful when they control inputs while leaving the behavior under test real.

Judge the claimed behavior, not the test's style: exact values or ordering can be public contracts, and a fixed expected example can protect a real calculation.

**Meaningful protection:** keep tests for important user journeys and public module contracts, including authorization, domain invariants, and rollback where relevant. Test count, coverage percentage, and one case per helper or CRUD branch do not establish value. Simple pass-through wiring and private refactoring already covered elsewhere often need no dedicated test.

Prefer the public boundary that exposes the risk using the consuming repo's conventions. Keep framework choices, route-testing rules, and network/database setup project-specific. Preserve production interfaces while improving tests; incidental glue does not need a new public API.

**Disposition:** rewrite a lying test when its claimed protection matters and is otherwise missing. Combine overlapping cases when their distinct risks remain protected. Delete tests with no meaningful risk or whose protection is already supplied by an effective test. Keep uncertain cases and report what evidence is missing.
