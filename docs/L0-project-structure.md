---
scope: L0
summary: "SPECial project structure standard"
modified: 2026-02-27
reviewed: 2026-02-27
depends:
  - path: index
    local: "Pages"
dependents:
  - docs/L0-file-structure
  - docs/L0-tooling
---

# SPECial project structure

A SPECial project is a collection of [SPECial files](L0-file-structure.md) downstream from a **project root** located at **paths**. Together, these files form a documentation / specification.

SPECial provides a project structure mainly to allow [SPECial CLI](L0-tooling.md); otherwise, [SPECial files](L0-file-structure.md) form independent cliques and can be located anywhere. If you are not using the [SPECial CLI](L0-tooling.md), consider SPECial project structure standard merely a recommendation.

## 1. Configuration

The project root is identified by the presence of `special.conf.toml`. If no `special.conf.toml` is present in the working directory, SPECial treats working directory as project root with default values.

```toml
# special.conf.toml
root = "README"                            # entry point file (default: "README")
paths = ["."]                              # directories containing SPECial files (default: ["."])
naming_order = ["lifecycle", "type", "scope"]  # segment order in file paths (default)
```

| Field          | Type       | Default                          | Description                                                                                                                                         |
| -------------- | ---------- | -------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `root`         | `string`   | `"README"`                       | Entry point file, without `.md` extension.                                                                                                          |
| `paths`        | `string[]` | `["."]`                          | Directories to scan for SPECial files, relative to project root.                                                                                    |
| `naming_order` | `string[]` | `["lifecycle", "type", "scope"]` | Order of metadata segments in file paths. Used by CLI for parsing and generating file names. See [File naming](L0-file-structure.md#2-file-naming). |

## 2. Path Resolution

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

!!! note

    SPECial is fully compatible with `mkdocs` (and possibly other markdown static site builders). You can add your mkdocs `docs_dir` to `paths`, and serve SPECial files as public documentation, utilising all of SPECial features for maintaining documentation and synchronising it with live code and specification.
