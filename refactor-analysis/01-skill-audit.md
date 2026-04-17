# Ketchup Skill Audit

## Executive Summary

The ketchup skill is comprehensive (497 lines, ~7,000 words) but suffers from **structural bloat and distributed duplication** across sections. The Research Pipeline dominates (238 lines, 48% of document) and could be a standalone guide. Parsing rules contain over-specified trivialities, and plugin/format handling is scattered across four sections. Consolidation would improve navigation and maintainability while reducing redundancy.

## Section Breakdown

### 1. Arguments + Parsing Rules (lines 10–42, ~900 words)
**Structure:** CLI flag table + 16 parsing/validation rules + examples.

**Issues:**
- **Over-specification:** Rules 1–6 describe trivial parser behavior (unquoted values, quoted values, flag order, etc.). Any competent implementation would handle these automatically — they waste tokens and don't inform agent behavior.
- **Scattered validation:** Rules 8–16 are edge-case validations that could be compressed into a "Validation Checklist" (5 bullets instead of 9 rules).
- **Ambiguity:** Rule 10 (CLI plugin replacement warning) contradicts rule 7 (split on commas, trim whitespace) — unclear which takes precedence when both `--plugin` and config exist.

**Recommendation:** 
- Collapse rules 1–6 into one bullet: "Parse flags using standard conventions."
- Extract rules 8–16 into a "Validation Checklist" subsection.
- Clarify plugin merging behavior before rule 10.
- **Result:** ~300 words saved, scope sharpened from "how to parse" to "what to validate."

---

### 2. .ketchup Config File (lines 43–114, ~1000 words)
**Structure:** File hierarchy, resolution order, behavior flowchart (Mermaid), rules, examples.

**Issues:**
- **Flowchart is redundant:** The Mermaid diagram (lines 84–103) restates the resolution logic from lines 68–72. The flowchart adds visual flair but no new information.
- **Over-detailed rules:** Rules 2–6 (e.g., "Omitting `--time` entirely = assume current knowledge") restate resolution order. Mentioned twice: once in rule prose, once in resolution flowchart.
- **Scattered plugin behavior:** Plugin merging rules appear here (lines 75) AND in Arguments (rule 10) AND in Research Pipeline Step 1.5 (lines 169–187). Three authoritative sources.

**Recommendation:**
- Replace flowchart with 3–4 bullet points: "CLI wins, then project config, then global, then prompt user."
- Merge plugin rules into Arguments section (single authority).
- Move file locations table to a "Configuration Files" section shared with Plugin Registry.
- **Result:** ~200 words saved, one source of truth for each behavior.

---

### 3. Session Perspective (lines 116–142, ~700 words)
**Structure:** Occupation-based shaping (5 principles), knowledge staleness (3 principles), session state (5 bullets).

**Status:** Well-structured, no major duplication. Subsections are appropriately scoped.

**Minor issue:** "Session State" (lines 137–142) could be moved to Arguments or a dedicated "State Management" section since it overlaps with config resolution concepts.

---

### 4. Research Pipeline (lines 144–381, ~2700 words — 48% of document)
**Structure:** 5 sequential steps (Scope, Plugin Prefetch, Two-Pass Research, Format, Verification), each with detailed sub-instructions.

**Critical issue:** **This section is too large to navigate as a single unit.** It contains:
- Step 1: Decomposition logic + facet selection lens table (60 lines).
- Step 1.5: Plugin prefetch algorithm with discovery-first resolution (19 lines).
- Step 2: Two-pass agent dispatch with full Haiku/Sonnet prompts (100 lines).
- Step 3: Validation, citation audit, synthesis checklist (80 lines).
- Step 4: Obsidian output formatting + report template (100 lines).
- Step 5: Standalone verification agent prompt (40 lines).

**Issues:**
- **Interleaved agent prompts:** Step 2 embeds full boilerplate prompts for fact extraction and perspective shaping. These are long blocks (~30 lines each) that make the section hard to scan for logic.
- **Scope creep:** Notebook output mentioned in Step 4 (delegating to `notebook-format.md`), but notebook-specific rules scattered in Arguments and Common Mistakes.
- **Verification as afterthought:** Step 5 is a full verification agent, but it's buried at the end rather than shown upfront as a quality gate.
- **Duplication with Plugin Registry:** Plugin discovery logic (lines 175–186) repeats concepts from Plugin Registry section (lines 396–399).

