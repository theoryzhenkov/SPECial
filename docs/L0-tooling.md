---
scope: L0
summary: "SPECial CLI overview"
modified: 2026-06-27
reviewed: 2026-06-27
depends:
  - path: index
  - path: docs/files/L0-files
  - path: docs/files/L1-files
  - path: docs/files/L2-files
  - path: docs/files/L1-files-assertions
  - path: docs/files/L2-files-assertions
dependents:
  - docs/L3-agent-reference
---

# Tooling

SPECial CLI exists to automate the parts that are tedious or error-prone to do by hand:

- Validating frontmatter against the schema
- Detecting stale files via the `depends` / `modified` / `reviewed` graph
- Keeping the dependency graph consistent (matching `depends` / `dependents` edges)
- Reporting [assertion coverage](files/L1-files-assertions.md) — the [realization](files/L1-files-assertions.md#5-realization) signal
- Indexing [deviation registers](L0-divergence.md) and surfacing a spec's recorded divergences
- Validating domain/subdomain encoding (directory vs filename agreement)

!!! note
CLI is not implemented: this document is a stub for a future CLI L0.
