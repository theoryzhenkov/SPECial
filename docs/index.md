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

## 1. Overview

SPECial is a lightweight specification / documentation standard. It is designed to perform optimally with AI agents and minimise specification drift. To this end, SPECial avoids relying on tooling, custom formats or any external state — all the information required to traverse and maintain your project's documentation is available within files themselves. 

SPECial attempts to reduce irrelevant context LLM agents accumulate, achieving better performance for projects with large documentations. For this purpose, SPECial provides guidance towards file body formatting and structure alongside the file organisation directives. 

SPECial was designed by Theo Ryzhenkov and is distributed under MIT license. Documentation for SPECial follows SPECial standard. 

!!! note

    Currently, SPECial doesn't provide any tooling: linters, formatters, drift detection, etc. If the standard's design receives wider adoption, I plan to develop a SPECial CLI / SDK. Often, the documentation is written as if tooling existed — this is in order to a) specify future tooling, and design with it in mind b) to allow agents to deterministically follow the specification.

## 1. Quickstart



