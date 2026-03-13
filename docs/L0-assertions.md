---
scope: L0
summary: "Testable assertions in specs — bridging SDD and TDD"
modified: 2026-03-15
reviewed: 2026-03-15
depends:
  - path: index
  - path: docs/L0-file-structure
    section: "1.1.1. Scope"
    local: "2. Assertions across scope levels"
dependents:
  - path: docs/L1-assertions
  - path: docs/L0-tooling
  - path: docs/L3-agent-reference
---

# Assertions

SPECial tracks whether specs are consistent with each other (spec-to-spec staleness), but has no mechanism for linking specs to tests. Specs describe what must hold; tests verify what actually holds. In practice these two artifacts are maintained independently — specs drift from tests, tests verify implementation details that may or may not reflect the specification, and the gap widens silently.

The root cause is operational, not technical. Specifications are not enforced because enforcement depends on humans remembering to check. Tests are not traced to specs because nothing requires the link. Both SDD (specification-driven development) and TDD suffer from the same bottleneck: manual discipline.

Assertions address this by making specifications testable. A spec file can declare concrete, verifiable claims — assertions — that tests then reference by ID. Tooling reports which assertions have linked tests and which don't. When a spec changes, linked tests are flagged stale through the existing dependency mechanism. The result is a closed spec-to-test loop: spec defines assertion, test verifies assertion, change propagates through the graph. Code correctness remains the test's responsibility — assertions close the gap between what specs require and what tests actually check.

## 1. What assertions are

An assertion is a testable claim within a spec file. It has a unique ID, an RFC 2119 severity keyword, and a natural-language statement of what must (or should) hold. Assertions live in the spec body as structured [assertion tables](L1-assertions.md#1-assertion-tables).

Assertions are not tests. They don't contain executable code and SPECial does not run them. They are the spec-side anchor that tests link back to. The assertion says *what*; the test proves *whether*.

## 2. Assertions across scope levels

Assertions apply at all spec levels where testable claims exist (L1–L3). The scope level shapes what kind of assertions appear:

- **L1 (Contracts)**: Invariants, API contracts, business rules. These are the most valuable assertions — they state what must always hold regardless of implementation. A handful per spec file, each high-severity.
- **L2 (Structure & Flows)**: Interaction rules, sequence constraints, data flow properties. These assertions describe how components cooperate and map naturally to integration tests.
- **L3 (Implementation)**: Edge cases, performance bounds, error handling behavior. Narrow and specific, often numerous. These map to unit tests.

L0 specs are too abstract for testable assertions. If an L0 claim is precise enough to test, it belongs in an L1.

## 3. Why this matters for agents

An agent reading a spec can extract assertions and use them as acceptance criteria. The workflow becomes: read spec, write tests for assertions (TDD red), implement until tests pass (green), refactor. The agent doesn't need a human to tell it "what done looks like" — the assertions define it.

Coverage reporting closes the loop. After implementation, tooling can verify that every MUST assertion has at least one linked test. An agent can self-check before declaring a task complete.

## 4. What assertions are not

SPECial defines the spec-side format and the linking convention. It does not:

- Run tests or integrate with test runners
- Generate test code from assertions
- Enforce coverage thresholds (though projects can build this on top)

Language-specific tooling (pytest plugins, Jest integrations, etc.) is out of scope. SPECial provides the specification layer; execution is the project's concern.
