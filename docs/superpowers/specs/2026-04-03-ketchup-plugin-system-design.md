# Ketchup Skill v2 + Plugin System — Design Spec

## Overview

Improvements to the ketchup skill based on 5-agent review, plus three new features: `--plugin` (opt-in MCP data sources), `.ketchup` config files (persistent defaults), and `--time` (knowledge staleness modifier).

## Arguments (Complete)

| Flag | Type | Storable in `.ketchup` | Purpose |
|------|------|------------------------|---------|
| `--occ` | Quoted string | Yes | Reader's occupation/role |
| `--tgt` | Quoted string | No (per-invocation) | Topic to research |
| `--plugin` | Comma-separated string | Yes | MCP data sources for prefetch |
| `--time` | Integer (years) | Yes | Knowledge staleness — years since last deep engagement |
| `--registry` | Subcommand | N/A | Manage plugin registry (list/add/remove) |

## Plugin System

### Design Principles

- Plugins are **strictly opt-in** — no prefetch without explicit `--plugin` or `.ketchup` config
- Context7 is not special — it's a registry entry like any other
- Plugins operate at the **prefetch layer only** — orchestrator queries MCP tools before dispatching subagents, injects results into subagent prompts
- Multiple plugins run sequentially per invocation

### Plugin Registry

**File locations:**
- `~/.claude/skills/ketchup/plugin-registry.yaml` — ships with skill, global defaults
- `<project>/.ketchup-plugins.yaml` — project-level additions/overrides, merged on top

**Format:**

```yaml
plugins:
  context7:
    match: "context7"
    description: "Current documentation for libraries, frameworks, and SDKs"
    workflow:
      - tool: "mcp__plugin_context7_context7__resolve-library-id"
        action: "Resolve topic to library ID"
      - tool: "mcp__plugin_context7_context7__query-docs"
        action: "Query per-facet with focused strings"

  microsoft-docs:
    match: "microsoft"
    description: "Official Microsoft and Azure documentation"
    workflow:
      - tool: "mcp__plugin_microsoft-docs_microsoft-learn__microsoft_docs_search"
        action: "Search topic"
      - tool: "mcp__plugin_microsoft-docs_microsoft-learn__microsoft_docs_fetch"
        action: "Fetch top results for depth"
      - tool: "mcp__plugin_microsoft-docs_microsoft-learn__microsoft_code_sample_search"
        action: "Search code samples if topic involves code"
```

### Plugin Resolution Order

1. Normalize input (lowercase, trim whitespace): `"Context7"` → `context7`
2. Check registry for substring match against `match` field (project-level first, then global)
3. If no registry match → **discovery fallback**: scan available MCP tools for names containing the normalized string, read MCP server instructions from system prompt, construct best-effort workflow. Warn user.
4. If discovery finds nothing → warn and skip this plugin

### Registry Management

| Command | Action |
|---------|--------|
| `--registry list` | Show all registered plugins (global + project) with source |
| `--registry add <name>` | Interactive: prompt for match pattern, description, workflow steps. Writes to project-level by default; `--global` flag writes to skill-level |
| `--registry remove <name>` | Remove from project-level (or `--global` for skill-level) |

## `.ketchup` Config File

**Hierarchy:** `~/.ketchup` (global) < `<project>/.ketchup` (project override)

Project-level values **replace** global values per-key (not deep-merged).

```yaml
# ~/.ketchup
occ: "Windows Systems Engineer"
time: 6
plugins:
  - context7

# <project>/.ketchup (overrides per-key)
occ: "Cloud Infrastructure Engineer"
plugins:
  - context7
  - microsoft-docs
```

**Resolution order:**
1. CLI flags (always win)
2. Project `.ketchup` (check cwd first, then walk up to git root; first match wins)
3. Global `~/.ketchup`
4. No defaults — prompt user

**Rules:**
- `--plugin` on CLI replaces config `plugins` (not appends)
- `--time 0` explicitly means "current practitioner" (overrides config)
- Omitting `--time` entirely (no CLI, no config) = assume current knowledge
- `--tgt` is never stored — always per-invocation
- Users edit `.ketchup` files directly (no `--save` command)

## `--time` — Knowledge Staleness Modifier

Integer representing years since the reader's last deep engagement with the field.

**Effect on outputs:**
- **Anchor vocabulary to the era** — Bridge from tools/patterns current N years ago
- **Explicit change markers** — "You may remember X — since then, Y replaced it"
- **Don't assume recent knowledge** — Anything from within the gap window needs introduction
- **Calibrate density** — 2 years stale = light catch-up; 10 years stale = foundational re-orientation

**Subagent prompt addition:**
```
KNOWLEDGE STALENESS: The reader's last deep technical engagement was {time} years ago.
Bridge from tools/patterns current in {current_year - time}. Explicitly flag what
has changed since then. Do not assume familiarity with anything that emerged in the
last {time} years.
```

**Report frontmatter:** Adds `staleness: N` field.

## Pipeline Changes (Step 1.5)

The hardcoded Context7 step becomes a generic plugin iterator:

```
Step 1.5: Plugin pre-fetch (if any plugins active)

For each active plugin:
  1. Look up plugin in registry (project-level first, then global)
  2. If found: execute workflow steps in order, collecting results
  3. If not found: attempt convention-based discovery
  4. If discovery fails: warn and skip

Attach results to relevant subagent prompts as:

  PREFETCHED DOCUMENTATION ({plugin_name}):
  {results}

  Use this as an authoritative source for factual claims. Cite as ({plugin_name}).
```

- Each plugin's results tagged with plugin name for citation
- Orchestrator decides per-facet which plugin results to attach to which subagent
- If ALL plugins fail, pipeline continues — subagents use web search only

## Review-Identified Fixes (Also Included)

These are from the 5-agent review and are integrated into the implementation plan:

1. Complete argument parsing spec (unquoted strings, typos, neither-flag case)
2. Mermaid flowchart replacing dot (renders in Obsidian)
3. Missing flowchart nodes (prior --occ? neither flag?)
4. Session state documentation
5. Expanded perspective lenses (risk tolerance, misconception correction)
6. Subagent failure handling (validation pass before synthesis)
7. Citation rule consolidation (eliminate 3-copy DRY violation)
8. Obsidian tags in frontmatter (not blockquote)
9. Output destination (ask user)
10. Visualization decision table
11. Facet decomposition menu
12. Synthesis contradiction checking
13. Subagent output budget cap (600-800 words)
14. Context7 section rewritten as plugin documentation
