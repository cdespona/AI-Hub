# Discover and Characterize Existing Behavior

Read repository guidance, source, manifests, test configuration, CI and relevant
documentation. Inspect the worktree and preserve existing user changes.

Identify:

- Public entry points and observable outputs.
- Domain rules, mapping, orchestration and error handling.
- External boundaries: databases, HTTP services, secrets, queues and filesystem.
- Existing injection seams and testing conventions.
- Authoritative schemas, migrations and contracts.
- Supported runtimes, dependency repositories and test commands.

Run the smallest useful baseline. Distinguish no tests from passing tests.
Reuse native tooling and existing dependencies before introducing new ones.

Ask before material choices that cannot be inferred safely, such as selecting a
database substitute, adding containers, or changing CI infrastructure requirements.
Do not assume a specific framework, database, build system or container manager.

## Build a Behavior Map

For each relevant behavior, choose its lowest useful test point and assign a
layer. Include successful paths, meaningful failures, empty/null inputs, ordering,
mapping and externally visible quirks where relevant.

Characterization tests record what the application actually does, not what its
author probably intended. Confirm surprising observations by executing the code.
Do not silently fix them while adding tests. Document important quirks and leave
intentional behavior changes for a separate verified step.

Find pinch points where a small number of assertions protect a broad path.
Mock only dependencies that prevent isolation or sensing. Preserve real logic
inside the boundary under test.
