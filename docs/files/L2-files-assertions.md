---
scope: L2
summary: "Assertion table format, test and code linking, coverage, staleness integration"
modified: 2026-06-27
reviewed: 2026-06-27
depends:
  - path: index
  - path: docs/files/L0-files
  - path: docs/files/L1-files-assertions
  - path: docs/files/L1-files
    section: "1.1.2. Depends"
    local: "4. Cross-spec assertions"
dependents:
  - docs/L0-tooling
  - docs/L0-divergence
  - docs/L3-agent-reference
---

# Assertion format

This document defines the assertion table format, the test- and code-linking conventions, coverage requirements, and how assertions integrate with SPECial's staleness mechanism. The assertion *requirement* is in [L1-files-assertions](L1-files-assertions.md).

## 1. Assertion tables

Assertions are declared in the spec body as Markdown tables under a heading that contains the word "Assertions". A spec file may have multiple assertion sections (e.g., one per section or per domain concept).

### 1.1. Table schema

| Column      | Required | Description                                                      |
| ----------- | -------- | ---------------------------------------------------------------- |
| `ID`        | yes      | Unique identifier within the spec file. Freeform string.         |
| `Sev.`      | yes      | RFC 2119 keyword: `MUST`, `SHOULD`, or `MAY`.                   |
| `Assertion` | yes      | Natural-language statement of what the requirement is.           |

Example:

```markdown
### Assertions

| ID              | Sev.   | Assertion                                                 |
| --------------- | ------ | --------------------------------------------------------- |
| token-fields    | MUST   | Valid tokens contain `user_id`, `expiry`, and `signature` |
| expired-reject  | MUST   | Expired tokens are rejected with HTTP 401                 |
| sig-audit       | SHOULD | Invalid signature attempts are logged for audit           |
```

### 1.2. Assertion IDs

IDs are freeform strings, unique within the spec file they belong to. Use short, descriptive, kebab-case identifiers: `token-fields`, `charge-idempotent`, `retry-backoff`.

When referencing an assertion from outside its spec file, qualify it with the spec path: `docs/L1-auth#token-fields`. Within the same file, the bare ID suffices.

### 1.3. Severity keywords

Severity follows [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119):

- **MUST**: The assertion is an absolute requirement. Violation is a bug.
- **SHOULD**: The assertion is recommended. Deviation requires justification.
- **MAY**: The assertion is optional. Useful for documenting permitted behaviors.

The Sev. column is the source of truth for severity. Assertion text is natural language and does not need to echo the RFC 2119 keyword.

## 2. Test linking

Tests reference assertions they verify. SPECial defines two linking conventions; projects choose whichever fits. Both can coexist.

### 2.1. Comment convention

A standardized comment in the test file links it to one or more assertions:

```
# spec: <spec-path>#<assertion-id>
```

The comment uses the line-comment syntax of the test's language. Place it immediately before the test function or at the top of a test file if the entire file verifies one assertion.

```python
# spec: docs/L1-auth#token-fields
def test_token_contains_required_fields():
    token = create_token(valid=True)
    assert "user_id" in token
    assert "expiry" in token
    assert "signature" in token
```

```typescript
// spec: docs/L1-auth#expired-reject
test("expired tokens return 401", async () => {
  const res = await client.get("/me", { token: expiredToken });
  expect(res.status).toBe(401);
});
```

Multiple assertions can be referenced with multiple comments:

```python
# spec: docs/L1-auth#token-fields
# spec: docs/L1-auth#expired-reject
def test_expired_token_missing_fields():
    ...
```

### 2.2. File convention

Test file paths mirror spec paths. Each assertion gets a test file or directory named after it:

```
tests/
  spec/
    L1-auth/
      token_fields.py
      expired_reject.py
    L2-auth-flow/
      login_sequence.py
```

The mapping rule: `tests/spec/<spec-name>/<assertion-id>.<ext>`. The `spec/` directory under the test root signals these are spec-linked tests as distinct from implementation tests. Any language-required prefix (e.g., `test_` for Python) is the language's concern, not SPECial's.

The file convention provides discoverability — you can see which assertions have tests by listing the directory. The comment convention provides flexibility — a single test can verify multiple assertions, and tests can live anywhere.

### 2.3. Code linking

Implementation code may reference the assertion it implements using the same `# spec:` comment convention as tests:

```python
# src/auth/token.py
# spec: docs/auth/L1-auth#token-fields
def issue_token(user):
    ...
```

