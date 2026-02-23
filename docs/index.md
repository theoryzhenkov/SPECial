---
scope: root
summary: "SPECial standard — project documentation index"
modified: 2026-02-23
reviewed: 2026-02-23
dependents:
  - docs/L0-project-structure
  - docs/L0-file-schema
---

# SPECial

SPECial is a lightweight specification / documentation standard. It is designed to provide a solid foundation for spec-driven software development, but is adaptable to other domains. SPECial is designed primarily for markdown — this document format was chosen for its wide adoption and simplicity.

!!! note

    Currently, SPECial doesn't provide any tooling: linters, formatters, drift detection, etc. If the standard's design receives wider adoption, I plan to develop a SPECial CLI / SDK. Often, the documentation is written as if tooling existed — this is in order to a) specify future tooling, and design with it in mind b) to allow agents to deterministically follow the specification.

## Pages

| Page                                         | Scope | Summary                                        |
| -------------------------------------------- | ----- | ---------------------------------------------- |
| [Project Structure](L0-project-structure.md) | `L0`  | Configuration, path resolution, project layout |
| [File Schema](L0-file-schema.md)             | `L0`  | Frontmatter, file naming, navigation           |
