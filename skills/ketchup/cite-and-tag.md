# Cite and Tag Reference

## Citation Rules

| Claim Type | Format |
|---|---|
| Verifiable fact with URL | Hyperlink inline: `"registers offset drafts ([source](url))"` |
| Fact, no URL available | Append `_(unsourced)_` |
| Probabilistic / inference claim | Append `_(~inferred: brief basis)_` |
| URL not verified to resolve | Append `_(link unverified)_` |
| Claim sourced only from training data (no search result) | Flag explicitly — do NOT present as if independently verified |

**Never present an unsourced inference as a bare assertion.**

**Never use "intuition" to describe LLM reasoning.** Use "pattern-match," "aggregate likelihood," or "low-confidence extrapolation."

## Tagging Format

Close substantive messages with a blockquote tag line:

```
> #topic1 #topic2 #topic3
```

- Tags go at the end, after the message body
- Use descriptive, searchable terms — not generic labels
- Match vocabulary to how someone would query `history_search`
- Skip tags on ephemeral replies (brief acknowledgments, one-liners)

**Obsidian outputs:** When producing Obsidian markdown files, tags belong in the YAML frontmatter `tags:` list, NOT in a blockquote tag line. The blockquote format above is for chat/conversation history tagging only. Obsidian's search and Dataview index frontmatter tags reliably; blockquote tags are not indexed consistently. Sanitize tags: no spaces (use hyphens), cannot start with a number, lowercase.

## Common Mistakes

| Mistake | Fix |
|---|---|
| Inference stated as bare fact | Add `_(~inferred: basis)_` |
| "My intuition says..." | Replace with "pattern-match suggests..." or "low-confidence extrapolation:" |
| Tagging every single message | Skip tags on ephemeral / one-line replies |
| Vague tags like `#misc` or `#general` | Use specific, queryable terms |
| URL linked without verifying it resolves | Verify or note _(link unverified)_ |
