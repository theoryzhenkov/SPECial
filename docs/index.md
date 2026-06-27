---
scope: root
summary: "SPECial standard"
modified: 2026-06-27
reviewed: 2026-06-27
dependents:
  - docs/files/L0-files
  - docs/files/L1-files
  - docs/files/L2-files
  - docs/files/L1-files-assertions
  - docs/files/L2-files-assertions
  - docs/L0-documentation-style
  - docs/L0-tooling
  - docs/L0-divergence
  - docs/L3-agent-reference
---

# SPECial

## 1. Overview

SPECial is a specification standard for spec-driven development. It provides a file structure, an assertion requirement, and an operational pipeline that help human and LLM agents manage context, navigate documentation, and keep specifications consistent with code.

## 1.1. When to use SPECial

Use SPECial when you have non-trivial amount of interconnected domains that need documentation and are subject to modification over time. SPECial is specifically optimised towards projects that employ LLM agents as active contributors / maintainers, but it is perfectly suitable for human-only usage.

SPECial's main implementation uses Markdown files as a basis of its [file structure](files/L0-files.md). You can use other file formats provided you have some metadata storage solution (either built-in into the file format, or external). 

## 2. Quickstart

### 2.1. Add your domain files

Create an L0 file for each major domain in your project. The file name prefix encodes its scope level. 

```
docs/
  auth/L0-auth.md        # context & motivation for the auth domain
  payments/L0-payments.md
```

Each file requires frontmatter with at minimum `scope`, `summary`, `modified`, and `reviewed`.

```yaml
---
scope: L0
summary: "Auth domain — purpose, constraints, and threat model"
modified: 2026-01-05
reviewed: 2026-01-05
---
```

Add deeper files as the domain grows: `L1-` for contracts and invariants, `L2-` for structure and flows, `L3-` for implementation details. See [Scope](files/L1-files.md#111-scope) for the full description.

### 2.2. Link files with depends and dependents

When changes to file X can make file Y stale, declare the relationship in Y's frontmatter.

```yaml
# docs/auth/L1-auth.md
---
scope: L1
summary: "Auth contracts and session invariants"
modified: 2026-01-05
reviewed: 2026-01-05
depends:
  - path: docs/auth/L0-auth
---
```

Add the matching `dependents` entry in `docs/auth/L0-auth.md` pointing back to `docs/auth/L1-auth`. SPECial uses this graph to surface potentially stale files: if `L0-auth` is modified after `L1-auth` was last reviewed, `L1-auth` is flagged for verification. See [File contracts](files/L1-files.md) for the full staleness rules and shorthand syntax.

### 2.3. What next?

The full standard is covered across its L0 and L1 documents. [Files](files/L0-files.md) explains why SPECial files carry metadata. [File contracts](files/L1-files.md) defines the frontmatter schema and staleness invariants. [File structure](files/L2-files.md) covers naming, domain structure, configuration, and path resolution. [Documentation style](L0-documentation-style.md) covers writing conventions for the file bodies themselves.

To make specs testable, see [Assertions](files/L1-files-assertions.md) — the per-section requirement for declaring verifiable claims and linking them to tests; assertion coverage is also how SPECial tracks [realization](files/L1-files-assertions.md#5-realization). To record intentional divergences between spec and implementation, see [Divergence](L0-divergence.md). For AI agent integration, see [Agent setup](agent-setup.md).

---

SPECial was designed by [Theo Ryzhenkov](https://home.theor.net) and is distributed under MIT license. Documentation for SPECial follows SPECial standard.
