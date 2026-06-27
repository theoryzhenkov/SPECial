---
scope: L0
summary: "Why SPECial files carry metadata — staleness, navigation, context budgeting"
modified: 2026-06-27
reviewed: 2026-06-27
depends:
  - path: index
dependents:
  - docs/files/L1-files
  - docs/files/L2-files
  - docs/files/L1-files-assertions
  - docs/files/L2-files-assertions
  - docs/L0-tooling
  - docs/L3-agent-reference
---

# Files

A SPECial project is a collection of spec files downstream from a **project root** located at **paths**. Together, these files form a documentation / specification. This document explains *why* those files carry structured metadata; the contracts are in [L1-files](L1-files.md) and the structure in [L2-files](L2-files.md).

## 1. The problem

Specifications drift. Specs drift from each other when one changes and its dependents are not updated. Specs drift from code when the implementation changes and no one checks the spec. Tests drift from specs when they verify implementation details that no longer reflect the specification. In practice these artifacts are maintained independently, and the gaps widen silently.

The root cause is operational, not technical: enforcement depends on humans remembering to check, and nothing requires the links that would make checking automatic. Both specification-driven development (SDD) and test-driven development (TDD) suffer the same bottleneck — manual discipline.

## 2. The approach

SPECial makes drift *derivable* rather than asserted. Each file carries a small amount of structured [frontmatter](L1-files.md#1-frontmatter): what it depends on, when it was last modified, and when it was last verified consistent. From this, a staleness rule (`reviewed < modified` along a dependency edge) computes which files are potentially stale — no hand-maintained status required.

[Assertions](L1-files-assertions.md) extend the same idea to the spec-to-code gap: a spec declares testable claims, tests link back to them, and coverage — hence [realization](L1-files-assertions.md#5-realization) — is derived from the links rather than typed in.

## 3. Navigation and context

SPECial is built for agents and humans managing a context budget. Files declare a one-line `summary` and a dependency graph, so a reader can orient from the root index, read summaries to pick relevant domains, and load full bodies only when needed. Scope levels (L0–L3) gate depth: read the level that matches the task.

## 4. Project structure

SPECial provides a project structure mainly to enable the [SPECial CLI](../L0-tooling.md); otherwise, SPECial files form independent cliques and can be located anywhere. If you are not using the CLI, consider the project structure a recommendation. Configuration and path resolution are covered in [L2-files](L2-files.md#3-configuration).
