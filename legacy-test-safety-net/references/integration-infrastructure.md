# Realistic and Disposable Integration Infrastructure

Test engine-specific behavior against the actual engine where feasible. An
in-memory substitute is not evidence for vendor-specific SQL, sequences, date
conversion, constraints, transactions or batch behavior.

Respect the user's lifecycle preference. If Compose is requested, have the build
start it before relevant tests; do not substitute another container test framework.

Infrastructure should:

- Use a test-specific project/namespace and disposable credentials.
- Bind locally where appropriate and avoid shared or production databases.
- Wait for readiness, not an arbitrary sleep.
- Allow port and project-name overrides for concurrent builds.
- Run state-sharing suites sequentially and reset state between tests.
- Avoid stale cached results for a freshly recreated database.
- Tear down only owned resources on success, test failure and startup failure.
- Provide a documented cleanup command for interrupted builds.

Verify readiness, successful execution and cleanup. Exercise an intentional
failure path to confirm cleanup rather than assuming the task graph is correct.
Never stop unrelated services or delete broad directories.

Use authoritative schema migrations when available. If they are absent, label
test DDL as an inferred fixture. State which production constraints, triggers,
grants and cascade semantics remain unverified. Do not invent them to make tests
pass or claim production fidelity from a minimal fixture.
