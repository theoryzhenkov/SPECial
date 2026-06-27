---
scope: L3
summary: "Sentinel for assets/special.md — tracks when the agent reference needs updating"
modified: 2026-06-27
reviewed: 2026-06-27
depends:
  - path: index
  - path: docs/files/L0-files
  - path: docs/files/L1-files
  - path: docs/files/L2-files
  - path: docs/files/L1-files-assertions
  - path: docs/files/L2-files-assertions
  - path: docs/L0-documentation-style
  - path: docs/L0-tooling
  - path: docs/L0-divergence
---

# Agent reference sentinel

This file tracks dependencies for `assets/special.md`, the downloadable agent reference. When any dependency listed above is modified, this sentinel goes stale, signaling that `assets/special.md` needs regeneration.

The agent reference itself is not a SPECial file — it is a standalone distribution artifact. This sentinel bridges it into the dependency graph.
