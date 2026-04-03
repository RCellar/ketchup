# Notebook Format — Ketchup Companion

Notebook generation and annotation logic for ketchup. Referenced by SKILL.md when `--fmt notebook` or `--annotate` is active.

## Notebook Generation (`--fmt notebook`)

### Input

The synthesized draft from Step 3 of the main pipeline — a single continuous markdown document with YAML frontmatter.

### Output

A `.ipynb` file (nbformat v4.5) with the following structure:

**Notebook-level metadata:**

~~~json
{
  "nbformat": 4,
  "nbformat_minor": 5,
  "metadata": {
    "kernelspec": {
      "name": "<python3|bash>",
      "display_name": "<Python 3|Bash>"
    },
    "language_info": {
      "name": "<python|bash>"
    },
    "ketchup": {
      "version": "1.1.1",
      "topic": "<from frontmatter>",
      "perspective": "<from frontmatter>",
      "staleness": "<from frontmatter, omit if not set>",
      "plugins": ["<from frontmatter, omit if none>"],
      "confidence": "<from frontmatter>",
      "source_count": "<from frontmatter>",
      "date": "<from frontmatter>",
      "tags": ["<from frontmatter>"]
    }
  },
  "cells": []
}
~~~

For `--kernel bash`: `kernelspec.name` = `"bash"`, `display_name` = `"Bash"`, `language_info.name` = `"bash"`.
For `--kernel python` (default): `kernelspec.name` = `"python3"`, `display_name` = `"Python 3"`, `language_info.name` = `"python"`.

### Translation Rules

Process the synthesized draft top-to-bottom:

| Source (Obsidian markdown) | Target (notebook) | Cell tags |
|---|---|---|
| YAML frontmatter | `metadata.ketchup` object — not a cell | n/a |
| `# Heading` (h1 title) | Markdown cell | `ketchup:generated`, `ketchup:section` |
| `## Heading` section boundary | New markdown cell (split here) | `ketchup:generated`, `ketchup:section` |
| Prose within a section | Part of that section's markdown cell | (inherits section tags) |
| Fenced code block — language matches `--kernel` | Code cell (extracted from prose) | `ketchup:generated`, `ketchup:code-example` |
| Fenced code block — language does NOT match `--kernel` | Stays in markdown cell as fenced block | (inherits section tags) |
| `> [!tip] Title` | HTML alert div (see Callout Conversion) | (inherits section tags) |
| `> [!warning] Title` | HTML alert div | (inherits section tags) |
| `> [!info] Title` | HTML alert div | (inherits section tags) |
| `> [!abstract] TL;DR` | HTML alert div | `ketchup:generated`, `ketchup:tldr` |
| Wikilinks `[[text]]` | Plain text — strip brackets | n/a |
| Mermaid fenced blocks | Preserved as-is in markdown cell | (inherits section tags) |
| GFM tables | Preserved as-is in markdown cell | (inherits section tags) |

### Section Splitting

Split at `## ` boundaries (h2 headings). Each h2 section becomes its own markdown cell. The h1 title + TL;DR become the first cell(s).

Within a section, when a fenced code block's language matches `--kernel`:
1. End the current markdown cell at the text just before the code block
2. Insert a code cell containing the code block's content (without the fence markers)
3. Start a new markdown cell for any text after the code block
4. If this creates an empty markdown cell (code block was at the end of a section), omit the empty cell

### Callout Conversion

Obsidian callout syntax does not render in Jupyter. Convert:

| Obsidian | HTML class | Label |
|---|---|---|
| `> [!tip]` | `alert alert-success` | Tip |
| `> [!warning]` | `alert alert-warning` | Warning |
| `> [!info]` | `alert alert-info` | Info |
| `> [!abstract]` | `alert alert-primary` | TL;DR |
| `> [!caution]` | `alert alert-danger` | Caution |

Template:
~~~html
<div class="alert alert-{class}">
<strong>{Label}:</strong> {title}
<p>{content}</p>
</div>
~~~

