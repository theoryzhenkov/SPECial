# Agent Setup

SPECial provides a downloadable reference file that gives AI agents the context they need to work with SPECial-documented projects.

## Install

Download `special.md` and place it in your agent's documentation directory:

```bash
# Claude Code
curl -o .claude/docs/SPECial.md https://the-o-space.github.io/special/special.md

# Or manually download and place at .claude/docs/SPECial.md
```

Once installed, the agent can navigate SPECial frontmatter, follow the dependency graph, respect staleness rules, and work with assertion tables.

## What the reference covers

- Project orientation (`special.conf.toml`, root file, paths)
- Frontmatter schema and all fields
- Scope levels (root, L0--L3) and when to read each
- Staleness detection (`modified`/`reviewed` rules)
- Assertions (tables, test linking, coverage workflow)
- Guidance for modifying SPECial files and spec-covered code

## Keeping it up to date

The reference tracks the current version of the SPECial standard. Re-download it when SPECial releases updates.
