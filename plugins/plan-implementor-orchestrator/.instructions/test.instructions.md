---
description: 'Guidelines for writing tests in Java and Kotlin projects using JUnit 5 & 6, Kotest, Mockito, and MockK.'
applyTo: '**/src/test/**/*.java,**/src/test/**/*.kt'
---

# 🧪 Test Instructions for GitHub Copilot

---

## 🏷️ Test Naming Guidelines

- Test names must be **human-readable** and **self-explanatory**, not mechanically derived from method or class names.
- Avoid naming tests like `happyPath_callMethodName()` or `testDoSomething()`.
- Naming conventions:
    - **Java:** `should_<expected_behavior>_when_<condition>()`
    - **Kotlin:** use backtick-enclosed descriptive names
- Focus on **behavior**, **outcome**, and **context** rather than the method under test.
- Names should describe the intent of the test so that a reader understands **what is being verified and under what conditions** without looking at the code.
- Apply consistently to **unit, integration, and acceptance tests**.

---

## 🏗️ Test Structure (AAA)

- Tests must follow **AAA (Arrange → Act → Assert)** strictly:
    1. **Arrange:** prepare all test data and mocks.
    2. **Act:** invoke the method or behavior under test.
    3. **Assert:** verify outcomes.
- **Do not interleave** additional setup, act, or asserts in the middle of a test.
- Prefer **one assert per logical behavior**:
    - If multiple unrelated checks must happen, use **JUnit 5 `assertAll`** or Kotest equivalent.
    - Do **not** use `assertAll` to assert multiple properties of the same object — these should be a single assert or custom matcher.
- Apply consistently across **unit, integration, and acceptance tests**.

---

## 🧭 When to Use Each Type of Test

### 🧩 Unit Tests
**Goal:** Validate the smallest possible piece of logic — fast and isolated.

Use unit tests when:
- Testing **pure domain logic** (value objects, aggregates, domain services).
- Verifying **business invariants** or edge cases.
- Checking **application services** that coordinate domain operations using mocked ports.
- No I/O or external dependencies are involved.

Avoid when:
- Behavior depends on databases, messaging systems, or external APIs.

> 💡 If it can run with plain objects and pure functions, it's a unit test.

---

### 🌐 Integration Tests
**Goal:** Validate collaboration between real components (ports, adapters, and infrastructure).

Use integration tests when:
- Testing **repositories**, **persistence**, or **outbound ports**.
- Verifying **serialization**, **schema compatibility**, or **transaction boundaries**.
- Testing **REST controllers** or **Kafka consumers** end-to-end within the same bounded context.

Avoid when:
- The logic is pure or already covered by a higher-level test.

> 💡 If the test touches I/O or multiple layers working together, it's integration.

---

### ✅ Acceptance Tests
**Goal:** Validate the system's behavior from the user's or external system's perspective.

Use acceptance tests when:
- Testing **complete use cases** end-to-end (input → output).
- Validating **domain language**, **business rules**, and **external contracts**.
- Ensuring the system works as expected across all adapters and infrastructure.

Avoid when:
- Testing small units or internal details.

> 💡 If the test checks business behavior from the outside, it's acceptance.

---

## 🌐 Integration & Acceptance Testing

- Integration and acceptance tests should validate the system through real ports and adapters.
- Ensure test resources (e.g., **WireMock**, **Testcontainers**) use **dynamic ports** or **isolated names** to prevent collisions across tests.
- Prefer **test resource lifecycle managers** that support:
    - **Shared-start semantics** (for efficiency when multiple tests share a container).
    - Or **per-test instances** (for strict isolation when required).
- When using **Docker Compose** or **Testcontainers**:
    - Isolate **network names** and **container names** to avoid cross-test interference.
- If a test stops a shared container prematurely, prefer:
    - A **shared lifecycle manager**, or
    - **Reference-counted start/stop logic** to ensure clean reuse without side effects.

