# Test-Layer Responsibilities and Overlap

Use this reference when adding a safety net or reviewing existing UT/IT pairs.
The objective is distinct regression protection at the cheapest useful layer,
not fewer tests for their own sake.

## Audit by Evidence, Not by Name

For each scenario, record the action, real collaborators, controlled collaborators,
asserted outcome and regression it can detect. This can be a working comparison;
do not create a tracking document unless requested.

The same input in two layers is not automatically duplication:

| Evidence | Best home |
|----------|-----------|
| A null/empty input never calls persistence | UT with explicit no-interaction assertion |
| Every input field becomes the correct boundary parameter | UT with captured arguments |
| A driver or database exception maps to the current application result | UT with a controlled exception |
| SQL executes correctly, converts dates and links persisted rows | IT against the actual engine |
| Quoted values are bound literally and unrelated rows are preserved | Database IT with distinguishing fixtures |
| JSON binds to the application request and changes persisted state | Focused HTTP-adapter IT |
| Status mapping for all service outcomes | Controller UT |
| Actual HTTP transport, application configuration and essential workflows | Small AT suite |

Repository adapters often justify substantial IT coverage because their behavior
lives in SQL. Do not force database contracts into mocks to improve a UT/IT ratio.

## Keep a Deliberate Test Pyramid

| Layer | Responsibility | Typical approach |
|-------|----------------|------------------|
| UT | Detailed behavior of individual units | Direct construction, controlled collaborators, no external infrastructure |
| IT | Contracts between real components or an external boundary | MVC request handling, serialization, SQL against the actual database engine |
| AT | Essential user-visible workflows through the application | Real entry point and persistence, substituting only unrelated external services |

Keep UT broadest, IT narrower and AT smallest. Do not manufacture tests to meet
counts or a fixed ratio: choose the cheapest layer that detects the regression.
Count parameterized invocations when comparing the actual distribution.

Acceptance tests should go straight to the matter:

- A representative successful workflow for each essential capability.
- A representative unsuccessful workflow proving failure crosses the stack.
- Additional AT only when it protects a distinct cross-layer risk.

Do not enumerate malformed payloads, every empty/null variant, all field mappings,
all status codes or every infrastructure error in AT. Move those assertions to UT
or focused IT. Do not repeat a detailed matrix across all three layers.

When moving tests down the pyramid, retain the original behavioral assertions.
Avoid deleting edge-case protection merely to reduce the acceptance count.
Remove unrelated smoke checks unless they protect an explicitly required contract.

## Remove Redundancy Without Losing Protection

1. Find the lowest layer that already proves the same behavior.
2. Compare assertions, not only scenario names. Move missing assertions before
   deleting a higher-layer duplicate.
3. Keep the higher-layer case if it protects a distinct binding, configuration,
   transport, database or cross-component risk.
4. Remove Java-only no-op cases from database IT when UT already proves both the
   return value and absence of database access.
5. Keep small representative success/failure paths across boundaries; do not
   replicate the complete failure matrix in IT and AT.

An empty HTTP list can still deserve IT even when a repository empty-list guard
belongs only in UT: request deserialization and endpoint behavior are distinct.

## Choose Mocks Deliberately

Mocking the same collaborator in UT and IT is a review signal, not an automatic
defect. A focused MVC slice can legitimately isolate HTTP conversion. State what
it does not prove; a standalone MVC harness is not proof of full Spring wiring.

If the purported integration test only repeats mocked service outcomes already
covered by UT, replace it with a distinct boundary scenario or remove it.
When infrastructure is already available and the goal is component integration,
consider a real controller, parser, service and repository against a disposable
database. Assert resulting state rather than mock invocations. Do not expand every
HTTP slice into a full-stack test by default; weigh fidelity against runtime and
retain AT for full configuration and transport.

WireMock and similar servers simulate downstream HTTP dependencies. They do not
make inbound endpoint tests more realistic when no downstream HTTP API exists.
Never stub the endpoint under test and claim its implementation was exercised.
For database boundaries use the actual engine rather than an HTTP stub.

## Strengthen Assertions Before Reducing Test Count

- Assert all fields promised by the test name, including every nullable parameter,
  required keys, collection size and relevant ordering.
- Avoid comparing objects built through the same potentially faulty mapping path.
- Test relationships explicitly: matching row counts do not establish correct
  foreign-key links, and a join result alone need not establish distinct IDs.
- For preservation cases, seed both targeted and unrelated records. Distinguish
  parent deletion, child deletion and joined visibility; a missing join row is
  not evidence that the child was deleted.
- Assert partial persistence after a batch failure if that is observed behavior.
  Do not assume atomicity merely because a method returns failure.
- Avoid copying SQL statements into UT expectations. Verify parameter binding and
  application handling there; verify predicates and SQL effects against the engine.
- Keep database helper queries typed and narrow, with assertions in the test.

Do not remove engine-specific negative scenarios simply because UT injects an
exception. A real date-conversion rejection and a simulated connectivity failure
prove different things.

## Verify the Refactoring

Keep production behavior unchanged and preserve unrelated worktree edits.
Run affected layers and existing combined coverage gates. Verify that lower-layer
tests are actually selected and that removed cases did not leave boundary gaps.
Use the additional verification procedure in `mutation-testing.md`.

Document which behaviors each layer owns and which schema or external-service
assumptions remain unverified. Do not claim that passing tests establish fidelity
to a production schema that is absent from the repository.