Code→spec links are **advisory navigation**, not a staleness-tracked edge. They carry no semantic weight — like `dependents` — because code is not part of the staleness graph: there is no edge from a spec to a source file, so a code change cannot flag the spec, and a spec change cannot be verified against code by the `reviewed < modified` rule. This is the same limitation that applies to [realization](L1-files-assertions.md#5-realization).

The verifiable relationship remains the **test→spec** link: a test proves the assertion, and coverage is derived from that. A code comment only helps a reader *find* the governing spec; it does not establish that the code satisfies it. Treat code→spec links as discoverability aids, in keeping with derive-over-assert: the truth of "this code implements spec X" comes from the test, not from the comment.

To keep code comments from rotting, tooling validates that every referenced assertion ID exists — a dangling reference (assertion renamed or removed) is flagged, the way doc-comment checkers validate references across markdown and source. This dangling-ref check replaces graph staleness, which cannot reach code. Optionally, change-level traceability can use Conventional Commits footers, which are immutable and coarse-grained:

```
feat(auth): refresh token rotation

Specs: docs/auth/L1-auth#token-fields
```

## 3. Coverage reporting

Coverage is **required**, not advisory. Every MUST assertion MUST have at least one linked test — complete coverage. SHOULD and MAY assertions SHOULD and MAY be covered respectively. SPECial prescribes this minimum; projects may enforce stricter thresholds or CI gates on top.

Tooling reports assertion coverage by cross-referencing assertion tables in spec files with test-linking comments or file conventions. The key metrics:

- **Covered assertions**: assertions with at least one linked test.
- **Uncovered assertions**: assertions with no linked tests. MUST assertions without tests are violations.
- **Orphan tests**: tests with `spec:` comments pointing to assertion IDs that don't exist in any spec.

Coverage is the [realization](L1-files-assertions.md#5-realization) signal: a spec is realized to the degree its assertions are covered by passing tests, so the coverage view doubles as the realization view.

## 4. Cross-spec assertions

When an invariant spans multiple specs, the assertion lives in whichever spec owns the concept. Other specs reference it through the existing `depends` mechanism with `section`/`local` scoping.

```yaml
# docs/L1-payments.md
---
depends:
  - path: docs/L1-auth
    section: "Assertions"
    local: "Token Validation"
---
```

The payments spec does not redeclare the assertion. It depends on the auth spec's assertion section and references it by qualified ID (`docs/L1-auth#token-fields`) in its own prose. Tests for the cross-cutting behavior link to the owning spec's assertion ID.

This avoids duplication and ensures changes to the assertion propagate staleness to all dependent specs through the existing graph.

## 5. Staleness integration

Assertions extend SPECial's existing staleness mechanism rather than introducing a new one.

When a spec file containing assertions is modified, its `modified` date bumps. Any files listed in its `dependents` — including test directories or test metadata files — are flagged stale if their `reviewed` date predates the spec's `modified` date.

To make this work, test directories or sentinel files within them participate in the dependency graph. Projects using sentinel files must include the test directory in [`paths`](L2-files.md#3-configuration) in `special.conf.toml` so that SPECial tooling can discover them.

```yaml
# docs/L1-auth.md
---
dependents:
  - path: tests/spec/L1-auth
---
```

```yaml
# tests/spec/L1-auth/SPEC.md  (sentinel file)
---
scope: L3
summary: "Spec tests for L1-auth assertions"
reviewed: 2026-03-13
depends:
  - path: docs/L1-auth
    section: "Assertions"
---
```

When `docs/L1-auth.md` is modified, the sentinel file's `reviewed < modified` flags the test suite for review. After updating tests to match the changed assertions, bump the sentinel's `reviewed` date.

Projects that don't want sentinel files can track staleness through the comment convention alone — tooling parses `spec:` comments and compares test file modification dates against spec modification dates. The sentinel approach is more explicit; the comment approach is lighter weight.

## 6. Assertions

| ID                    | Sev. | Assertion                                                                          |
| --------------------- | ---- | ---------------------------------------------------------------------------------- |
| assertion-table-format | MUST | Assertions appear as Markdown tables under a heading containing "Assertions".     |
| unique-ids            | MUST | Assertion IDs are unique within their spec file.                                  |
| severity-column       | MUST | The Sev. column is an RFC 2119 keyword (`MUST`, `SHOULD`, or `MAY`).            |
| must-coverage         | MUST | Every MUST assertion has at least one linked test.                                 |
| code-link-advisory    | MAY  | Implementation code may reference assertions via the `spec:` comment; such links carry no staleness weight. |
