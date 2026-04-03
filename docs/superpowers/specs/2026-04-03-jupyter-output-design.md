# Jupyter Notebook Output & Annotation System

**Date:** 2026-04-03
**Branch:** `feature/jupyter-output`
**Status:** Design approved, pending implementation

## Overview

Add Jupyter notebook (`.ipynb`) as an output format for ketchup reports, and an annotation system that lets users query the generated notebook for clarifications or expansions, with responses shaped to the original report's perspective.

Two features:
1. **`--fmt notebook`** — translates ketchup's synthesized output into a `.ipynb` file with executable code cells, HTML callouts, and ketchup metadata.
2. **`--annotate <path>`** — reprocesses an existing ketchup notebook, answering `%%ketchup` query cells in-place with perspective-shaped responses.

## Architecture: Companion File (Approach B)

All notebook-specific logic lives in `notebook-format.md`, a companion skill file referenced by SKILL.md. SKILL.md gains routing logic (new flags, flowchart additions) but delegates format-specific behavior to the companion.

**File changes:**
- `skills/ketchup/SKILL.md` — new flags, routing logic, Step 4 fork
- `skills/ketchup/notebook-format.md` — new file, notebook generation and annotation system

## New Flags

| Flag | Type | Purpose | Default | Example |
|------|------|---------|---------|---------|
| `--fmt` | `obsidian` \| `notebook` | Output format selection | `obsidian` | `--fmt notebook` |
| `--kernel` | `python` \| `bash` | Kernelspec for code cells | `python` | `--kernel bash` |
| `--annotate` | File path (`.ipynb`) | Reprocess notebook for `%%ketchup` queries | n/a | `--annotate ./report.ipynb` |

### Parsing Rules

- `--fmt` accepts only `obsidian` or `notebook`. Any other value is rejected: "Invalid `--fmt` value. Use `obsidian` or `notebook`."
- `--kernel` without `--fmt notebook` is rejected: "The `--kernel` flag requires `--fmt notebook`."
- `--kernel` accepts only `python` or `bash`. Any other value is rejected: "Invalid `--kernel` value. Use `python` or `bash`."
- `--annotate` is a standalone mode. Combined with `--tgt` or `--occ`, reject: "The `--annotate` flag is a standalone mode — do not combine with `--tgt` or `--occ`."
- `--annotate` path must end in `.ipynb`. Reject otherwise: "The `--annotate` flag requires a `.ipynb` file path."
- `--fmt` is never stored in `.ketchup` config — always per-invocation (same as `--tgt`).
- `--kernel` may be stored in `.ketchup` config under the `kernel` key.

### Routing

```
--annotate provided?
  yes → Annotation flow (notebook-format.md)
  no  → Normal pipeline
         Step 4: --fmt notebook?
           yes → Notebook generation (notebook-format.md)
           no  → Obsidian output (existing behavior, unchanged)
```

## Notebook Generation (`--fmt notebook`)

### When It Activates

Step 4 of the existing pipeline. Steps 1-3 (decompose, prefetch, two-pass research, synthesis) and Step 5 (verification) are unchanged. The synthesized draft from Step 3 is the input.

### Notebook Structure

**Notebook-level metadata:**

```json
{
  "nbformat": 4,
  "nbformat_minor": 5,
  "metadata": {
    "kernelspec": {
      "name": "python3|bash",
      "display_name": "Python 3|Bash"
    },
    "language_info": {
      "name": "python|bash"
    },
    "ketchup": {
      "version": "1.1.1",
      "topic": "...",
      "perspective": "...",
      "staleness": 6,
      "plugins": ["context7"],
      "confidence": "high",
      "source_count": 42,
      "date": "2026-04-03",
      "tags": ["ketchup", "topic-tag", "occ-tag"]
    }
  },
  "cells": [...]
}
```

