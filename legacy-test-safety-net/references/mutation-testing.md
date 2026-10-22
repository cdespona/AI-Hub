# Mutation Testing as Safety-Net Evidence

Use mutation testing to assess whether tests detect changes to application behavior,
not merely execute lines. For JVM projects, PIT is an option; use the ecosystem's
native tool elsewhere. Do not impose Gradle, Java or a particular framework on an
unrelated project.

## Keep Three Questions Separate

| Question | Evidence |
|----------|----------|
| Was the code executed? | Line/branch coverage |
| Did assertions detect the generated behavior changes? | Mutation analysis |
| Are outcomes reproducible? | Repeated unmodified runs with isolated state |

Mutation testing does not prove tests are non-flaky. A flaky failure can falsely
appear to kill a mutant. Repeated green runs do not prove absence of flakiness
either. Report these forms of evidence separately.

## Establish an Honest Baseline

- Run the unmodified affected suites first. Do not skip failing baseline tests or
  use retries to turn failures green.
- For stability evidence, repeat representative unmodified suites with result
  caching disabled and fixtures recreated/reset between runs. Use a bounded,
  documented repeat count; fail the command on the first failure.
- Keep fixed fixture data, readiness checks, isolated resources and bounded I/O.
  Diagnose environmental failures separately from behavioral regressions.
- Capture the initial mutation report before changing tests or exclusions.
- Confirm runtime, build-plugin, engine and test-framework compatibility using
  current authoritative documentation. Do not assume a successful task means the
  intended tests ran.

## Integrate with the Existing Build

For Gradle/PIT:

- Use the maintained Gradle PIT plugin and the appropriate test-framework adapter.
  A JUnit Platform project needs the compatible PIT JUnit adapter and launcher.
  Keep versions in the existing version catalog or dependency-management mechanism.
- Configure production targets and test selection explicitly. PIT launches its own
  test JVMs; do not assume Gradle `Test` tag filters, system properties or JVM
  settings automatically carry over.
- Pass fixture connection settings to child JVMs, including any port used to
  construct connection details within tests.
- Expose a separately runnable task and human-readable plus machine-readable
  reports at predictable paths. Ask before making an expensive task part of
  ordinary verification or CI.
- Fail when no mutations are generated unexpectedly and do not skip failing tests.
- For a full verification run, force fresh analysis and avoid stale task outputs
  or mutation history. Incremental history can speed routine work but is not a
  substitute for a fresh final comparison.
- Keep fixtures alive until analysis completes. Verify cleanup ordering when
  baseline tests, coverage tasks and mutation tasks share one database.
- Use one worker for shared schemas unless per-worker isolation is implemented.
  A JVM-local lock does not coordinate separate mutation worker processes.

Separate baseline prerequisites from mutation-test selection. A full-stack suite
may run unmodified before analysis while remaining excluded from mutant execution.
Explain that dependency clearly; it is an optional confidence policy, not a
requirement of the mutation engine.

## Select Tests by Execution Model

UT/IT/AT names alone do not determine suitability. A repository IT executing
application code in the instrumented JVM can be useful even with an external
database. Tests against another application process cannot be assumed to exercise
the instrumented classes.

For Spring:

- Plain unit tests and in-process adapter tests avoid many context-lifecycle risks.
- Context caching can mask changes in startup and factory methods. A context that
  started successfully earlier is not evidence that mutated startup code ran.
- Prefer direct unit tests for meaningful configuration behavior: configured
  identifier/type selection, returned values, datasource settings, URL construction,
  JSON options and propagation of loading failures. Mock only the external adapter
  needed to avoid real credentials or network calls.
- `@DirtiesContext` alone is not a universal fix; caches outside the context and
  execution on other threads/processes may matter. Follow current tool support.
- Arcmutate's Spring documentation explicitly excludes integration tests that
  launch a web server, even with its plugin. Conservatively exclude such tests
  from PIT analysis; retain real HTTP AT as ordinary regression/baseline checks.
  Do not present a green PIT run containing them as supported mutation evidence.

The Arcmutate Spring plugin clears additional Spring state and offers optional
annotation operators, but requires a licence according to its documentation.
An advisory that it is absent does not mean free PIT cannot test Spring application
logic. Do not add commercial dependencies, require a licence or promise that buying
one removes web-server limitations. When free tooling is required, use supported
in-process tests and direct configuration tests.

