---
scope: L1
summary: "Actionable rules for writing SPECial-compliant documentation"
modified: 2026-02-23
reviewed: 2026-02-23
depends:
  - path: docs/L0-documentation-style
dependents:
  - docs/L2-documentation-style
---

# Documentation style — rules

Concise directives for writing SPECial documentation. For context and motivation see [L0-documentation-style](L0-documentation-style.md). For worked examples see [L2-documentation-style](L2-documentation-style.md).

## 1. Front-loading

Place the highest-value information at the beginning of every unit: document, section, and paragraph.

- **Document level.** The opening paragraph states what the document covers and why it exists. A reader consuming only this paragraph must be able to decide whether to continue.
- **Section level.** Open each section with a sentence that summarises the rule or concept. Detail, rationale, and examples follow.
- **Paragraph level.** The first sentence carries the key assertion. Subsequent sentences support or qualify it.

Do not defer the point with lead-ins ("In order to understand X, you first need to consider Y…").

## 2. Voice and tone

### 2.1. Active voice

Use active voice. Make it clear who or what performs an action. Passive voice is acceptable only when the actor is genuinely irrelevant or when you want to emphasise the object over the actor.

### 2.2. Second person

Address the reader as "you". Use "you" for instructions, explanations, and descriptions of reader actions. Avoid first person ("we", "our") except when referring to a specific, named group.

### 2.3. Tone

Write in a clear, direct tone.

- Avoid excessive politeness in instructions.
- Avoid words that trivialise difficulty: "simply", "just", "easily", "quickly".
- Avoid exclamation marks, slang, emoji, and internet abbreviations.
- Write for a global audience. Avoid idioms, culture-specific references, and humour that depends on local context.

## 3. Declarative over narrative

Prefer direct factual statements over storytelling or procedural narration. Declarative prose compresses better and resists ambiguity.

## 4. One concept per unit

Each section addresses a single concept. Each paragraph makes a single point.

- When a section grows beyond what its heading promises, split it.
- When a paragraph makes two unrelated claims, break it in two.
- Avoid "umbrella" sections that group loosely related items under a vague heading ("Miscellaneous", "Other considerations", "Notes").

## 5. Consistent terminology

Use the same term for the same concept throughout the documentation. Do not alternate between synonyms for stylistic variety.

- Define domain terms on first use.
- After first definition, use terms without re-explanation.
- If a term has a common English meaning that differs from its domain meaning, make the distinction explicit once.

## 6. Structured data over prose

When information has a regular shape — fields with types, options with descriptions, steps with outcomes — prefer tables or lists over prose paragraphs.

- Prose is appropriate for explanations and rationale.
- Structured formats are appropriate for reference data.
- Avoid hybrid forms where a list item contains multiple paragraphs of explanation. If an item needs that much context, it is a section.

## 7. Lists

Use numbered lists for sequential steps or ranked items. Use bulleted lists for unordered sets. Use description lists (bold term followed by explanation) for term–definition pairs.

- **Parallel structure.** Start every item in a list with the same part of speech.
- **Punctuation.** End list items with a period if they contain a verb or form a complete sentence. Omit the period for single-word items, code-only items, or fragments. Be consistent within a single list.
- **Introductions.** Precede a list with a complete introductory sentence. If the sentence leads directly into the list, end it with a colon.

## 8. Minimal filler

Omit sentences that exist only for transition, courtesy, or rhetorical effect. Every sentence should state a fact, define a rule, or provide an example. If it does none of these, remove it.

## 9. Headings

Headings are the primary navigation mechanism for both agents and humans.

- **Sentence case.** Capitalise only the first word and proper nouns.
- **Numbering.** Use decimal numbering (`1.`, `1.1.`, `1.1.1.`).
- **Hierarchy.** Do not skip heading levels. An `h3` (`###`) appears only under an `h2` (`##`). Prefer at most three levels of nesting.
- **Naming.** Use noun phrases for conceptual headings and bare infinitive verbs for task-based headings. Do not use question-form or gerund-led headings. Do not place links inside headings.

## 10. Examples and code blocks

- Place examples immediately after the rule they illustrate.
- Annotate code blocks with the language identifier for syntax highlighting.
- Keep examples minimal — show exactly the pattern being discussed, nothing more.
- If an example requires unrelated setup code, either omit the setup or note it with a comment (`# ... existing setup ...`).
- One example, one concept — matching the [one-concept-per-unit](#4-one-concept-per-unit) principle.
- Format inline code references — file names, field names, command names, flags — in code font. Use **bold** for UI element names.

## 11. Cross-references

Use Markdown anchor links for references within the same file. Use SPECial `depends`/`dependents` paths for cross-file references.

- **Descriptive link text.** Write link text that tells the reader what they will find at the destination. Place the most important words first. Do not use a URL as link text.
- **Conditional clauses first.** When a cross-reference is conditional, state the condition before the link.
- Do not link to the same destination multiple times on the same page unless the page is long enough to justify it.
- Avoid forward references where possible. When unavoidable, use an anchor link.

## 12. Accessibility

Write for a global, diverse audience.

- Provide descriptive alt text for all images.
- Use unambiguous date formats: `2026-01-05` (ISO 8601) or "5 January 2026".
- Use serial (Oxford) commas in all lists.
- Avoid directional language ("the image above", "the section below") — use anchor links or heading references instead.
