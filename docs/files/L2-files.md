---
scope: L2
summary: "File naming, domain structure, navigation, configuration, path resolution"
modified: 2026-06-27
reviewed: 2026-06-27
depends:
  - path: index
  - path: docs/files/L0-files
  - path: docs/files/L1-files
dependents:
  - docs/L0-tooling
  - docs/L3-agent-reference
---

# File structure

This document covers how SPECial files are named, grouped into domains, navigated, and resolved. Frontmatter contracts and staleness invariants are in [L1-files](L1-files.md); motivation in [L0-files](L0-files.md).

## 1. File naming

SPECial files encode metadata in two places: **frontmatter** (source of truth) and the **file path** (discoverability aid). The file path can encode scope, [type](L1-files.md#116-type), [lifecycle](L1-files.md#115-lifecycle), and [domain](#13-domain-and-subdomain) using filename prefixes or directory segments interchangeably.

### 1.1. Scope prefix

Every SPECial file encodes its scope level as a filename prefix:

```
L0-security.md
L1-authentication.md
L2-auth-flow.md
L3-token-validation.md
```

The prefix makes scope immediately visible in file listings and allows tools to infer scope without parsing frontmatter.

### 1.2. Type and lifecycle encoding

Type and lifecycle can each be expressed as an **uppercase filename prefix** or a **lowercase directory segment**. These two forms are interchangeable — a project may use either or both, as long as the encoding is consistent with frontmatter.

| Encoding              | Filename prefix | Directory segment |
| --------------------- | --------------- | ----------------- |
| Type (e.g. `PLAN`)    | `PLAN-`         | `plan/`           |
| Lifecycle `ephemeral` | `EPH-`          | `eph/`            |

The ordering of lifecycle, type, and scope segments in the file path is a project-level convention. SPECial does not enforce a particular order — choose whichever grouping makes navigating your documentation easiest and stay consistent within the project. The [SPECial CLI](../L0-tooling.md) reads the configured order from [`naming_order`](#3-configuration) in `special.conf.toml`.

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

### 1.3. Domain and subdomain

A *domain* is a cohesive area of a system — auth, payments, notifications. A *subdomain* is a finer subdivision, such as `oidc` or `sessions` within `auth`. Domains are an **optional** organizational layer: a small project may keep files flat; a larger one groups files into domain directories.

Unlike [type](L1-files.md#116-type) and [lifecycle](L1-files.md#115-lifecycle), domain has **no frontmatter field**. It is derived from the file path, following the same derive-over-assert principle as [realization](L1-files-assertions.md#5-realization): the path is the single source of truth, so domain cannot drift from it.

Domain and subdomain are materialized as directories under a [`paths`](#4-path-resolution) entry, and may optionally be mirrored in the filename for flat-list self-description. The forms are interchangeable, like type and lifecycle encoding:

```
docs/L0-auth.md                              # filename-only: domain in name, no directory
docs/auth/L0-auth.md                         # directory + filename: domain in both
docs/auth/oidc/L2-auth-oidc-token-flow.md    # directory + filename, with subdomain
docs/auth/oidc/L2-token-flow.md              # directory-only: domain in path, not name
```

When a domain directory is present, it is authoritative: if the filename also encodes the domain, it must match the directory (tooling flags a mismatch as drift). When no directory is present, the filename's domain segment is the source.

Domain is **not** a [`naming_order`](#3-configuration) segment. `naming_order` governs only the `lifecycle`/`type`/`scope` prefixes; the domain/subdomain/name form the trailing name portion. The full encoded filename is therefore:

```
<lifecycle>-<type>-<scope>-<domain>-<subdomain>-<name>.md
```

with any of lifecycle/type/domain/subdomain omitted when unused. A domain's L0 lives inside its domain directory when one is used:

```
docs/
  auth/                                 # domain
    L0-auth.md                           # domain motivation
    L1-auth.md                           # contracts
    oidc/                                # subdomain
      L2-auth-oidc-token-flow.md
  payments/
    L0-payments.md
```

### 1.4. Glob patterns

The encoding styles give different glob ergonomics. Choose whichever fits the project:

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

# domain directories
auth/*.md            →  all files in the auth domain
auth/oidc/*.md       →  all files in the auth/oidc subdomain
*/L0-*.md            →  every domain's L0
```

## 2. Navigation

SPECial uses `summary` and the dependency graph for incremental navigation — both for human readers and agents managing context budgets.

### 2.1. Summary

Each file's `summary` is the authoritative one-line description of its contents. An agent encountering a file path in a `dependents` list can read just the frontmatter of the referenced file to get its summary and scope, then decide whether to load the full body.

By convention, SPECial files may list their dependents' summaries in the file body. This avoids fetching all dependents' frontmatter to learn their approximate contents. As with other content, if a dependent's summary is updated, `modified` is bumped, and changes propagate via the [staleness mechanism](L1-files.md#114-modified-reviewed).

### 2.2. Root index file

A **root index file** (configured as `root` in `special.conf.toml`, default: `README`) serves as the entry point, effectively an `L-1` scope. It may or may not list `L0` files as dependents — [file discovery](#1-file-naming) does not depend on it.

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

## 3. Configuration

The project root is identified by the presence of `special.conf.toml`. If no `special.conf.toml` is present in the working directory, SPECial treats the working directory as project root with default values.

```toml
# special.conf.toml
root = "README"                            # entry point file (default: "README")
paths = ["."]                              # directories containing SPECial files (default: ["."])
naming_order = ["lifecycle", "type", "scope"]  # segment order in file paths (default)
```

| Field          | Type       | Default                          | Description                                                                                                         |
| -------------- | ---------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `root`         | `string`   | `"README"`                       | Entry point file, without `.md` extension.                                                                          |
| `paths`        | `string[]` | `["."]`                          | Directories to scan for SPECial files, relative to project root.                                                    |
| `naming_order` | `string[]` | `["lifecycle", "type", "scope"]` | Order of metadata segments in file paths. Used by CLI for parsing and generating file names. See [File naming](#1-file-naming). |

## 4. Path resolution

All SPECial paths are resolved **relative to the project root**, without the `.md` extension.

```toml
# special.conf.toml
paths = ["docs", "src"]
```

```
project/
  special.conf.toml              # paths = ["docs", "src"]
  docs/
    index.md                     # path: docs/index (root file)
    security/                    # domain
      L0-security.md             # path: docs/security/L0-security
    auth/                        # domain
      L1-auth.md                 # path: docs/auth/L1-auth
      oidc/                      # subdomain
        L2-auth-oidc-token-flow.md  # path: docs/auth/oidc/L2-auth-oidc-token-flow
  src/
    auth/
      L3-auth-token-validation.md  # path: src/auth/L3-auth-token-validation
```

!!! note

    SPECial is fully compatible with `mkdocs` (and possibly other markdown static site builders). You can add your mkdocs `docs_dir` to `paths`, and serve SPECial files as public documentation, utilising all of SPECial features for maintaining documentation and synchronising it with live code and specification.

## 5. Assertions

| ID                 | Sev. | Assertion                                                                          |
| ------------------ | ---- | ---------------------------------------------------------------------------------- |
| scope-prefix       | MUST | A file's scope is encoded as its filename prefix.                                  |
| domain-derived     | MUST | A file's domain is derived from its path; it has no frontmatter field.             |
| domain-not-segment | MUST | Domain is not a `naming_order` segment.                                            |
| paths-relative     | MUST | All spec paths are resolved relative to the project root, without the `.md` extension. |
