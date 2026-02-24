---
scope: root
summary: "SPECial standard"
modified: 2026-02-23
reviewed: 2026-02-23
dependents:
  - docs/L0-project-structure
  - docs/L0-file-structure
  - docs/L0-documentation-style
  - docs/L0-tooling
---

# SPECial

## 1. Overview

SPECial is a lightweight specification standard. It is designed to improve the efficiency of creating, using and maintaining documentation. By providing a style guide, a file & directory structure, and an operational pipeline, SPECial assists human & LLM agents both in managing context, navigating documentation, ensuring specification consistency with code and other tasks.

## When to use SPECial?

Use SPECial when you have non-trivial amount of interconnected domains that need documentation and are subject to modification over time. SPECial is specifically optimised towards projects that employ LLM agents as active contributors / maintainers, but it is perfectly suitable for human-only usage.

SPECial's main implementation uses Markdown files as a basis of its [file structure](L0-file-structure.md). You can use other file formats provided you have some metadata storage solution (either built-in into the file format, or external). 

## 2. Quickstart

### 2.1. Add your domain files

Create an L0 file for each major domain in your project. The file name prefix encodes its scope level. 

```
docs/
  L0-auth.md        # context & motivation for the auth domain
  L0-payments.md    # context & motivation for payments
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

Add deeper files as the domain grows: `L1-` for contracts and invariants, `L2-` for structure and flows, `L3-` for implementation details. See [Scope](L0-file-structure.md#111-scope) for the full description.

### 2.2. Link files with depends and dependents

When changes to file X can make file Y stale, declare the relationship in Y's frontmatter.

```yaml
# docs/L1-auth.md
---
scope: L1
summary: "Auth contracts and session invariants"
modified: 2026-01-05
reviewed: 2026-01-05
depends:
  - path: docs/L0-auth
---
```

Add the matching `dependents` entry in `docs/L0-auth.md` pointing back to `docs/L1-auth`. SPECial uses this graph to surface potentially stale files: if `L0-auth` is modified after `L1-auth` was last reviewed, `L1-auth` is flagged for verification. See [File structure](L0-file-structure.md) for the full staleness rules and shorthand syntax.

### 2.3. What next?

The full standard is covered across three L0 documents. [File structure](L0-file-structure.md) goes deeper on frontmatter, scope levels, and the staleness mechanism you saw in step 2.2. [Project structure](L0-project-structure.md) covers configuration and path resolution if you need to customise the layout. [Documentation style](L0-documentation-style.md) covers writing conventions for the file bodies themselves.

---

SPECial was designed by [Theo Ryzhenkov](https://home.theor.net) and is distributed under MIT license. Documentation for SPECial follows SPECial standard.
