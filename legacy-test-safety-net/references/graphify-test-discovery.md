# Graphify-Assisted Test Discovery

If the `graphify` skill is available, invoke it and follow its guidance to
understand the codebase's shape before selecting test points.

## Explore Efficiently

Prefer querying an existing, current graph over rebuilding one. Use architecture
and relationship information to locate:

- Public entry points and the paths to observable outputs.
- Dependency boundaries and existing injection seams.
- Central components whose changes can affect several features.
- Pinch points where a few focused tests protect several paths.

Use graph findings to prioritize source inspection and identify candidate
coverage gaps efficiently, not as proof that behavior is tested.

## Choose Tests Using Evidence

Confirm relevant relationships against current source, particularly when the graph
may be stale. Compare existing tests with measured coverage before choosing new
UT, IT or AT cases. A graph cannot establish runtime behavior or coverage by itself.

Use the local skill's test-pyramid guidance to assign each behavior to its lowest
useful layer. Do not create acceptance tests merely because a graph path crosses
several components.

## Keep Exploration Proportional

If no graph exists, follow the Graphify skill's guidance when useful. Keep graph
generation proportional to the task; do not make it a prerequisite for a small,
well-understood change.

If `graphify` is unavailable, continue with code intelligence and focused source
inspection. Do not install it automatically or block testing work on its absence.
