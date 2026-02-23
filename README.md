# 1. SPECial

SPECial is a lightweight specification / documentation standard. It is designed to provide a solid foundation for spec-driven software development, but is adaptable to other domains. SPECial is designed primarily for markdown — this document format was chosen for its wide adoption and simplicity. 

> [!note] 
> Currently, SPECial doesn't provide any tooling: linters, formatters, drift detection, etc. If the standard's design receives wider adoption, I plan to develop a SPECial CLI / SDK. Often, the documentation is written as if tooling existed — this is in order to a) specify future tooling, and design with it in mind b) to allow agents to deterministically follow the specification.

---

## 2. Project structure

A SPECial project is a collection of files downstream from a **project root**.

### 2.1 Configuration

The project root is identified by the presence of `special.conf.toml`. If no `special.conf.toml` is present in the working directory, SPECial treats working directory as project root with default values.

```toml
# special.conf.toml
root = "README"         # entry point file (default: "README")
paths = ["."]           # directories containing SPECial files (default: ["."])
```

|Field|Type|Default|Description|
|---|---|---|---|
|`root`|`string`|`"README"`|Entry point file, without `.md` extension.|
|`paths`|`string[]`|`["."]`|Directories to scan for SPECial files, relative to project root.|

### 2.3 Path resolution

All SPECial paths are resolved **relative to the project root**, without the `.md` extension. 

```toml
# special.conf.toml
paths = ["docs", "src"]
```

```
project/
  special.conf.toml
  README.md                      # path: README (root file)
  docs/
    L0-security.md               # path: docs/L0-security
    auth/
      L1-authentication.md       # path: docs/auth/L1-authentication
  src/
    auth/
      L2-auth-flow.md            # path: src/auth/L2-auth-flow
      L3-token-validation.md     # path: src/auth/L3-token-validation
```

> [!note]
> SPECial is fully compatible with `mkdocs`. You can add your mkdocs `docs_dir` to `paths`, and serve SPECial files as public documentation, utilising all of SPECial features for maintaining documentation and synchronising it with live code and specification.

---

## 3. SPECial file schema

SPECial .md files include a **body** and **frontmatter**.

### 3.1 Frontmatter

Frontmatter is a YAML, TOML or JSON block at the beginning of the .md file. Frontmatter is used by SPECial to store structured metadata.

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

#### 3.1.1 Schema

|Field|Type|Required|Description|
|---|---|---|---|
|`scope`|`enum`|yes|Depth of detail: `root`, L0–L3. Also expressed in file name.|
|`summary`|`string`|yes|One-line description, used for [navigation](#33-navigation).|
|`modified`|`date`|yes|Last content change, ISO 8601.|
|`reviewed`|`date`|yes|Last verified consistency timestamp, ISO 8601.|
|`depends`|`object[]`|no|This file can become stale after changes to these files.|
|`dependents`|`object[]`|no|These files can become stale after changes to this file.|

##### 3.1.1.1 Scope

|Level|Answers|Access|
|---|---|---|
|root: Index|What domains exist in this project?|Entry point. An agent reads this first to orient and pick a domain.|
|L0: Context & Motivation|What is this domain, why does it exist, what are the stakes?|An agent reads this to decide whether this domain is relevant to its task.|
|L1: Contracts|What are the interfaces, invariants, constraints, rules?|An agent reads this to work _with_ the domain without understanding its internals.|
|L2: Structure & Flows|What are the components, how do they interact, what are the key sequences?|An agent reads this to modify or extend the domain.|
|L3: Implementation|What are the implementation patterns, edge cases, known issues, performance considerations?|An agent reads this to debug or optimize within the domain.|

##### 3.1.1.2 Depends

If changes to file X can affect file Y, then Y depends on X. For example, if changes in `L0-accessibility` can affect `L1-user-interface`, then `L1-user-interface` sets `depends: [L0-accessibility]`.

`depends` is the **source of truth** for staleness detection. Each entry has the following schema:

|Field|Type|Required|Description|
|---|---|---|---|
|`path`|`string`|yes|Path to the dependency file.|
|`section`|`string`|no|Heading in the dependency file that this file depends on. If omitted, the dependency is file-level.|
|`local`|`string`|no|Heading in this file that is dependent on the `section`. If omitted, the whole file is considered dependent.|

When `section` and `local` are specified, staleness is scoped: only changes under that heading in the dependency are relevant, and only the local heading needs review. 

Entries can use a shorthand when only `path` is needed:

```yaml
depends:
  - L0-accessibility          # shorthand, equivalent to { path: L0-accessibility }
  - path: auth/L2-user-model  # full form, needed when section/local are specified
    section: "2.3.4 Token schema"
    local: "1.2 Validation rules"
```

##### 3.1.1.3 Dependents

Inverse of `depends`. Lists files that depend on this file, used for navigating the dependency graph in the forward direction — after changing a file, its `dependents` tell you which files to check for staleness.

`dependents` carries **no semantic weight** for staleness — that is always computed from `depends`. It exists purely as a navigation aid. Every `depends` edge should have a matching `dependents` edge and vice versa.

`dependents` entries follow the same schema and shorthand rules as `depends`.

##### 3.1.1.4 Modified & Reviewed

These two fields track documentation staleness and drift.

**`modified`** is bumped when any content change is made to the document. Editing a file does _not_ automatically bump `reviewed` — a modification is not a consistency check.

**`reviewed`** is bumped when you verify that the file is faithful to all files it depends on. Reviewing does _not_ bump `modified` unless the content also changes.

The staleness rule: if Y depends on X, and `Y.reviewed < X.modified`, then Y is **potentially stale**. This requires verification. If Y is still consistent, bump `Y.reviewed`. If edits are required, bump both `Y.modified` and `Y.reviewed` — the modified change then propagates outward through Y's own dependents, and the process repeats until the entire dependency graph is consistent.

### 3.2 File naming

SPECial files encode their scope level in the file name:

```
L0-security.md
L1-authentication.md
L2-auth-flow.md
L3-token-validation.md
```

The prefix makes scope immediately visible in file listings and allows tools to infer scope without parsing frontmatter. CLI tools can use globs to efficiently query SPECial files:

```
L0-*.md  →  all domains
L1-*.md  →  all contracts
L2-*.md  →  all design docs
L3-*.md  →  all implementation docs
```

### 3.3 Navigation

SPECial uses `summary` and the dependency graph for incremental navigation — both for human readers and agents managing context budgets.

#### 3.3.1 Summary

Each file's `summary` is the authoritative one-line description of its contents. An agent encountering a file path in a `dependents` list can read just the frontmatter of the referenced file to get its summary and scope, then decide whether to load the full body.

By convention, SPECial files may list their dependants' summaries in the file body. This avoids fetching all dependants' frontmatter to learn their approximate contents. As with other content, if a dependant's summary is updated, `modified` is bumped, and changes propagate via the [staleness mechanism](#3114-modified--reviewed).

#### 3.3.2 Root index file

A **root index file** (configured as `root` in `special.conf.toml`, default: `README`) serves as the entry point, effectively an `L-1` scope. It may or may not list `L0` files as dependents — [file discovery](#32-file-naming) does not depend on it.

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
