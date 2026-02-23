---
scope: L0
summary: "Configuration, path resolution, project layout"
modified: 2026-02-23
reviewed: 2026-02-23
depends:
  - path: index
    local: "Pages"
dependents:
  - docs/L0-file-schema
---

# Project Structure

A SPECial project is a collection of files downstream from a **project root**.

## 1. Configuration

The project root is identified by the presence of `special.conf.toml`. If no `special.conf.toml` is present in the working directory, SPECial treats working directory as project root with default values.

```toml
# special.conf.toml
root = "README"         # entry point file (default: "README")
paths = ["."]           # directories containing SPECial files (default: ["."])
```

| Field   | Type       | Default    | Description                                                      |
| ------- | ---------- | ---------- | ---------------------------------------------------------------- |
| `root`  | `string`   | `"README"` | Entry point file, without `.md` extension.                       |
| `paths` | `string[]` | `["."]`    | Directories to scan for SPECial files, relative to project root. |

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

    SPECial is fully compatible with `mkdocs`. You can add your mkdocs `docs_dir` to `paths`, and serve SPECial files as public documentation, utilising all of SPECial features for maintaining documentation and synchronising it with live code and specification.
