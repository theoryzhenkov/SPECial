# SPECial — Agent Reference

Reference for working with projects that use the [SPECial](https://the-o-space.github.io/special/) documentation standard. Install this file at `.claude/docs/SPECial.md` (or equivalent) so agents can work with SPECial-documented codebases.

## Orientation

A SPECial project is identified by `special.conf.toml` at its root:

```toml
root = "README"       # entry point file (no .md extension, default: "README")
paths = ["."]         # directories containing spec files (default: ["."])
```

Start by reading the `root` file. It lists top-level domains as `dependents`. Each domain has a `summary` in its frontmatter — read summaries to decide which files are relevant before loading full bodies.

## Frontmatter

Every SPECial file has YAML frontmatter:

```yaml
---
scope: L1
summary: "One-line description for navigation"
modified: 2026-01-05
reviewed: 2026-01-05
depends:
  - path: docs/L0-auth
    section: "Token Schema"    # optional: heading in dependency
    local: "Validation Rules"  # optional: heading in this file
dependents:
  - path: docs/L2-auth-flow
---
```

| Field        | Required | Description                                              |
| ------------ | -------- | -------------------------------------------------------- |
| `scope`      | yes      | Depth level: `root`, `L0`, `L1`, `L2`, `L3`             |
| `summary`    | yes      | One-line description, used for navigation                |
| `modified`   | yes      | Last content change (ISO 8601 date)                      |
| `reviewed`   | yes      | Last consistency verification (ISO 8601 date)            |
| `depends`    | no       | Files that can make this file stale when changed         |
| `dependents` | no       | Files that this file can make stale (navigation aid)     |
| `lifecycle`  | no       | `permanent` (default) or `ephemeral`                     |
| `type`       | no       | Document type: `PLAN`, `ISSUE`, `RFC`, `DEVIATION`, or custom |
| `status`     | no       | Lifecycle status of a typed doc (e.g. `draft`, `accepted`, `done`) |

All paths are relative to the project root, without `.md` extension.

## Scope Levels

| Level  | Answers                                              | When to read                                    |
| ------ | ---------------------------------------------------- | ----------------------------------------------- |
| `root` | What domains exist?                                  | First, to orient and pick a domain              |
| `L0`   | Why does this domain exist, what are the stakes?     | To decide if a domain is relevant to your task  |
| `L1`   | What are the interfaces, invariants, rules?          | To work *with* the domain without internals     |
| `L2`   | What components exist, how do they interact?         | To modify or extend the domain                  |
| `L3`   | Implementation patterns, edge cases, performance?    | To debug or optimize within the domain          |

Scope is encoded in the filename prefix: `L0-auth.md`, `L1-auth.md`, etc.

## File Naming

SPECial files encode metadata in the file path. Scope is always a prefix. Type and lifecycle can appear as uppercase filename prefixes or lowercase directory segments:

```
L0-security.md                      # standard permanent doc
PLAN-L1-auth-refactor.md            # type: PLAN, scope: L1
EPH-PLAN-L1-auth-refactor.md        # lifecycle: ephemeral, type: PLAN, scope: L1
eph/plan/L1-auth-refactor.md        # same, directory-style
```

The segment order (lifecycle, type, scope) is configurable via `naming_order` in `special.conf.toml`. Frontmatter is always the source of truth — the file path is a discoverability aid.

### Domain structure

Files may be grouped into optional domain/subdomain directories. The directory is the source of truth for a file's domain; the filename may mirror it for flat-list self-description (domain has no frontmatter field — it is derived from the path):

```
docs/auth/L0-auth.md                       # domain dir + filename
docs/auth/oidc/L2-auth-oidc-token-flow.md  # subdomain, mirrored in filename
docs/L0-auth.md                            # filename-only (no domain dir)
```

Domain is not a `naming_order` segment — it forms the trailing name portion, after the lifecycle/type/scope prefixes.

## Staleness

Two dates track drift between files:

- **`modified`**: bump when you change a file's content. Editing does *not* bump `reviewed`.
- **`reviewed`**: bump when you verify the file is consistent with everything it depends on. Reviewing does *not* bump `modified` unless content also changes.

**The rule**: if Y depends on X and `Y.reviewed < X.modified`, then Y is potentially stale. Verify Y against X. If Y is still correct, bump `Y.reviewed`. If Y needs edits, bump both `Y.modified` and `Y.reviewed` — the change then propagates outward through Y's dependents.

`depends` is the source of truth for staleness. `dependents` is a navigation aid (the inverse edges) with no semantic weight.

## Assertions

Spec files declare testable claims — assertions — that tests reference by ID. This bridges specifications and tests: the spec says *what* must hold, the test proves *whether* it holds. L1–L3 specs require an assertion table per section of testable behavior, with complete coverage of every MUST assertion; L0 motivation docs are exempt.

### Assertion tables

Assertions appear in the spec body as Markdown tables under a heading containing "Assertions":

```markdown
### Assertions

| ID             | Sev.   | Assertion                                                 |
| -------------- | ------ | --------------------------------------------------------- |
| token-fields   | MUST   | Valid tokens contain `user_id`, `expiry`, and `signature` |
| expired-reject | MUST   | Expired tokens are rejected with HTTP 401                 |
| sig-audit      | SHOULD | Invalid signature attempts are logged for audit           |
```

- **ID**: unique within the spec file. Kebab-case. Cross-file references use `path#id` (e.g., `docs/L1-auth#token-fields`).
- **Sev.**: RFC 2119 keyword (`MUST`, `SHOULD`, `MAY`). The column is the source of truth.

### Test linking

Tests link to assertions via a comment convention:

```python
# spec: docs/L1-auth#token-fields
def test_token_contains_required_fields():
    ...
```

Or via a file convention where test paths mirror spec paths:

```
tests/spec/<spec-name>/<assertion-id>.<ext>
```

Implementation code may use the same `# spec: path#id` comment to point at the assertion it implements; these links are advisory (no staleness weight) — the verifiable link is the test.

### Working with assertions

When implementing against a spec with assertions:

1. Read the assertion table as acceptance criteria
2. Write tests for assertions (TDD red)
3. Implement until tests pass (green)
4. Verify coverage: every `MUST` assertion must have at least one linked test (complete coverage)

## Realization

Realization — whether code does what a spec says — is **derived**, not asserted. A permanent spec with assertions is realized to the degree its assertions are covered by linked, passing tests. There is no `realized` field: a hand-maintained status would drift the moment code ships without a doc update. Specs without assertions (e.g. L0 context docs) have no derivable realization — if you need to track it, add assertions.

## Divergence

When the implementation *intentionally* differs from a spec and both are kept, record it in a deviation register — a `type: DEVIATION` file with a Deviations table (Ref, Spec says, Reality, Rationale, Disposition). The register `depends` on every spec it references, so divergences show up in those specs' `dependents` and participate in staleness. Do not edit a divergence back into the spec — keep intent and reality separate.

## When modifying SPECial files

1. Bump `modified` on any content change
2. Check `dependents` — downstream files may now be stale
3. If you change a file and a dependent's `reviewed < modified`, flag it

When modifying code covered by a SPECial spec, check if the change diverges from the specification. If the divergence is intentional and standing, record it in a deviation register. If it's a defect, fix the code. If the spec is wrong, update the spec.

## Full documentation

- [Files](https://the-o-space.github.io/special/files/L0-files/)
- [File contracts](https://the-o-space.github.io/special/files/L1-files/)
- [File structure](https://the-o-space.github.io/special/files/L2-files/)
- [Assertions](https://the-o-space.github.io/special/files/L1-files-assertions/)
- [Assertion format](https://the-o-space.github.io/special/files/L2-files-assertions/)
- [Documentation style](https://the-o-space.github.io/special/L0-documentation-style/)
- [Divergence](https://the-o-space.github.io/special/L0-divergence/)
- [Tooling](https://the-o-space.github.io/special/L0-tooling/)
