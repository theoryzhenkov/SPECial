---
scope: L0
summary: "Writing conventions optimised for AI-agent consumption and human readability"
modified: 2026-02-23
reviewed: 2026-02-23
depends:
  - path: index
  - path: docs/L0-file-schema
    section: "1.1.1 Scope"
    local: "2. Principles"
dependents:
  - docs/L1-documentation-style
---

# Documentation style

This document defines writing conventions for SPECial file bodies. The primary design goal is **context efficiency** — minimising the tokens an agent must consume to extract the information it needs, while keeping documentation accessible to human readers.

## 1. References

SPECial's style conventions build on established documentation standards. When this document does not address a specific question, consult the following in order:

| Reference                                  | Focus                                                               | URL                                                                                             |
| ------------------------------------------ | ------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| Google Developer Documentation Style Guide | General technical writing: voice, tense, word choice, formatting    | [developers.google.com/style](https://developers.google.com/style)                              |
| Microsoft Writing Style Guide              | Tone, accessibility, global-readiness, UI terminology               | [learn.microsoft.com/en-us/style-guide](https://learn.microsoft.com/en-us/style-guide/welcome/) |
| Diátaxis                                   | Documentation structure: tutorials, how-tos, reference, explanation | [diataxis.fr](https://diataxis.fr)                                                              |

Where these references conflict with each other, prefer Google. Where any of them conflict with this document, this document takes precedence.

Diátaxis is particularly relevant to SPECial's scope hierarchy — its four documentation modes map loosely onto SPECial's scope levels, though the alignment is not one-to-one.

## 2. Principles

The following principles govern SPECial documentation style. Each is summarised here; the full rules are in [L1-documentation-style](L1-documentation-style.md) and worked examples are in [L2-documentation-style](L2-documentation-style.md).

1. **Front-loading.** Place the highest-value information at the beginning of every unit — document, section, and paragraph — so readers can decide early whether to continue.

2. **Voice and tone.** Use active voice, address the reader as "you", and write in a clear, direct tone free of filler, excessive politeness, and trivialising language.

3. **Declarative over narrative.** Prefer direct factual statements over storytelling or procedural narration to compress better and resist ambiguity.

4. **One concept per unit.** Each section addresses a single concept and each paragraph makes a single point, enabling agents to skip irrelevant sections confidently.

5. **Consistent terminology.** Use the same term for the same concept throughout the documentation; do not alternate between synonyms.

6. **Structured data over prose.** When information has a regular shape, prefer tables or lists over prose paragraphs.

7. **Lists.** Use the correct list type for the content, maintain parallel structure, and precede lists with a complete introductory sentence.

8. **Minimal filler.** Omit sentences that exist only for transition, courtesy, or rhetorical effect — every sentence should state a fact, define a rule, or provide an example.

9. **Headings.** Use sentence case, decimal numbering, strict hierarchy, and descriptive noun or verb phrases as headings.

10. **Examples and code blocks.** Place examples immediately after the rule they illustrate, keep them minimal, and annotate code blocks with the language identifier.

11. **Cross-references.** Use anchor links within files and `depends`/`dependents` paths across files, with descriptive link text and conditions stated before the link.

12. **Accessibility.** Write for a global, diverse audience with descriptive alt text, unambiguous date formats, serial commas, and no directional language.
