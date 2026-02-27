---
scope: L0
summary: "SPECial file standard"
modified: 2026-02-27
reviewed: 2026-02-27
depends:
  - path: index
    local: "Pages"
  - path: docs/L0-project-structure
    section: "Path Resolution"
    local: "File Naming"
dependents:
  - docs/L0-tooling
---

# SPECial file structure

Markdown SPECial files include a **body** and YAML, TOML or JSON **frontmatter**. While documentation for a project can consist of files other than Markdown, SPECial relies on some sort of per-file metadata storage to provide assistance with specification drift and navigation.

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
| `summary`    | `string`   | yes      | One-line description, used for [navigation](#3-navigation).            |
| `modified`   | `date`     | yes      | Last content change, ISO 8601.                                         |
| `reviewed`   | `date`     | yes      | Last verified consistency timestamp, ISO 8601.                         |
| `lifecycle`  | `enum`     | no       | `permanent` (default) or `ephemeral`. See [Lifecycle](#115-lifecycle). |
| `type`       | `string`   | no       | Document type. Also expressed in file name. See [Type](#116-type).     |
| `depends`    | `object[]` | no       | This file can become stale after changes to these files.               |
| `dependents` | `object[]` | no       | These files can become stale after changes to this file.               |

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

Lifecycle can also be encoded in [file naming](#2-file-naming) via the `EPH` prefix or `eph/` directory segment. Frontmatter is the source of truth.

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

When a type has a default lifecycle, that default applies unless overridden by an explicit `lifecycle` field. Custom types default to `permanent`.

Type can also be encoded in [file naming](#2-file-naming) via a filename prefix or directory segment. Frontmatter is the source of truth.

## 2. File Naming

SPECial files encode metadata in two places: **frontmatter** (source of truth) and the **file path** (discoverability aid). The file path can encode scope, [type](#116-type), and [lifecycle](#115-lifecycle) using filename prefixes or directory segments interchangeably.

### 2.1. Scope prefix

Every SPECial file encodes its scope level as a filename prefix:

```
L0-security.md
L1-authentication.md
L2-auth-flow.md
L3-token-validation.md
```

The prefix makes scope immediately visible in file listings and allows tools to infer scope without parsing frontmatter.

### 2.2. Type and lifecycle encoding

Type and lifecycle can each be expressed as an **uppercase filename prefix** or a **lowercase directory segment**. These two forms are interchangeable — a project may use either or both, as long as the encoding is consistent with frontmatter.

| Encoding              | Filename prefix | Directory segment |
| --------------------- | --------------- | ----------------- |
| Type (e.g. `PLAN`)    | `PLAN-`         | `plan/`           |
| Lifecycle `ephemeral` | `EPH-`          | `eph/`            |

The ordering of lifecycle, type, and scope segments in the file path is a project-level convention. SPECial does not enforce a particular order — choose whichever grouping makes navigating your documentation easiest and stay consistent within the project. The [SPECial CLI](L0-tooling.md) reads the configured order from [`naming_order`](L0-project-structure.md#1-configuration) in `special.conf.toml`.

All of the following encode the same file (`lifecycle: ephemeral`, `type: PLAN`, `scope: L1`):

```
# lifecycle → type → scope (default)
docs/security/EPH-PLAN-L1-auth-refactor.md
docs/security/eph/plan/L1-auth-refactor.md

# type → lifecycle → scope
docs/security/PLAN-EPH-L1-auth-refactor.md
docs/security/plan/eph/L1-auth-refactor.md

# type → scope (lifecycle implied by type)
docs/security/PLAN-L1-auth-refactor.md
docs/security/plan/L1-auth-refactor.md
```

Standard permanent documentation uses neither type nor lifecycle encoding — just the scope prefix:

```
docs/security/L0-security.md
```

When type already implies ephemeral lifecycle (e.g. `PLAN`), the `EPH` encoding is redundant and can be omitted:

```
docs/security/plan/L1-auth-refactor.md      # type implies ephemeral
docs/security/PLAN-L1-auth-refactor.md       # same, prefix style
```

### 2.3. Glob patterns

The two encoding styles give different glob ergonomics. Choose whichever fits the project:

```
# prefix style
L1-*.md              →  all standard L1 contracts
PLAN-*.md            →  all plans (any scope)
PLAN-L1-*.md         →  all L1 plans
EPH-*.md             →  all ephemeral files (prefix-encoded)
*L1-*.md             →  all L1 files including typed ones

# directory style
plan/L1-*.md         →  all L1 plans
plan/**/*.md         →  all plans (any scope)
eph/**/*.md          →  all ephemeral files (directory-encoded)
```

## 3. Navigation

SPECial uses `summary` and the dependency graph for incremental navigation — both for human readers and agents managing context budgets.

### 3.1. Summary

Each file's `summary` is the authoritative one-line description of its contents. An agent encountering a file path in a `dependents` list can read just the frontmatter of the referenced file to get its summary and scope, then decide whether to load the full body.

By convention, SPECial files may list their dependants' summaries in the file body. This avoids fetching all dependants' frontmatter to learn their approximate contents. As with other content, if a dependant's summary is updated, `modified` is bumped, and changes propagate via the [staleness mechanism](#114-modified-reviewed).

### 3.2. Root Index File

A **root index file** (configured as `root` in `special.conf.toml`, default: `README`) serves as the entry point, effectively an `L-1` scope. It may or may not list `L0` files as dependents — [file discovery](#2-file-naming) does not depend on it.

```yaml
# README.md
---
scope: root
summary: "Project documentation index"
modified: 2026-01-05
reviewed: 2026-01-05
---
# My Project
```
