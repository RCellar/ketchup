# Cite and Tag Reference

Citation and tagging rules for ketchup report outputs.

## Citation Rules

| Claim Type | Format |
|---|---|
| Verifiable fact with URL | Hyperlink inline: `"claim ([source](url))"` |
| Fact, no URL available | Append `_(unsourced)_` |
| Probabilistic / inference claim | Append `_(~inferred: brief basis)_` |
| URL not verified to resolve | Append `_(link unverified)_` |
| Claim sourced only from training data (no search result) | Flag explicitly — do NOT present as if independently verified |

**Never present an unsourced inference as a bare assertion.**

**Never use "intuition" to describe LLM reasoning.** Use "pattern-match," "aggregate likelihood," or "low-confidence extrapolation."

## Tagging (Obsidian Frontmatter)

All ketchup reports use **YAML frontmatter `tags:`** for tagging — not inline blockquotes.

```yaml
tags:
  - ketchup
  - selinux
  - windows-sysadmin
```

**Rules:**
- Tags live in frontmatter only — Obsidian's search and Dataview index these reliably
- Use descriptive, searchable terms — not generic labels like `misc` or `general`
- Sanitize: no spaces (use hyphens), cannot start with a number, lowercase
- Skip tags on ephemeral replies (brief acknowledgments, one-liners)

### Notebook Format

When outputting to `--fmt notebook`, tags are stored in `metadata.ketchup.tags` (notebook-level JSON) instead of YAML frontmatter. The same sanitization rules apply. Individual cells use `cell.metadata.tags` for structural tags (`ketchup:generated`, `ketchup:section`, etc.) — these are not content tags and should not be confused with the report's topic tags.

## Common Mistakes

| Mistake | Fix |
|---|---|
| Inference stated as bare fact | Add `_(~inferred: basis)_` |
| "My intuition says..." | Replace with "pattern-match suggests..." or "low-confidence extrapolation:" |
| Vague tags like `#misc` or `#general` | Use specific, queryable terms |
| URL linked without verifying it resolves | Verify or note `_(link unverified)_` |
| Tags in a blockquote at end of document | Move to YAML frontmatter `tags:` list |