**Recommendation:**
- **Extract to separate "Research Pipeline Guide"** (sub-skill or linked document). Keep only the 2–3 sentence overview + link here.
- **Move agent prompts to templates:** Keep prompt structure in main skill; move full boilerplate to a `prompts/` subdirectory.
- **Consolidate notebook output:** Collect all `--fmt` rules (Arguments, Step 4, Common Mistakes) into one "Output Formats" section.
- **Front-load verification:** Mention "verification is a separate quality gate" in the overview, not buried at step 5.
- **Result:** Main skill becomes ~500 words of navigation; reduces cognitive load by 50%.

---

### 5. Plugin Registry (lines 383–423, ~600 words)
**Structure:** File locations table, match semantics, hint-based resolution, format spec, subcommands.

**Issues:**
- **Overlaps with Step 1.5:** Discovery-first resolution (lines 175–186) and hint-based tool resolution (lines 175–186 again, and lines 399–400) are explained in two places with slightly different vocabulary.
- **Format spec is dense:** The YAML example is clear, but the surrounding prose (match field semantics, hint field explanation) could be compressed.
- **Subcommand details are verbose:** `--registry` subcommands are documented in a 3-row table, but their implementation details would benefit from a separate "Registry Management" sub-skill if this feature grows.

**Recommendation:**
- **Merge file locations** into a "Configuration Files" subsection (with Config File paths).
- **Create a "Plugin System" sub-section** that briefly covers discovery-first resolution (reference Step 1.5 for detail).
- **Keep the YAML format spec** — it's clear and necessary.
- **Result:** ~150 words condensed; better integration with Arguments + Config sections.

---

### 6. Common Mistakes (lines 431–457, ~600 words)
**Structure:** 27 mistakes in a single lookup table.

**Issues:**
- **Scattered across phases:** Mistakes span argument parsing (5), research pipeline (12), output formatting (5), and conceptual misunderstandings (5), but no section headings to navigate by phase.
- **Mix of enforcement and guidance:** Some are mandatory rules ("Don't use MCP tools in subagent prompts"), others are recommendations ("Don't pad narrow topics with extra facets").
- **Duplication with earlier sections:** "Using sonnet/opus for fact extraction" (line 448) restates the dispatch table from Step 2 (lines 195–200). "Ignoring `--time` in subagent prompts" restates Session Perspective rules.

**Recommendation:**
- **Split into 4 subsections:** Argument Parsing, Research Execution, Output & Formatting, Conceptual Pitfalls.
- **Remove duplicates:** Cross-reference earlier sections instead of restating rules.
- **Demote recommendations to tips:** Mark "Don't pad narrow topics" as a style suggestion, not a blocker.
- **Result:** ~200 words saved; table becomes a quick-reference by phase.

---

### 7. Cite-and-Tag (lines 425–429, ~100 words)
**Status:** Healthy. Brief reference to external file. No changes needed.

---

### 8. Examples (lines 459–497, ~400 words)
**Status:** Well-chosen and clear. No structural issues.

---

## Consolidation Opportunities

| Duplication | Locations | Merged Into |
|-------------|-----------|------------|
| Plugin behavior (merging, discovery, config override) | Arguments, Config, Step 1.5, Registry, Mistakes | Plugin section (single authority) |
| `--fmt` / `--kernel` rules | Arguments, Step 4, Mistakes | Output Formats section |
| Facet decomposition logic | Step 1 + Step 3 synthesis | Research Pipeline or sub-skill |
| Citation rules | Step 2, cite-and-tag.md, Mistakes | cite-and-tag.md (single authority) |

---

## Recommended Refactoring

1. **Extract Research Pipeline** to a linked "Pipeline Guide" sub-document (480 lines → 50 lines main skill).
2. **Consolidate plugin behavior** into a "Plugin System" subsection.
3. **Create "Output Formats" section** covering `--fmt`, `--kernel`, and notebook-specific rules.
4. **Compress parsing rules** (rules 1–9) into a validation checklist.
5. **Reorganize Common Mistakes** by phase with cross-references instead of duplication.
6. **Result:** Main skill becomes ~2000 words (navigable in 10 minutes) with clear structure: CLI → Config → Pipeline → Plugins → Output → Troubleshooting.
