# Readable Tests and Focused Fixtures

## Arrange-Act-Assert

Every test should expose three phases, normally separated by blank lines:

1. Arrange inputs, collaborators and prerequisite state.
2. Act once on the behavior named by the test.
3. Assert the observable outcome and any relevant boundary interaction.

Use descriptive names that state one behavior. Multiple assertions are appropriate
when they describe that same behavior; multiple independent actions usually are not.

Split tests such as:

```text
nullAndEmptyInputsAndUnknownUsersAreSuccessfulNoOps
```

into independently named cases:

```text
nullInputIsASuccessfulNoOp
emptyInputIsASuccessfulNoOp
unknownUserLeavesExistingDataUnchanged
```

For the last case, arrange existing data so the assertion proves preservation,
not merely that an already empty database remains empty.

Keep the action out of ordinary assertions:

```java
List<Block> blocks = List.of(validBlock());

boolean stored = repository.store(blocks);

assertThat(stored).isTrue();
assertThat(database.storedProductIds()).containsExactly(PRODUCT_ID);
```

For exception cases, use the ecosystem's exception-capture idiom when it makes
Act and Assert clearer. Do not introduce complex scaffolding just to force a
visual layout. Trivial construction tests may omit an unnecessary Arrange phase.

Use parameterized tests for the same behavior over equivalent inputs, not to
combine unrelated scenarios. Give cases meaningful display names or labels when
the raw parameter does not explain the scenario.

## Extract Helpers Without Hiding the Test

Extract repeated domain values into meaningful constants. Prefer named status
constants to magic numbers. Keep one-off, self-explanatory values local.

Use small, cohesive helpers only where they improve readability:

- Object fixtures/builders: valid objects and explicit variants such as invalid dates.
- Request fixtures: stable payloads and endpoint constants.
- Database fixtures: seeding and typed queries.
- Infrastructure helpers: connection settings and lifecycle mechanics.

Keep expected outcomes and assertions visible in each test. Avoid hidden
assertions, universal test utility classes, inheritance-heavy harnesses, boolean
switches and unnecessary abstractions.

Arrange persistence fixtures directly through dedicated setup SQL or migrations
rather than invoking the repository operation being tested. Exceptions are
deliberate AT workflows where multiple application calls are the behavior itself.

Prefer narrow query methods such as `configurationCount()` and
`storedProductIds()` over arbitrary table-name SQL. Fail explicitly when fixture
queries return an impossible result; do not turn nulls or exceptions into zero
counts or success-shaped defaults.

Verify important construction and getter contracts directly when requested,
even if another component uses the object. Assert independent field values rather
than comparing two objects produced through the same potentially faulty path.
Do not add redundant tests solely to inflate coverage.
