---
scope: L1
summary: "Frontmatter schema and staleness contracts"
modified: 2026-06-27
reviewed: 2026-06-27
depends:
  - path: index
  - path: docs/files/L0-files
dependents:
  - docs/files/L1-files-assertions
  - docs/files/L2-files
  - docs/files/L2-files-assertions
  - docs/L0-tooling
  - docs/L0-divergence
  - docs/L3-agent-reference
---

# File contracts

This document defines the frontmatter schema and the staleness invariants that govern SPECial files. Motivation is in [L0-files](L0-files.md); file naming, domain structure, and configuration are in [L2-files](L2-files.md).

## 1. Frontmatter

Frontmatter is a YAML, TOML or JSON block at the beginning of the `.md` file. Frontmatter is used by SPECial to store structured metadata.

```yaml
---
scope: L1
summary: "OAuth2/OIDC flows, token lifecycle, session management"
modified: 2026-01-05
reviewed: 2026-01-03
depends:
  - path: auth/L2-user-model
    section: "2.3.4 Token schema"
    local: "1.2 Validation rules"
dependents:
  - path: auth/L2-auth-datastructures
---
```

### 1.1. Schema

| Field        | Type       | Required | Description                                                            |
| ------------ | ---------- | -------- | ---------------------------------------------------------------------- |
| `scope`      | `enum`     | yes      | Depth of detail: `root`, L0–L3. Also expressed in file name.           |
| `summary`    | `string`   | yes      | One-line description, used for [navigation](L2-files.md#2-navigation).  |
| `modified`   | `date`     | yes      | Last content change, ISO 8601.                                         |
| `reviewed`   | `date`     | yes      | Last verified consistency timestamp, ISO 8601.                         |
| `lifecycle`  | `enum`     | no       | `permanent` (default) or `ephemeral`. See [Lifecycle](#115-lifecycle). |
| `type`       | `string`   | no       | Document type. Also expressed in file name. See [Type](#116-type).     |
| `status`     | `string`   | no       | Lifecycle status of a typed doc. See [Status](#117-status).           |
| `depends`    | `object[]` | no       | This file can become stale after changes to these files.               |
| `dependents` | `object[]` | no       | These files can become stale after changes to this file.               |

A file's domain is derived from its path, not a frontmatter field. See [Domain and subdomain](L2-files.md#13-domain-and-subdomain).

#### 1.1.1. Scope

| Level                    | Answers                                                                                     | Access                                                                             |
| ------------------------ | ------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| root: Index              | What domains exist in this project?                                                         | Entry point. An agent reads this first to orient and pick a domain.                |
| L0: Context & Motivation | What is this domain, why does it exist, what are the stakes?                                | An agent reads this to decide whether this domain is relevant to its task.         |
| L1: Contracts            | What are the interfaces, invariants, constraints, rules?                                    | An agent reads this to work _with_ the domain without understanding its internals. |
| L2: Structure & Flows    | What are the components, how do they interact, what are the key sequences?                  | An agent reads this to modify or extend the domain.                                |
| L3: Implementation       | What are the implementation patterns, edge cases, known issues, performance considerations? | An agent reads this to debug or optimize within the domain.                        |

#### 1.1.2. Depends

If changes to file X can affect file Y, then Y depends on X. For example, if changes in `L0-accessibility` can affect `L1-user-interface`, then `L1-user-interface` sets `depends: [L0-accessibility]`.

`depends` is the **source of truth** for staleness detection. Each entry has the following schema:

| Field     | Type     | Required | Description                                                                                                  |
| --------- | -------- | -------- | ------------------------------------------------------------------------------------------------------------ |
| `path`    | `string` | yes      | Path to the dependency file.                                                                                 |
| `section` | `string` | no       | Heading in the dependency file that this file depends on. If omitted, the dependency is file-level.          |
| `local`   | `string` | no       | Heading in this file that is dependent on the `section`. If omitted, the whole file is considered dependent. |

When `section` and `local` are specified, staleness is scoped: only changes under that heading in the dependency are relevant, and only the local heading needs review.

Entries can use a shorthand when only `path` is needed:

```yaml
depends:
  - L0-accessibility # shorthand, equivalent to { path: L0-accessibility }
  - path: auth/L2-user-model # full form, needed when section/local are specified
    section: "2.3.4 Token schema"
    local: "1.2 Validation rules"
```

#### 1.1.3. Dependents

Inverse of `depends`. Lists files that depend on this file, used for navigating the dependency graph in the forward direction — after changing a file, its `dependents` tell you which files to check for staleness.

`dependents` carries **no semantic weight** for staleness — that is always computed from `depends`. It exists purely as a navigation aid. Every `depends` edge should have a matching `dependents` edge and vice versa.

`dependents` entries follow the same schema and shorthand rules as `depends`.

#### 1.1.4. Modified & Reviewed

These two fields track documentation staleness and drift.

**`modified`** is bumped when any content change is made to the document. Editing a file does _not_ automatically bump `reviewed` — a modification is not a consistency check.

**`reviewed`** is bumped when you verify that the file is faithful to all files it depends on. Reviewing does _not_ bump `modified` unless the content also changes.

The staleness rule: if Y depends on X, and `Y.reviewed < X.modified`, then Y is **potentially stale**. This requires verification. If Y is still consistent, bump `Y.reviewed`. If edits are required, bump both `Y.modified` and `Y.reviewed` — the modified change then propagates outward through Y's own dependents, and the process repeats until the entire dependency graph is consistent.

#### 1.1.5. Lifecycle

`lifecycle` distinguishes permanent documentation from temporary specifications that exist for a bounded period — feature plans, implementation designs, branch-scoped work.

| Value       | Meaning                                                                                                             |
| ----------- | ------------------------------------------------------------------------------------------------------------------- |
| `permanent` | Default. Omitting the field is equivalent to `permanent`. Part of the long-lived documentation.                     |
| `ephemeral` | Temporary specification. Expected to be deleted when the work it supports is complete (e.g. after a branch merges). |

Some [document types](#116-type) default to `ephemeral`. An explicit `lifecycle` value in frontmatter always takes precedence over the type-implied default.

Lifecycle can also be encoded in [file naming](L2-files.md#1-file-naming) via the `EPH` prefix or `eph/` directory segment. Frontmatter is the source of truth.

Ephemeral files follow the same frontmatter schema as permanent files. The differences are in how they participate in the dependency graph:

- **Ephemeral → permanent `depends`: one-way.** An ephemeral file may `depends` on permanent files. The permanent files do **not** add matching `dependents` entries. Since `dependents` carries [no semantic weight](#113-dependents), this does not affect staleness detection.
- **Permanent → ephemeral `depends`: forbidden.** A permanent file must never `depends` on an ephemeral file. This ensures deleting ephemeral files never creates dangling references in permanent documentation.
- **Ephemeral → ephemeral: normal rules.** Ephemeral files that form their own subgraph (e.g. a plan with a detailed design underneath) use standard bidirectional `depends`/`dependents` edges. They share a lifecycle and are deleted together.

Staleness tracking works normally for ephemeral files: if a permanent dependency is modified, the ephemeral file is flagged stale. The difference is that staleness never propagates _from_ an ephemeral file into permanent files, because no permanent file depends on it.

#### 1.1.6. Type

`type` classifies the purpose of a document. Standard documentation files omit the field.

The type set is open — any uppercase string is valid. The following types are recommended:

| Type     | Purpose                                          | Default lifecycle |
| -------- | ------------------------------------------------ | ----------------- |
| _(none)_ | Standard documentation                           | `permanent`       |
| `PLAN`   | Implementation plans, feature designs            | `ephemeral`       |
| `ISSUE`  | Tracked problems, bugs, improvement requests     | `ephemeral`       |
| `RFC`    | Requests for comment, proposals under discussion | `ephemeral`       |
| `DEVIATION` | Recorded divergences between spec and implementation | `permanent`   |

When a type has a default lifecycle, that default applies unless overridden by an explicit `lifecycle` field. Custom types default to `permanent`.

`DEVIATION` is the one recommended type that defaults to `permanent` — a deviation is a durable fact about the system, not temporary work. Deviation registers are covered in [Divergence](../L0-divergence.md).

Type can also be encoded in [file naming](L2-files.md#12-type-and-lifecycle-encoding) via a filename prefix or directory segment. Frontmatter is the source of truth.

#### 1.1.7. Status

`status` records where a typed document sits in its own lifecycle — distinct from [`lifecycle`](#115-lifecycle) (how long the document lives) and from [realization](L1-files-assertions.md#5-realization) (whether code matches the spec). It applies to typed documents whose purpose implies a life cycle: plans, issues, RFCs. Permanent, un-typed specifications carry no `status` — their state is [realization](L1-files-assertions.md#5-realization), derived from assertions.

The value set is open; recommended values are conventional per type:

| Type        | Recommended status values                                      |
| ----------- | -------------------------------------------------------------- |
| `PLAN`      | `draft` → `accepted` → `in-progress` → `done` \| `abandoned`  |
| `RFC`       | `draft` → `accepted` → `superseded`                           |
| `ISSUE`     | `open` → `resolved` \| `wont-fix`                             |
| `DEVIATION` | _(no `status`; the register's `disposition` column covers it)_ |

A status change is a content change: bump `modified`, and staleness propagates to dependents normally. When an ephemeral document reaches a terminal status (`done`, `resolved`, `superseded`, `abandoned`), it is a candidate for deletion under its [ephemeral lifecycle](#115-lifecycle).

## 2. Assertions

| ID                          | Sev.   | Assertion                                                                                    |
| --------------------------- | ------ | -------------------------------------------------------------------------------------------- |
| required-fields             | MUST   | Every SPECial file declares `scope`, `summary`, `modified`, and `reviewed`.                 |
| staleness-rule              | MUST   | If Y depends on X and `Y.reviewed < X.modified`, Y is potentially stale.                     |
| depends-source-of-truth     | MUST   | `depends` is the source of truth for staleness; `dependents` is a navigation aid only.      |
| edge-symmetry               | SHOULD | Every `depends` edge has a matching `dependents` edge and vice versa.                       |
| no-permanent-on-ephemeral   | MUST   | A permanent file never depends on an ephemeral file.                                         |
| type-default-lifecycle      | MAY    | A type's default lifecycle applies unless overridden by an explicit `lifecycle` field.       |
