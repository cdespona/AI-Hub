# Build Verification, Stability and Handoff

## Wire Commands, Coverage and CI

Expose independently runnable UT, IT and AT commands using native conventions.
Connect relevant layers to the normal verification lifecycle and produce useful
per-layer failure reports.

Generate combined coverage from the actual execution-data files for each suite.
Inspect the report: a successful coverage task alone does not prove the expected
classes or layers contributed.

Preserve existing coverage gates. For a new gate, use measured evidence and team
policy rather than an arbitrary percentage. Do not exclude handwritten behavior
or weaken thresholds just to obtain a green build.

Update directly related CI configuration when appropriate. State any new agent
requirements, such as a container engine, Compose plugin or registry access.
Do not claim CI execution was verified when only local commands ran.

## Verify Outcomes

Run targeted tests first, then all affected layers and existing build checks.
Check:

- The requested behavior is protected, not just a proxy.
- Refactoring retained prior assertions and current production behavior.
- Tests have focused names, clear AAA and independently arranged state.
- UT/IT/AT distribution reflects the intended pyramid.
- Coverage is measured and gates still pass.
- Infrastructure is cleaned up and temporary artifacts are removed.

Assess stability separately: repeat unmodified tests with cached results disabled,
independently reset fixtures and no automatic retries that hide failures. Report
intermittent failures explicitly; repeated green runs are evidence, not a guarantee.

## Document and Hand Off

Update the repository's existing documentation with commands, layer boundaries,
report locations, prerequisites, preserved quirks and honest limitations.
Keep the final handoff concise: meaningful changes and remaining caveats; include
test counts, commands or coverage figures when requested.
