# Ketchup Documentation Overlap Audit

## Overview
Three files define ketchup's output standards: `SKILL.md` (main orchestration), `notebook-format.md` (notebook-specific), and `cite-and-tag.md` (citation/tagging reference). Significant content duplication exists across citation rules, tagging conventions, and output format specifications.

## Overlapping Content

### 1. Citation Rules (HIGH PRIORITY)
**Appears in:**
- `cite-and-tag.md` lines 6–17 (table format with 5 claim types)
- `SKILL.md` lines 223–229 (embedded in Pass 1 agent prompt)
- `notebook-format.md` lines 190–194 (embedded in annotation agent prompt)

**Issue:** Citation rules are defined once but embedded twice more verbatim in agent prompts. Changes to rules require updates in three places.

**Duplication severity:** Rules and rationale appear identically in two prompts; cite-and-tag.md owns the reference.

---

### 2. Tagging Rules (HIGH PRIORITY)
**Appears in:**
- `cite-and-tag.md` lines 19–34 (Obsidian frontmatter + rules)
- `SKILL.md` lines 295–299 (frontmatter tags brief mention)
- `SKILL.md` line 443 (common mistake: "Tags outside YAML frontmatter")

**Issue:** Obsidian tagging is comprehensive in cite-and-tag.md, but SKILL.md doesn't reference it, only giving a passing mention in its template. Notebook variant mentioned only in notebook-format.md lines 34–38.

**Duplication severity:** Guidelines scattered; cite-and-tag.md is authoritative but not fully cited by SKILL.md.

---

### 3. Callout Syntax & Conversion (MODERATE)
**Appears in:**
- `SKILL.md` lines 295–296 (output uses callout syntax: `> [!info]`, `> [!tip]`, `> [!warning]`)
- `notebook-format.md` lines 79–98 (complete Obsidian → HTML conversion rules)

**Issue:** SKILL.md specifies that Obsidian output uses callout syntax, but doesn't explain conversion. notebook-format.md owns conversion rules. Users reading SKILL.md won't find the "why" for the Obsidian syntax choice.

**Duplication severity:** Low — different purposes (spec vs. implementation), but linked content should cross-reference.

---

### 4. Common Mistakes (LOW)
**Appears in:**
- `SKILL.md` lines 431–457 (14-item table, broad architecture scope)
- `cite-and-tag.md` lines 40–48 (5-item table, citation/tagging specific)

**Issue:** Both tables include citation/tagging mistakes (e.g., "Inference stated as bare fact", "Tags outside YAML frontmatter"). cite-and-tag.md items are a subset of architectural mistakes in SKILL.md.

**Duplication severity:** Low — both serve different audiences (orchestration vs. reference), but table is small enough to merge or deduplicate.

---

### 5. Wikilinks vs. Standard Markdown (LOW)
**Appears in:**
- `SKILL.md` line 279 (rule: wikilinks for obsidian, markdown links for notebook)
- `notebook-format.md` line 64 (action: wikilinks strip brackets)

**Issue:** SKILL.md states the rule, notebook-format.md states the conversion action. notebook-format.md alone doesn't explain the rationale.

**Duplication severity:** Low — complementary content, not truly duplicated.

---

## Consolidation Recommendations

| Content | Current Owner | Issue | Action |
|---|---|---|---|
| Citation rules (all 5 types + rationale) | cite-and-tag.md | Embedded in 2+ prompts | cite-and-tag.md stays canonical; SKILL.md & notebook-format.md should reference it, not re-specify |
| Tagging rules (Obsidian frontmatter) | cite-and-tag.md | Not referenced by SKILL.md, scattered | cite-and-tag.md stays canonical; SKILL.md line 443 should link to cite-and-tag.md |
| Tagging rules (notebook metadata) | notebook-format.md | Variant not in cite-and-tag.md | Merge into cite-and-tag.md as "## Tagging (Format-Specific)" subsection |
| Callout conversion (Obsidian → HTML) | notebook-format.md | SKILL.md mentions callouts but doesn't explain why | No action — complementary content. Add a line in notebook-format.md linking to SKILL.md's callout choice |
| Common mistakes (citation-specific) | cite-and-tag.md | Duplicated in SKILL.md's broader list | Keep both; SKILL.md's list is broader (architecture), cite-and-tag.md's is focused reference |
| Wikilinks rule | SKILL.md (rule) + notebook-format.md (action) | Split across files | Move rule to cite-and-tag.md; notebook-format.md references it |

---

## Proposed File Ownership

### cite-and-tag.md (Canonical Reference)
**Owner:** All output formatting conventions (citations, tagging, link styles)

**Add:**
- Current: Citation Rules, Tagging (Obsidian), Common Mistakes
- Add: Tagging (Notebook metadata variant)
- Add: Wikilinks vs. Markdown rule (moved from SKILL.md line 279)

**Sections after consolidation:**
```
# Cite and Tag Reference

## Citation Rules
[current content]

## Tagging
### Obsidian Frontmatter
[current content]

### Notebook Metadata
[move from notebook-format.md lines 34-38]

## Link Styles
### Obsidian: Wikilinks
[from SKILL.md line 279]

### Notebook: Markdown Links
[from SKILL.md line 279]

## Common Mistakes
[current content]
```

### SKILL.md (Main Orchestration)
**Owner:** Research pipeline, agent dispatch, session state

**Modify:**
- Line 223–229 (Pass 1 citation rules): Replace with inline reference: "Apply citation rules from cite-and-tag.md"
- Line 279 (wikilinks rule): Remove; add reference to cite-and-tag.md instead
- Line 295–296 (callout syntax): Change to: "Use Obsidian callout syntax (see notebook-format.md § Callout Conversion for notebook translation)"
- Line 443 (Common mistake): Update link to cite-and-tag.md for full reference

**Result:** ~20 lines shorter; citation rules delegated; clearer cross-file navigation.

### notebook-format.md (Format-Specific)
**Owner:** Notebook `.ipynb` structure, annotation system, callout conversion

**Modify:**
- Line 190–194 (annotation agent citation rules): Replace with: "Apply citation rules from cite-and-tag.md"
- Line 34–38 (notebook metadata tags): Remove; consolidated into cite-and-tag.md
- Line 64 (wikilinks): Add note linking to cite-and-tag.md § Link Styles
- Line 79–98 (Callout Conversion): Keep as-is; this is notebook-specific implementation

---

## Migration Checklist

1. ✓ Create "## Tagging (Notebook Metadata)" section in cite-and-tag.md
2. ✓ Move wikilinks rule from SKILL.md to cite-and-tag.md as "## Link Styles"
3. ✓ Replace citation rule embeds in SKILL.md and notebook-format.md with references
4. ✓ Remove duplicate tagging metadata description from notebook-format.md
5. ✓ Add cross-reference links (SKILL.md → cite-and-tag.md, notebook-format.md → cite-and-tag.md)
6. ✓ Verify no content loss during consolidation

---

## Files Affected & Line Impact

| File | Lines Before | Lines After | Net Change | Reason |
|---|---|---|---|---|
| cite-and-tag.md | 49 | ~75 | +26 | Add notebook tagging, link styles, wikilinks rule |
| SKILL.md | 457 | ~430 | -27 | Remove embedded citation rules, wikilinks rule, replace with references |
| notebook-format.md | 257 | ~225 | -32 | Remove citation rules embed, remove notebook tagging duplicate, add references |
| **Total** | **763** | **730** | **-33** | ~4% reduction; improved cross-referencing |

The consolidation reduces total content by ~33 lines while improving clarity and maintainability by establishing cite-and-tag.md as the authoritative reference for all output formatting rules.