Multi-line callout content: each line after the `> ` prefix (with the `[!type]` line removed) becomes paragraph content. Preserve markdown formatting within the `<p>` tags.

### Cell Requirements

- **Cell IDs:** Every cell requires a unique `id` field (nbformat v4.5). Generate 8-character lowercase alphanumeric strings, unique within the notebook.
- **Tags:** Stored in `cell.metadata.tags` as an array of strings. Every ketchup-generated cell gets `ketchup:generated`.
- **Code cells:** Set `execution_count: null` and `outputs: []` (not yet executed).
- **Source format:** The `source` field is written as a **multiline string** (not an array of lines). The nbformat v4 schema defines `source` as `{"oneOf": [string, array-of-strings]}` — Jupyter accepts both, but a single string is simpler to generate and what Jupyter itself produces when saving. When *reading* notebooks (e.g., `--annotate`), always handle both formats: if `source` is an array, join it into a single string before processing.

### Output Destination

Same prompt as Obsidian output: ask the user where to save. File extension must be `.ipynb`. If the user provides a path ending in `.md`, replace with `.ipynb` and confirm: "Notebook saved as {path}.ipynb (notebooks use .ipynb extension)."

## Annotation System (`--annotate`)

### Entry Point

`/ketchup --annotate ./path/to/report.ipynb`

This is a standalone mode — the research pipeline does not run. The annotation system reads the notebook, finds queries, and answers them in-place.

### Step 1: Load and Inherit Context

Read the `.ipynb` file. Parse `metadata.ketchup`:

| Field | Maps to | Required |
|---|---|---|
| `perspective` | Session `--occ` | Yes |
| `staleness` | Session `--time` | No — omit if absent |
| `plugins` | Active plugin list for prefetch | No — skip prefetch if absent |
| `topic` | Original report topic (context) | Yes |

If `perspective` or `topic` is missing: "This notebook has no ketchup metadata. Use `--annotate` on notebooks generated by ketchup."

**Kernel detection:** Read `metadata.kernelspec.name` to determine the kernel for code cell splitting in response translation. If `kernelspec.name` is `python3` or `python`, use `python`. If `bash`, use `bash`. If the value is absent, unrecognized (e.g., `julia`, `ir`, `R`), or missing entirely, default to `python` and warn: "Unrecognized kernel '{name}' — defaulting to python for code cell splitting. Non-python/bash code blocks will be preserved in markdown cells."

Confirm loaded context: "Loaded from notebook: perspective={perspective} | staleness={staleness}yr | plugins={plugins} | topic={topic}"

### Step 2: Scan for Query Cells

Walk all cells in order. A query cell is any cell whose `source` starts with `%%ketchup\n`. Everything after that first line is the query text. The cell type (markdown or code) does not matter — only the `%%ketchup` prefix. If `source` is an array (older nbformat convention), join into a single string before checking.

Skip cells already tagged `ketchup:query-answered` in their `cell.metadata.tags`.

If no query cells found: "No annotation queries found. Add a markdown cell starting with `%%ketchup` followed by your question."

If queries found, confirm: "Found {N} annotation queries. Processing..."

### Step 3: Plugin Prefetch (conditional)

If `plugins` was inherited from notebook metadata and is non-empty:
- Run prefetch using the same mechanism as the main pipeline's Step 1.5
- Scope queries to the annotation text (not the original facets)
- Build per-query focused query strings from the `%%ketchup` text

If no plugins in metadata, skip this step.

### Step 4: Dispatch Annotation Agents

Consult the dispatch table in SKILL.md. For each query, dispatch a **sonnet** agent. Parallelize all queries.

Agent prompt:

~~~
You are an annotation agent. Your task:

ORIGINAL REPORT TOPIC: {topic}
READER PERSPECTIVE: {perspective}
{If staleness set:}
KNOWLEDGE STALENESS: The reader's last deep technical engagement was {staleness} years ago.
Bridge from tools/patterns current in {current_year - staleness}. Explicitly flag what
has changed since then.

QUERY: {query_text}

