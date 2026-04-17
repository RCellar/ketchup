# Ketchup Refactor Plan

## Executive Summary

The Ketchup skill (currently ~7,000 words across 4 files) suffers from **structural bloat, distributed duplication, and over-specified rules**. The Research Pipeline dominates at 48% of SKILL.md; parsing rules describe trivial parser behavior; and citation/tagging rules are embedded identically in multiple agent prompts. The refactor achieves ~15–20% token savings (1,000–1,500 words) by extracting the Pipeline to a linked sub-guide, collapsing redundant parsing rules from 16 → 8, consolidating citation/tagging rules into a single canonical file, and moving implementation detail out of the user-facing README. The result: SKILL.md becomes ~3,500 words (readable in 15 minutes), README becomes ~200 words (clear quickstart path), and all formatting rules live in a single cite-and-tag reference.

---

## File Structure Transformation

### Before (Current State)
```
ketchup/
├── README.md (321 lines; mixes user guide + implementation detail)
├── skills/ketchup/
│   ├── SKILL.md (497 lines; monolithic — 48% is Pipeline, 27% is Plugin Registry + Config)
│   ├── notebook-format.md (257 lines; format-specific, duplicates citation rules)
│   ├── cite-and-tag.md (49 lines; reference, but embedded also in SKILL.md + notebook-format.md)
│   └── plugin-registry.yaml (20 lines)
└── CHANGELOG.md
```

### After (Proposed State)
```
ketchup/
├── README.md (~200 lines; user-focused: Install → Quick Start → Flags → Notebook → Config)
├── skills/ketchup/
│   ├── SKILL.md (~330 lines; core spec: CLI → Config → Session Perspective → Validation → Output → Plugins)
│   ├── SKILL-PIPELINE-GUIDE.md (~400 lines; extracted Research Pipeline deep-dive, linked from main)
│   ├── cite-and-tag.md (~75 lines; canonical reference: Citations + Tagging + Link Styles + Mistakes)
│   ├── notebook-format.md (~225 lines; format-specific: structure, annotation, callout conversion)
│   └── plugin-registry.yaml (20 lines; unchanged)
└── CHANGELOG.md
```

**Benefit:** Each file serves one purpose. SKILL.md is navigable in 15 minutes. cite-and-tag.md is the single source of truth for all output formatting. Pipeline depth available on-demand via linked guide.

---

## Prioritized Changes

### P0: Must-Do (Blocking; major token savings)

1. **Extract Research Pipeline to SKILL-PIPELINE-GUIDE.md**
   - **Current:** SKILL.md lines 144–381 (238 lines, ~3,500 words)
   - **Action:** Move verbatim to new file; replace in SKILL.md with 2–3 sentence overview + link
   - **Savings:** ~2,800 words / ~430 tokens when SKILL.md loaded (Pipeline only loaded if user explicitly requests)
   - **Why:** Pipeline is a self-contained, complex subsystem. Readers don't need it for basic CLI/Config understanding. Lazy-load on demand.

2. **Consolidate Citation Rules into cite-and-tag.md**
   - **Current:** Citation rules defined in cite-and-tag.md (canonical) but embedded verbatim in:
     - SKILL.md lines 223–229 (Pass 1 agent prompt)
     - notebook-format.md lines 190–194 (annotation agent prompt)
   - **Action:** Replace both embeds with single-line reference: "Apply citation rules from cite-and-tag.md § Citation Rules"
   - **Savings:** ~30 lines / ~150 tokens from both files
   - **Why:** Single source of truth prevents divergence. Maintenance is easier. Changes propagate instantly.

3. **Collapse Parsing Rules from 16 → 8**
   - **Current:** SKILL.md lines 24–41 (16 numbered rules; many describe default behaviors)
   - **Action:** Merge per auditor recommendations:
     - Rules 1–2 (value extraction) → single extraction rule
     - Rules 8–9 (validation) → single validation schema
     - Rules 12–13 (enum validation) → single enum rule
     - Rules 15–16 (persistence) → single persistence model
     - Rules 5+4+10 (config resolution) → single resolution rule
     - Remove Rules 3, 6 (default behaviors; document in implementation notes only)
   - **Savings:** ~200 words / ~80 tokens from SKILL.md
   - **Why:** Current rules duplicate each other in different words. Collapsed rules are clearer and easier to validate during implementation.

