---
scope: L1
summary: "Assertions contract — the per-section requirement, complete coverage, realization"
modified: 2026-06-27
reviewed: 2026-06-27
depends:
  - path: index
  - path: docs/files/L0-files
  - path: docs/files/L1-files
    section: "1.1.1. Scope"
    local: "3. Assertions across scope levels"
dependents:
  - docs/files/L2-files-assertions
  - docs/L0-tooling
  - docs/L0-divergence
  - docs/L3-agent-reference
---

# Assertions

SPECial tracks whether specs are consistent with each other (spec-to-spec staleness). Assertions track whether specs are realized by code: a spec declares testable claims, tests link back to them, and coverage — hence realization — is derived from the links. Without assertions, specs and tests are maintained independently: specs drift from tests, tests verify implementation details that may or may not reflect the specification, and the gap widens silently.

The root cause is operational, not technical. Specifications are not enforced because enforcement depends on humans remembering to check. Tests are not traced to specs because nothing requires the link. Both specification-driven development (SDD) and test-driven development (TDD) suffer the same bottleneck: manual discipline.

Assertions close this by making specifications testable *and required*. A spec file declares concrete, verifiable claims that tests reference by ID. Tooling reports which assertions have linked tests and which don't. When a spec changes, linked tests are flagged stale through the existing dependency mechanism. The result is a closed spec-to-test loop: spec defines assertion, test verifies assertion, change propagates through the graph. Code correctness remains the test's responsibility — assertions close the gap between what specs require and what tests actually check.

## 1. What assertions are

An assertion is a testable claim within a spec file. It has a unique ID, an RFC 2119 severity keyword, and a natural-language statement of what must (or should) hold. Assertions live in the spec body as structured [assertion tables](L2-files-assertions.md#1-assertion-tables).

Assertions are not tests. They don't contain executable code and SPECial does not run them. They are the spec-side anchor that tests link back to. The assertion says *what*; the test proves *whether*.

## 2. The assertion requirement

SPECial requires assertions. An L1–L3 spec file declares its testable claims as assertions, and every MUST assertion is covered by at least one linked test — complete coverage. This is what makes SPECial's SDD true TDD: the spec states what must hold, tests prove it, and the gap between spec and tests is closed by construction rather than by discipline.

L0 motivation documents are exempt — they state *why* a domain exists, not *what* the code must do, so they have no testable truths to assert. If an L0 claim is precise enough to test, it belongs in an L1.

The recommended granularity is one assertion table per section that states testable behavior; a file with a single cohesive contract may use one table for the file. Assertion format and linking conventions are in [L2-files-assertions](L2-files-assertions.md).

### Assertions

| ID                | Sev.   | Assertion                                                                              |
| ----------------- | ------ | -------------------------------------------------------------------------------------- |
| assert-truths     | MUST   | A SPECial implementation asserts truths in its domain documents with complete coverage. |
| per-section       | MUST   | Every section in an L1–L3 spec that states testable behavior declares an assertion table. |
| l0-exempt         | MUST   | L0 documents are exempt from the assertion requirement.                                |
| track-realization | SHOULD | A SPECial implementation tracks specification realization through assertion coverage. |

## 3. Assertions across scope levels

Assertions apply at all spec levels where testable claims exist (L1–L3). The scope level shapes what kind of assertions appear:

- **L1 (Contracts)**: Invariants, API contracts, business rules. These are the most valuable assertions — they state what must always hold regardless of implementation. A handful per spec file, each high-severity.
- **L2 (Structure & Flows)**: Interaction rules, sequence constraints, data flow properties. These assertions describe how components cooperate and map naturally to integration tests.
- **L3 (Implementation)**: Edge cases, performance bounds, error handling behavior. Narrow and specific, often numerous. These map to unit tests.

L0 specs are too abstract for testable assertions. If an L0 claim is precise enough to test, it belongs in an L1.

## 4. Why this matters for agents

An agent reading a spec can extract assertions and use them as acceptance criteria. The workflow becomes: read spec, write tests for assertions (TDD red), implement until tests pass (green), refactor. The agent doesn't need a human to tell it "what done looks like" — the assertions define it.

Coverage reporting closes the loop. After implementation, tooling verifies that every MUST assertion has at least one linked test. An agent can self-check before declaring a task complete.

## 5. Realization

Realization — whether the implementation actually does what a specification says — is the spec-to-code counterpart of staleness. SPECial derives it rather than asserting it: a permanent specification with [assertions](#1-what-assertions-are) is realized to the degree its assertions are [covered by linked tests](L2-files-assertions.md#3-coverage-reporting).

- **Realized assertion**: has at least one linked test that passes.
- **Unrealized assertion**: has no linked test, or its linked tests fail.
- **Realization of a spec file**: the roll-up of its assertions — fully realized, partial, or unrealized.

This is a derived signal, not a field. There is no `realized` frontmatter property: a hand-maintained status would drift the moment code ships without a doc update, the same manual-discipline rot assertions exist to eliminate. The [coverage view](L2-files-assertions.md#3-coverage-reporting) is the realization view.

For specifications without assertions — typically L0 context documents — realization does not meaningfully apply. An L0 states *why* a domain exists, not *what* the code must do, so it is neither realized nor unrealized. If a permanent contract lacks assertions and you want to track its realization, the answer is to add assertions: declare the testable claims, link the tests, and realization follows.

Intentional divergences — where the code deliberately does *not* match the spec and both are kept — are a separate concern, recorded in [deviation registers](../L0-divergence.md), not as realization status.

## 6. What assertions are not

SPECial defines the spec-side format and the linking convention. It does not:

- Run tests or integrate with test runners
- Generate test code from assertions
- Prescribe thresholds beyond complete coverage of MUST assertions — stricter thresholds and CI gates are the project's concern

Language-specific tooling (pytest plugins, Jest integrations, etc.) is out of scope. SPECial provides the specification layer; execution is the project's concern.
