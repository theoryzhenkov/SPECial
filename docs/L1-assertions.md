---
scope: L1
summary: "Assertion format, test linking conventions, staleness integration"
modified: 2026-03-15
reviewed: 2026-03-15
depends:
  - path: docs/L0-assertions
  - path: docs/L0-file-structure
    section: "1.1.2. Depends"
    local: "4. Cross-spec assertions"
  - path: docs/L0-file-structure
    section: "1.1.4. Modified & Reviewed"
    local: "5. Staleness integration"
dependents:
  - docs/L3-agent-reference
---

# Assertion format

This document defines the assertion table format, test linking conventions, and how assertions integrate with SPECial's staleness mechanism.

## 1. Assertion tables

Assertions are declared in the spec body as Markdown tables under a heading that contains the word "Assertions". A spec file may have multiple assertion sections (e.g., one per domain concept).

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

## 3. Coverage reporting

Tooling can report assertion coverage by cross-referencing assertion tables in spec files with test-linking comments or file conventions. The key metrics:

- **Covered assertions**: assertions with at least one linked test.
- **Uncovered assertions**: assertions with no linked tests. MUST assertions without tests are higher priority.
- **Orphan tests**: tests with `spec:` comments pointing to assertion IDs that don't exist in any spec.

Coverage reporting is advisory. SPECial does not prescribe thresholds or CI gates — projects decide their own enforcement level.

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

To make this work, test directories or sentinel files within them participate in the dependency graph. Projects using sentinel files must include the test directory in [`paths`](L0-project-structure.md#1-configuration) in `special.conf.toml` so that SPECial tooling can discover them.

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