SURROUNDING CONTEXT (2 cells before and after the query):
--- BEFORE ---
{cell[i-2].source}
{cell[i-1].source}
--- AFTER ---
{cell[i+1].source}
{cell[i+2].source}

{If plugin docs prefetched:}
PREFETCHED DOCUMENTATION ({plugin_name}):
{results}
Use this as an authoritative source. Cite as ({plugin_name}).

Respond to the query shaped for this reader's perspective. Follow the same
vocabulary bridging, staleness anchoring, and risk calibration used in the
original report.

CITATION RULES (mandatory):
- Verifiable fact with URL → hyperlink inline: "claim ([source](url))"
- Fact, no URL available → append _(unsourced)_
- Inference/probability → append _(~inferred: brief basis)_
- Never present unsourced inference as bare assertion

OUTPUT: Markdown prose, 200-600 words depending on query scope.
For section expansions, use ## or ### headings as appropriate.
For code examples, use fenced code blocks with language tags.
~~~

### Step 5: Insert Responses and Update Annotation Index

Process query cells in **reverse index order** (highest index first) to preserve cell positions during insertion.

Maintain a running annotation counter `N` starting from the highest existing annotation number in the notebook (scan for `ketchup:annotation-marker` cells and extract their number), or 1 if none exist.

For each query cell at index `i`:

1. **Replace the query cell with a marker:** Replace the `%%ketchup` cell's source with a compact reference:
   ```
   *[Annotation #N](#ketchup-annotation-index) — "{first 60 characters of query}..."*
   ```
   Set the marker cell's tags to `ketchup:query-answered` and `ketchup:annotation-marker`. Store the full original query text in `cell.metadata.ketchup.original_query` and the annotation number in `cell.metadata.ketchup.annotation_number`.

2. **Translate the response:** Determine the kernel from the notebook's `metadata.kernelspec.name`. Apply the same translation rules as notebook generation:
   - Prose → markdown cell(s)
   - Code blocks matching kernel → code cells
   - Callouts → HTML alert divs

3. **Insert after the marker cell:** Insert the translated cell(s) at index `i + 1`. Tag each with `ketchup:response` and `ketchup:generated`. Store `annotation_number` in each response cell's `cell.metadata.ketchup.annotation_number`.

4. **Generate cell IDs:** Each inserted cell gets a unique 8-character alphanumeric `id`, checked against all existing IDs in the notebook.

5. **Increment** `N` for the next query.

**After all queries are processed, update the Annotation Index:**

Scan the notebook for an existing cell tagged `ketchup:annotation-index`. If found, replace its source. If not found, append a new markdown cell at the end of the notebook.

The annotation index cell contains:

```markdown
## Annotation Index
<a id="ketchup-annotation-index"></a>

| # | Original Query | Date |
|---|---------------|------|
| 1 | First 80 chars of query... | 2026-04-03 |
| 2 | First 80 chars of query... | 2026-04-03 |
```

Tag the index cell with `ketchup:generated` and `ketchup:annotation-index`.

Build the table by scanning all `ketchup:annotation-marker` cells in the notebook (not just the ones processed in this run — include previously answered annotations too). Read the full query from `cell.metadata.ketchup.original_query` and the number from `cell.metadata.ketchup.annotation_number`.

### Step 6: Write and Confirm

Write the modified notebook back to the same file path. Confirm: "Annotated {N} queries in {path}. Annotation index updated."

### Idempotency

- `ketchup:query-answered` cells (annotation markers) are skipped on subsequent `--annotate` runs
- To re-ask a question: replace the marker cell's source with a new `%%ketchup` query and remove `ketchup:query-answered` from its tags. The previous response cells remain in place below it.
- Running `--annotate` on a notebook with no unanswered queries is a no-op with a message
- The annotation index is rebuilt from all marker cells on every run, so it stays in sync even if markers are manually edited
- **Anchor fragility:** Marker cells link to `#ketchup-annotation-index`. If a user deletes the index cell manually, these links will be broken until the next `--annotate` run recreates it. This is self-healing — any `--annotate` invocation (even one that finds no new queries) rebuilds the index from existing markers.