## Set Scope Deliberately

Inspect every survivor and uncovered target before excluding anything.

- Trivial object construction/dependency wiring may reasonably be out of scope.
- A bootstrap-only class containing entry points and servlet initialization can be
  excluded by its exact class name if deployment checks own that boundary.
- Do not exclude a whole configuration package or all `@Bean` methods: factories
  can contain application behavior worth asserting.
- Keep semantic configuration and error handling in scope; add direct tests when
  removing a full-stack suite exposes their missing coverage.
- PIT method exclusions match method-name patterns globally. Avoid broad names
  such as `configure` or `create`; prefer exact bootstrap-class exclusions or another
  supported narrowly scoped mechanism. Check all matching methods before adding
  any method-name exclusion and document its global effect.
- Treat exclusions as documented ownership decisions, not a way to reach 100%.
  Distinguish excluded code from code tested successfully.
- Keep mutation exclusions separate from line/branch coverage exclusions. Do not
  silently weaken existing coverage gates.

After an exclusion change, inspect the generated report: intended mutations should
be absent, while meaningful neighboring methods must remain in scope.

## Interpret Results and Repair Gaps

| Outcome | Response |
|---------|----------|
| Killed | A test detected this generated change; confirm the failure is meaningful |
| Survived | Covering tests did not detect it; inspect assertions, execution and equivalence |
| No coverage | No selected test exercised the target; inspect selection and missing test points |
| Timed out | Investigate mutation-induced loops versus environmental slowness |
| Run/memory error or non-viable | Investigate execution/tooling; do not present these as assertion success |

Report killed, survived and uncovered counts separately, with execution anomalies.
"Zero survivors" alone is misleading when uncovered mutants remain. A build can
succeed with survivors if no score gate is configured.

For a survivor, reproduce its behavioral difference and add the smallest test that
asserts an observable contract. Do not assert implementation trivia merely to kill
it. Document genuinely equivalent mutations or intentional scope boundaries with
evidence; do not assume every survivor is equivalent.

When introducing a score gate, use measured results and team policy. A report-only
starting point can be appropriate, but state explicitly that it is not a gate.
Never lower a threshold, change operators or exclude meaningful behavior solely
to make verification pass.

## Compare Before and After Test Refactoring

When removing overlap or moving scenarios between layers:

1. Save a fresh machine-readable baseline and record engine, operators, scope,
   production revision and outcome counts.
2. Preserve production behavior and analysis scope during the test-only change.
3. Run all affected unmodified suites and existing coverage gates.
4. Run fresh mutation analysis and compare mutation identities, not just percentages.
   Use class, method/signature, operator and instruction index/location where
   available. Account for changed source locations if production was edited.
5. Verify each previously killed in-scope mutant is still killed. A smaller
   denominator can conceal weakened tests behind the same score.
6. Explain any changed mutation set, uncovered targets or anomalies rather than
   claiming an unchanged score proves equivalent protection.

If scope changes intentionally, compare the retained mutation set and separately
list the removed targets and ownership rationale.

## Know What a Perfect Score Does Not Prove

A 100% score applies only to the selected operators and in-scope mutations. It does
not establish complete requirements coverage, freedom from flakiness, production
schema fidelity, live IAM/credentials, or all framework annotation behavior.

Standard bytecode operators do not systematically vary SQL strings, constraints,
foreign-key relationships or date formats. Keep direct database assertions for
binding, distinct linked IDs, unrelated-row preservation and partial writes.
Mutation-tool coverage can also differ from JaCoCo because of built-in filters,
generated code and excluded targets. Report both scopes honestly.

## Sources

Consult current versions when implementing; these links are references, not a
requirement to install optional commercial products.

- PIT concepts and outcomes: https://pitest.org/quickstart/basic_concepts/
- PIT command-line options: https://pitest.org/quickstart/commandline/
- Gradle plugin: https://gradle-pitest-plugin.solidsoft.info/
- PIT JUnit adapter: https://github.com/pitest/pitest-junit5-plugin
- Spring limitations and optional licensed support: https://docs.arcmutate.com/docs/spring.html