4. **Reorganize Common Mistakes by Phase**
   - **Current:** SKILL.md lines 431–457 (27 mistakes in flat table; scattered across parsing/pipeline/output phases)
   - **Action:** Split into 4 subsections: Argument Parsing, Research Execution, Output & Formatting, Conceptual Pitfalls; remove duplication with earlier sections
   - **Savings:** ~150 words / ~60 tokens (removing cross-phase duplication)
   - **Why:** Readers can navigate mistakes by the phase they're working in. Duplication is removed.

5. **Consolidate Plugin Behavior into Single Section**
   - **Current:** Plugin behavior scattered across Arguments (rule 10), Config (lines 75), Step 1.5 (lines 169–186), Plugin Registry (lines 399–400), Common Mistakes
   - **Action:** Merge into new "Plugin System" subsection (2–3 paragraphs) covering: discovery-first resolution, config override, confirmation protocol; reference from Arguments only
   - **Savings:** ~100 words / ~40 tokens (removing 3+ duplicate explanations)
   - **Why:** Single source of truth for plugin behavior.

### P1: Should-Do (Improves clarity, moderate savings)

6. **Move README Implementation Detail to SKILL.md**
   - **Current:** README lines 111–171 duplicate SKILL.md's "Perspective Shaping" and "Cost Discipline"
   - **Action:** Remove from README; expand section in SKILL.md (already present)
   - **Savings:** ~200 words / ~80 tokens from README
   - **Why:** Users don't need to understand cost discipline or shaping lenses; implementation detail belongs in SKILL.md

7. **Create "Output Formats" Section in SKILL.md**
   - **Current:** Format rules scattered: Arguments (`--fmt`, `--kernel`), Step 4 (notebook output), Common Mistakes (format validation)
   - **Action:** Consolidate into single "Output Formats" subsection (2–3 paragraphs) covering `--fmt` enum, `--kernel` dependency, persistence rules; reference from Arguments
   - **Savings:** ~100 words / ~40 tokens (removing duplication)
   - **Why:** All format behavior in one place. Easier to update when adding new formats.

8. **Expand cite-and-tag.md to Include Notebook Tagging Variant**
   - **Current:** Obsidian tagging rules in cite-and-tag.md; notebook metadata rules in notebook-format.md lines 34–38 (scattered)
   - **Action:** Add "## Tagging (Notebook Metadata)" subsection to cite-and-tag.md; move from notebook-format.md; update notebook-format.md to reference
   - **Savings:** ~20 lines moved (no net token loss; improved organization)
   - **Why:** All tagging rules (regardless of format) in single reference.

9. **Simplify Config File Section in SKILL.md**
   - **Current:** Lines 84–103 have redundant Mermaid flowchart + prose explanation of resolution order
   - **Action:** Replace flowchart with 3–4 bullets; remove redundant prose
   - **Savings:** ~80 words / ~30 tokens
   - **Why:** Flowchart adds visual flair but no new info. Bullets are faster to read.

### P2: Nice-to-Have (Polish, minimal savings)

10. **Restructure Common Mistakes Table for Scannability**
    - **Current:** 27 mistakes in single table; reader must scan all to find phase-relevant mistakes
    - **Action:** Add phase subheadings; group by context
    - **Savings:** ~20 words / ~10 tokens (only from added structure markup, not content removal)
    - **Why:** Faster reference during implementation.

11. **Add Cross-File Reference Links**
    - **Current:** Files mention related concepts but don't link (e.g., SKILL.md cites rules that live in cite-and-tag.md but doesn't say so)
    - **Action:** Add explicit references at end of relevant sections (e.g., "See cite-and-tag.md § Tagging Rules for variant formats")
    - **Savings:** ~30 words / ~15 tokens (links replace duplicated prose)
    - **Why:** Readers know where to find authoritative info without guessing.

---

## Token-Footprint Reduction

