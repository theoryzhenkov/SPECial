---
scope: L3
summary: "Sentinel for assets/special.md — tracks when the agent reference needs updating"
modified: 2026-03-16
reviewed: 2026-03-16
depends:
  - path: index
  - path: docs/L0-file-structure
  - path: docs/L0-project-structure
  - path: docs/L0-documentation-style
  - path: docs/L0-assertions
  - path: docs/L1-assertions
  - path: docs/L0-tooling
---

# Agent reference sentinel

This file tracks dependencies for `assets/special.md`, the downloadable agent reference. When any dependency listed above is modified, this sentinel goes stale, signaling that `assets/special.md` needs regeneration.

The agent reference itself is not a SPECial file — it is a standalone distribution artifact. This sentinel bridges it into the dependency graph.
