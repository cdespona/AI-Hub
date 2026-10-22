---
name: legacy-test-safety-net
description: Add or refactor a maintainable characterization test safety net for an existing application. Use when the user asks for tests or coverage on untested code, unit/integration/acceptance tests, mutation testing with PIT, test stability, reducing test overlap, clearer AAA tests, shared fixtures, a balanced test pyramid, or build-managed database test infrastructure. Preserve current behavior, keep acceptance tests focused, and verify coverage without inflating it.
---

# Legacy Test Safety Net

Make existing behavior safe to change through understandable, deterministic tests.
Characterize before modifying production behavior. Prefer existing seams and the
least invasive implementation; do not turn a testing task into a rewrite.

## Workflow

Read the linked reference before each applicable step. Load optional references
only when needed; keep detailed rules and examples in their owning reference.

1. **Load the upstream method.** Follow
   [upstream retrieval](references/remote-legacy-code-skill.md) before starting.
2. **Inspect and map behavior.** Follow
   [characterization discovery](references/characterization-discovery.md).
   Preserve worktree changes, measure a baseline and choose existing seams.
   For optional graph-assisted exploration, use
   [test discovery](references/graphify-test-discovery.md).
3. **Assign test responsibilities.** Follow
   [layers and overlap](references/test-layer-overlap.md). Choose the lowest useful
   layer; retain distinct boundary evidence rather than enforcing test-count ratios.
4. **Write focused tests.** Follow
   [test writing and fixtures](references/test-writing-and-fixtures.md).
   Keep one behavior per test and expected outcomes visible.
5. **Manage external fixtures when needed.** Follow
   [integration infrastructure](references/integration-infrastructure.md).
   Use isolated, disposable resources and document fidelity limits.
6. **Verify and hand off.** Follow
   [verification and CI](references/verification-and-ci.md). Preserve coverage
   gates, assess stability separately and document honest limitations.
   When adding mutation analysis or verifying test-refactoring effectiveness, also
   follow [mutation testing](references/mutation-testing.md).

## Guardrails

- Preserve observed behavior; separate intentional production changes from tests.
- Ask before material tooling, infrastructure or scope decisions.
- Never remove protection or broaden exclusions merely to obtain a green result.
- Do not claim complete coverage, stability or production fidelity from a score.