| Change | Tokens Saved | Reason |
|--------|--------------|--------|
| Extract Pipeline (P0.1) | ~430 | Don't load SKILL-PIPELINE-GUIDE.md unless requested |
| Consolidate citations (P0.2) | ~150 | Replace embeds with references |
| Collapse parsing rules (P0.3) | ~80 | Merge 16 → 8 rules |
| Reorganize mistakes (P0.4) | ~60 | Remove phase duplication |
| Consolidate plugins (P0.5) | ~40 | Single source of truth |
| Move README detail (P1.1) | ~80 | Users don't need cost/shaping detail |
| Create Output Formats section (P1.2) | ~40 | Remove scattered `--fmt` rules |
| Expand cite-and-tag.md (P1.3) | ~0 | Moved, not removed (net neutral) |
| Simplify Config (P1.4) | ~30 | Flowchart → bullets |
| **Total Savings** | **~910 tokens** | **~12% reduction in loaded skill** |

**Real-world impact:**
- Current total footprint: ~7,500 words / ~1,150 tokens (SKILL.md + README + supporting docs)
- After refactor: ~6,300 words / ~960 tokens (same feature, lighter load)
- SKILL.md specifically: 7,000 words → 3,500 words (50% reduction) when Pipeline is extracted
- Cache hit improvement: cite-and-tag.md changes no longer require 2+ re-edits in agent prompts

---

## What NOT to Touch (Risks)

### High-Risk Areas (No Changes)
1. **Agent Prompts (inner boilerplate):** Don't rewrite agent prompts (Step 2, Step 5) without testing end-to-end. Changing wording could affect model behavior.
2. **Examples (lines 459–497):** These serve as a specification checklist. Don't remove; instead, ensure they remain valid after other changes.
3. **Plugin Registry YAML (plugin-registry.yaml):** Leave unchanged. External tools may depend on exact format.
4. **Plugin Discovery Logic (Step 1.5):** Don't simplify discovery-first resolution without testing cross-platform tool resolution (Windows, Linux, macOS).

### Medium-Risk Areas (Verify After Refactor)
1. **Parsing Rule Collapse:** After merging 16 → 8 rules, verify no edge case is lost. Test all `--flag` combinations.
2. **Citation Rule Extraction:** After moving to cite-and-tag.md, ensure agent prompts still reference correctly and no rule is missed.
3. **README Simplification:** Remove implementation detail but verify quick-start examples still guide new users to success.

### Verification Checklist
- [ ] All 8 collapsed parsing rules tested (quoted/unquoted values, flag aliases, validation, enums, config, persistence, standalone, reset)
- [ ] agent prompts (Step 2, Step 5) still work after citation rules move
- [ ] README examples still work (test all 3 quickstart scenarios)
- [ ] Cross-file references in SKILL.md, README, notebook-format.md work (no broken links)
- [ ] Pipeline guide (new file) is readable and self-contained
- [ ] cite-and-tag.md is the single canonical source (no other file repeats citation/tagging rules)

---

## Implementation Timeline

**Phase 1 (Low Risk):** P0.1 (Extract Pipeline), P0.2 (Consolidate Citations)
- These are deletions + file moves, low risk of breaking behavior

**Phase 2 (Medium Risk):** P0.3 (Collapse Parsing Rules), P0.4 (Reorganize Mistakes)
- These change rule wording; verify with test suite

**Phase 3 (Low Risk):** P1.1–P1.4, P2 (Polish & References)
- Structural improvements, no behavior changes

---

## Summary

The refactor is **incremental, reversible, and focused on clarity + efficiency**. No features are removed; only organization is tightened. The key moves:

1. **Extract** the 238-line Pipeline to a linked guide (lazy-load)
2. **Consolidate** citation rules from 3 locations → 1 (cite-and-tag.md)
3. **Collapse** 16 parsing rules → 8 (remove defaults, merge duplicates)
4. **Reorganize** Common Mistakes by phase (remove cross-phase duplication)
5. **Simplify** README by moving implementation detail to SKILL.md

**Result:** ~12% token footprint reduction, faster SKILL.md navigation, single sources of truth for plugins/citations/tagging, and clearer README for new users.