For `--kernel bash`: `kernelspec.name` = `"bash"`, `kernelspec.display_name` = `"Bash"`, `language_info.name` = `"bash"`.

### Translation Rules

| Source (Obsidian markdown) | Target (notebook) |
|---|---|
| YAML frontmatter | `metadata.ketchup` object (not a cell) |
| `# Heading` / section prose | Markdown cell, tagged `ketchup:section` |
| Fenced code block (language matches kernel) | Code cell, tagged `ketchup:code-example` |
| Fenced code block (language doesn't match kernel) | Markdown cell with fenced block preserved |
| `> [!tip]`, `> [!warning]`, `> [!info]`, `> [!abstract]` callouts | HTML `<div class="alert alert-{type}">` in markdown cell |
| `> [!abstract]` TL;DR specifically | HTML alert div, additionally tagged `ketchup:tldr` |
| Wikilinks `[[text]]` | Plain text (brackets stripped) |
| Mermaid fenced blocks | Preserved as-is in markdown cell (JupyterLab 4.1+ renders natively) |
| GFM tables | Preserved as-is in markdown cell |

### Cell Tagging

Every ketchup-generated cell receives `ketchup:generated` in `cell.metadata.tags`. Additional tags by role:

| Cell role | Tags |
|---|---|
| Section prose | `ketchup:generated`, `ketchup:section` |
| Code example | `ketchup:generated`, `ketchup:code-example` |
| TL;DR callout | `ketchup:generated`, `ketchup:tldr` |

### Callout Conversion

Obsidian callout syntax does not render in Jupyter. Convert to HTML:

```html
<!-- > [!tip] Title becomes: -->
<div class="alert alert-success">
<strong>Tip:</strong> Title
<p>Content...</p>
</div>

<!-- > [!warning] Title becomes: -->
<div class="alert alert-warning">
<strong>Warning:</strong> Title
<p>Content...</p>
</div>

<!-- > [!info] Title becomes: -->
<div class="alert alert-info">
<strong>Info:</strong> Title
<p>Content...</p>
</div>

<!-- > [!abstract] TL;DR becomes: -->
<div class="alert alert-primary">
<strong>TL;DR</strong>
<p>Content...</p>
</div>
```

### Cell ID Generation

nbformat v4.5 requires a unique `id` on every cell. Generate 8-character alphanumeric IDs (lowercase + digits), unique within the notebook.

### Section Splitting

The synthesized draft is one continuous markdown document. Split into cells at `## ` boundaries (h2 headings). Each h2 section becomes its own markdown cell. The h1 title gets its own cell. Code blocks within a section are extracted into separate code cells positioned after the prose that references them.

## Annotation System (`--annotate`)

### Entry Point

`/ketchup --annotate ./path/to/report.ipynb`

### Step 1: Load and Inherit Context

Read the `.ipynb` file. Extract from `metadata.ketchup`:

| Field | Maps to | Required |
|---|---|---|
| `perspective` | Session `--occ` | Yes — reject if missing |
| `staleness` | Session `--time` | No — omit if absent |
| `plugins` | Active plugin list | No — skip prefetch if absent |
| `topic` | Context for responses | Yes — reject if missing |

If `perspective` or `topic` is missing: "This notebook has no ketchup metadata. Use `--annotate` on notebooks generated by ketchup."

### Step 2: Scan for Query Cells

Walk all cells. A query cell is any cell whose `source` starts with `%%ketchup\n`. Everything after that first line is the query text.

```
%%ketchup
Can you explain how Group Policy inheritance works in a multi-domain forest?
```

Cells already tagged `ketchup:query-answered` are skipped (idempotency).

If no `%%ketchup` cells found: "No annotation queries found. Add a markdown cell starting with `%%ketchup` followed by your question."

### Step 3: Plugin Prefetch

If `plugins` were inherited from notebook metadata, run prefetch (same as main pipeline Step 1.5) scoped to the query text — not the original facets.

If no plugins in metadata, skip.

### Step 4: Dispatch Annotation Agents

For each query, dispatch a **sonnet** agent in parallel:

```
You are an annotation agent. Your task:

ORIGINAL REPORT TOPIC: {topic}
READER PERSPECTIVE: {perspective}
KNOWLEDGE STALENESS: {staleness} years
QUERY: {query_text}

SURROUNDING CONTEXT (2 cells before and after the query):
{context_cells}

{If plugin docs prefetched:}
PREFETCHED DOCUMENTATION ({plugin_name}):
{results}

Respond to the query shaped for this reader's perspective.
Follow the same vocabulary bridging, staleness anchoring, and
risk calibration as the original report.

CITATION RULES: Same as main pipeline — sourced, unsourced, inferred markers.

OUTPUT: Markdown prose, 200-600 words depending on query scope.
For section expansions, use ## or ### headings as appropriate.
```

### Step 5: Insert Responses and Update Annotation Index

Process query cells in **reverse index order** (to preserve positions during insertion). Maintain a running annotation counter starting from the highest existing annotation number in the notebook, or 1 if none exist.

For each query cell:

1. **Replace the query cell with a marker:** Replace the `%%ketchup` cell's source with `*[Annotation #N](#ketchup-annotation-index) — "first 60 chars of query..."*`. Tag with `ketchup:query-answered` and `ketchup:annotation-marker`. Store the full original query in `cell.metadata.ketchup.original_query` and the annotation number in `cell.metadata.ketchup.annotation_number`.
2. Insert response cell(s) immediately after the marker:
   - Tag with `ketchup:response`, `ketchup:generated`
   - Store `annotation_number` in `cell.metadata.ketchup.annotation_number`
   - If response contains code blocks matching the notebook kernel, split into code cells (same translation rules as generation)
3. Generate unique cell IDs for all inserted cells

**After all queries:** Append or update an **Annotation Index** cell (tagged `ketchup:annotation-index`) at the end of the notebook containing a table of all annotations (number, original query excerpt, date). The index is rebuilt from all `ketchup:annotation-marker` cells on every run.

### Step 6: Write and Confirm

Write modified `.ipynb` back to the same path. Confirm: "Annotated {N} queries in {path}. Annotation index updated."

### Idempotency

- Annotation marker cells (tagged `ketchup:query-answered`) are skipped on re-runs
- To re-ask a question: replace the marker cell's source with a new `%%ketchup` query and remove `ketchup:query-answered` from its tags
- Previous response cells remain in place below the marker
- The annotation index is rebuilt from all marker cells on every run, keeping it in sync even after manual edits

## What Does NOT Change

- Steps 1-3 of the research pipeline (decompose, prefetch, two-pass research, synthesis)
- Step 5 verification (runs on the synthesized draft before format conversion)
- Obsidian output (default, unchanged behavior)
- `.ketchup` config format (except optional `kernel` key)
- Plugin registry system
- cite-and-tag citation rules (applied in both formats)
- Session perspective mechanics

## Edge Cases

| Scenario | Behavior |
|---|---|
| `--fmt notebook` with no code in report | Notebook has only markdown cells — valid |
| `--kernel bash` but report has Python examples | Python blocks stay as markdown cells (not executable), bash blocks become code cells |
| `--annotate` on non-ketchup `.ipynb` | Rejected — missing `metadata.ketchup` |
| `--annotate` with zero `%%ketchup` cells | Message shown, no changes written |
| Mixed kernel code in report | Only blocks matching `--kernel` become code cells |
| `%%ketchup` in a code cell (not markdown) | Still detected — cell type doesn't matter, only `%%ketchup` prefix. Response is always inserted as markdown cell(s) regardless of query cell type |
| User deletes `ketchup:query-answered` tag | Query is reprocessed on next `--annotate` run |
| Very long query (500+ words) | Accepted — no length limit on queries |
