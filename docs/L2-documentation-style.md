---
scope: L2
summary: "Examples, rationale, and anti-patterns for documentation style rules"
modified: 2026-02-23
reviewed: 2026-02-23
depends:
  - path: docs/L1-documentation-style
---

# Documentation style — examples and rationale

Worked examples, prefer/avoid tables, and extended rationale for each rule in [L1-documentation-style](L1-documentation-style.md).

## 1. Front-loading

Front-loading mirrors the scope hierarchy: a document's opening paragraph functions like a `summary` — it lets you decide whether to invest further tokens.

**Anti-pattern — deferred point:**

> In order to understand how tokens expire, you first need to consider how sessions are created…

The reader must consume the entire paragraph before reaching the actual topic. Restructure so the key assertion comes first.

## 2. Voice and tone

### 2.1. Active voice

| Prefer                              | Avoid                                   |
| ----------------------------------- | --------------------------------------- |
| The server sends an acknowledgment. | An acknowledgment is sent.              |
| Set the `timeout` flag to 30.       | The `timeout` flag should be set to 30. |

**Acceptable passive voice.** Passive voice is acceptable when the actor is genuinely irrelevant ("The database was purged in January") or when you want to emphasise the object over the actor ("The file is saved").

### 2.2. Second person

| Prefer                                                | Avoid                                                 |
| ----------------------------------------------------- | ----------------------------------------------------- |
| You can configure the cache by editing `config.yaml`. | One can configure the cache by editing `config.yaml`. |
| If you omit the flag, the default applies.            | If the flag is omitted, the default applies.          |

"We recommend" is vague — state the recommendation directly instead.

### 2.3. Tone

- "Set the flag to 30" rather than "Please set the flag to 30".
- If a procedure requires multiple steps, words like "simply" and "just" mislead.

## 3. Declarative over narrative

| Prefer                                           | Avoid                                                                        |
| ------------------------------------------------ | ---------------------------------------------------------------------------- |
| Tokens expire after 3600 seconds.                | What happens is that after 3600 seconds the tokens will expire.              |
| Each request requires an `Authorization` header. | You need to make sure you include an `Authorization` header in each request. |
| The cache invalidates on write.                  | When a write operation occurs, the system proceeds to invalidate the cache.  |

Narrative forms ("what happens is…", "the system proceeds to…") add tokens without adding information.

## 4. One concept per unit

**Anti-pattern — umbrella sections.** Headings like "Miscellaneous", "Other considerations", or "Notes" force full reads because the heading carries no signal about its content. Split into sections with descriptive headings instead.

**Rationale.** Tight scope per unit allows agents to skip irrelevant sections confidently — if the heading does not match the query, the content will not either.

## 5. Consistent terminology

If a concept is called `scope` in one document, it is `scope` everywhere — not "level", "tier", or "depth" in other places. Terminological consistency allows agents to search and cross-reference reliably.

## 6. Structured data over prose

**Anti-pattern — hybrid list items.** A list item that contains multiple paragraphs of explanation is a section masquerading as a list item. If an item needs that much context, promote it to its own section.

**When to choose prose vs. structured format.** If you find yourself writing "X is A, Y is B, and Z is C" in a paragraph, a table is likely clearer.

## 7. Lists

### 7.1. Parallel structure

| Prefer                                               | Avoid                                                       |
| ---------------------------------------------------- | ----------------------------------------------------------- |
| Set the flag. Configure the path. Run the migration. | Setting the flag, to configure the path, run the migration. |
| Reports, metrics, and alerts                         | Reports, metric data, and alerting systems                  |

Start every item in a list with the same part of speech. If the first item begins with a verb, all items begin with a verb.

### 7.2. Punctuation

**Anti-pattern — inconsistent punctuation within one list.** Ending some items with a period and others without creates visual noise. Apply one rule to the entire list: periods for complete sentences, no periods for fragments.

### 7.3. Introductions

**Anti-pattern — missing introductory sentence.** Starting a list without a preceding sentence leaves the purpose of the list ambiguous. Always precede a list with a complete introductory sentence or a sentence ending in a colon.

## 8. Minimal filler

| Filler pattern                          | Why it wastes tokens                                      |
| --------------------------------------- | --------------------------------------------------------- |
| "It is worth noting that…"              | Adds tokens, zero information.                            |
| "As mentioned in the previous section…" | Back-reference without a link. Use an anchor or remove.   |
| "This section will cover…"              | The heading already states the topic.                     |
| "In conclusion…"                        | Summaries belong in `summary`, not in closing paragraphs. |
| "Please note that…"                     | Drop "please note that" and state the fact directly.      |
| "Simply run the command…"               | "Simply" trivialises difficulty. State the instruction.   |

## 9. Headings

**Sentence case rationale.** Sentence case is easier to scan and avoids ambiguity about which words qualify as "major" in title case.

**Numbering rationale.** Numbered headings enable unambiguous cross-references and survive reordering better than position-dependent references ("the section above").

**Hierarchy rationale.** Deeper nesting beyond three levels signals that a section should be split into a separate file or restructured.

**Anti-pattern — question-form and gerund headings.** "How does X work?" and "Configuring the cache" add tokens and delay the answer. Use "X lifecycle" or "Configure the cache" instead.

## 10. Examples and code blocks

**Anti-pattern — wall of code.** Examples that demonstrate multiple concepts at once violate the one-concept-per-unit principle. One example, one concept.

**Setup code.** If an example requires setup code unrelated to the point, either omit the setup or note it with a comment:

```python
# ... existing setup ...
result = process(data)
```

## 11. Cross-references

| Prefer                                                                              | Avoid                                                    |
| ----------------------------------------------------------------------------------- | -------------------------------------------------------- |
| For more information, see [Scope levels](../L0-file-schema#scope).                  | Click [here](../L0-file-schema#scope).                   |
| The [staleness mechanism](../L0-file-schema#modified--reviewed) propagates changes. | This is described in [this document](../L0-file-schema). |

**Conditional clauses first.** "If you need to customise scopes, see [Scope levels](../L0-file-schema#scope)." Placing the condition first lets readers skip the sentence early if it does not apply.

**Forward references.** Avoid forward references where possible. If section 2 depends on a concept from section 5, consider reordering. When forward references are unavoidable, use an anchor link so the reader can jump directly.

## 12. Accessibility

- **Alt text example.** Write "Diagram showing the three-step authentication flow: request, challenge, response" rather than "Auth diagram" or "Image".
- **Date format.** Prefer `2026-01-05` (ISO 8601) or "5 January 2026" over `01/05/2026`, which is ambiguous across locales.
- **Directional language.** "See the [Token lifecycle](#9-headings) section" rather than "see the section below".
