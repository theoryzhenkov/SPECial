---
scope: L0
summary: "Recording divergences between spec and implementation"
modified: 2026-06-27
reviewed: 2026-06-27
depends:
  - path: index
  - path: docs/files/L1-files
    section: "1.1.6. Type"
  - path: docs/files/L1-files-assertions
    section: "5. Realization"
  - path: docs/files/L2-files-assertions
    section: "1.2. Assertion IDs"
dependents:
  - docs/L3-agent-reference
---

# Divergence

SPECial tracks whether specs are consistent with each other (spec-to-spec staleness) and whether specs are realized by tests (assertion coverage). What it does not capture is the case where the *implementation intentionally differs* from the specification — a deliberate choice, not a missing feature or a bug. Without a home for these, divergences get buried in commit messages and code comments, invisible to anyone reading the spec.

Deviation registers address this. A spec project can keep one or more `DEVIATION`-typed files that record every intentional divergence: what the spec says, what the code does, why, and the disposition. The register links back to the spec claims it diverges from through the existing `depends` mechanism, so a deviation shows up in a spec's `dependents` and participates in staleness — reusing the engine rather than building a parallel one.

This is the as-built principle: keep the specification (intent) and the deviation register (reality) as separate artifacts. Do not edit the divergence back into the spec — a spec that silently records its own exceptions is no longer a statement of intent.

## 1. What deviations are

A deviation is a recorded, intentional divergence between a specification claim and the implementation. It is **not**:

- A bug — a spec violation is a bug, fixed in code, not recorded as a deviation.
- A missing feature — uncovered by an assertion, tracked as [realization](files/L1-files-assertions.md#5-realization), not a deviation.
- A spec change — if the code is right and the spec is wrong, update the spec and bump `modified`; do not file a deviation.

A deviation is the case where the code is *deliberately* not what the spec says, and both are kept: the spec states the intended design, the register records the standing exception and its rationale.

## 2. The DEVIATION type

A deviation register is a SPECial file with `type: DEVIATION`. Unlike other recommended types, `DEVIATION` defaults to the `permanent` [lifecycle](files/L1-files.md#115-lifecycle) — a deviation is a durable fact about the system, not temporary work. A register is therefore a living document: it grows as new divergences are accepted and shrinks as old ones are reconciled back into the spec.

A `DEVIATION` file carries no [`status`](files/L1-files.md#117-status) field. The per-entry `disposition` column (below) serves the same purpose at the row level.

## 3. Deviation tables

Deviations are declared in the register body as Markdown tables under a heading containing the word "Deviations". A register may have multiple deviation sections (e.g., one per domain).

### 3.1. Table schema

| Column        | Required | Description                                                                  |
| ------------- | -------- | ---------------------------------------------------------------------------- |
| `Ref`         | yes      | The spec claim being diverged from: `path#assertion-id` or `path#section`.   |
| `Spec says`   | yes      | What the specification requires (quoted or paraphrased).                     |
| `Reality`     | yes      | What the implementation actually does.                                      |
| `Rationale`   | yes      | Why the divergence is intentional and acceptable.                            |
| `Disposition` | no       | `accepted` (default) \| `superseded` \| `reconciled-into-spec`.             |

Example:

```markdown
### Deviations

| Ref                              | Spec says                | Reality                | Rationale                         | Disposition |
| -------------------------------- | ------------------------ | ---------------------- | --------------------------------- | ----------- |
| docs/auth/L1-auth#token-fields   | Tokens carry `signature` | Tokens carry `mac`     | Legacy clients validate MAC only  | accepted    |
| docs/auth/L1-auth#expiry-window  | 15-minute expiry         | 60-minute expiry       | Mobile client refresh constraints | accepted    |
```

### 3.2. Referencing spec claims

`Ref` points at the precise spec claim being diverged from. When the claim is an [assertion](files/L2-files-assertions.md#12-assertion-ids), use its qualified ID: `docs/auth/L1-auth#token-fields`. When it is a prose section, use the path and heading: `docs/auth/L2-auth-flow#token-issuance`.

## 4. Staleness integration

A deviation register declares every spec it references as a `depends` entry. This is what makes divergences visible from the spec: each referenced spec lists the register among its `dependents`, and the staleness rule applies normally — if a spec is modified after the register was reviewed, the register is flagged for re-verification.

```yaml
# docs/deviations.md
---
type: DEVIATION
scope: L0
summary: "Recorded divergences between spec and implementation"
modified: 2026-06-27
reviewed: 2026-06-27
depends:
  - path: docs/auth/L1-auth
    section: "Assertions"
---
```

Reviewing a flagged register means checking each deviation against the changed spec:

- If the spec changed such that the divergence no longer exists, set the row's `Disposition` to `reconciled-into-spec` (and, ideally, remove the row once reconciled).
- If the divergence still holds, bump the register's `reviewed` date.

A change to code that makes a recorded divergence inaccurate (e.g., the implementation catches up to the spec) is **not** automatically detected — code is not part of the staleness graph, the same limitation that applies to [realization](files/L1-files-assertions.md#5-realization). Reconciliation there is manual, recorded via the `Disposition` column.

## 5. Organization

A project may keep a single global register, one register per domain, or one file per deviation. A single register with a table is the lightest form and mirrors the [assertion table](files/L1-files-assertions.md) convention; per-domain registers give finer staleness granularity. Choose one and stay consistent.
