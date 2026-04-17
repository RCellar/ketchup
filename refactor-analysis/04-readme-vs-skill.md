# README vs SKILL.md: Content Audit

## Summary

README.md mixes user-facing installation/examples with internal implementation details. SKILL.md is comprehensive but duplicates README's examples and quick-start material. Both can be tightened by pushing implementation rules into SKILL.md and keeping README focused on getting users started.

---

## Content Belonging Only in README (User-Facing)

**Install section (lines 21–35)**
- Marketplace add/install commands, `/reload-plugins`, skill namespace syntax
- Users need this to get ketchup running
- SKILL.md should not repeat this

**Quick Start (lines 37–57)**
- Concise one-liner examples showing immediate value
- The "Windows sysadmin learning SELinux" scenario
- Keep this in README; SKILL.md examples can be more exhaustive

**Jupyter Notebook Output user tutorial (lines 72–109)**
- How to invoke `--fmt notebook` and `--kernel` flags
- Annotation workflow with `%%ketchup` cells
- This is a user feature guide, belongs in README only

**File Structure / Project Navigation (lines 300–317)**
- Directories, file roles, where to find skill files
- Helps users orient themselves; not needed in SKILL.md

**License (line 319–322)**
- Legal only; irrelevant to SKILL.md

---

## Content Belonging Only in SKILL.md (Claude-Facing)

**Parsing Rules (lines 24–41)**
- How to handle quoted values, unquoted values, single-dash aliases, repeated flags
- Validation logic: `--time` non-negative integer, `--tgt` non-empty, `--fmt` enum check
- When `--kernel` requires `--fmt notebook`, when `--annotate` is standalone
- This is implementation detail; users don't need to know Claude's parsing strategy

**Session Perspective deep dive (lines 116–142)**
- Five shaping lenses: vocabulary bridging, assumed knowledge, practical framing, risk calibration, misconception correction
- Knowledge staleness anchor dates (e.g., 2020-era for `--time 6`)
- **Remove from README's lines 151–171** ("Perspective Shaping") and consolidate here

**Research Pipeline steps 1–5 (lines 144–380)**
- Scope & decompose (which lenses to use, when to ask user to narrow)
- Plugin pre-fetch with discovery-first tool resolution
- Dispatch table: haiku for fact extraction, sonnet for shaping, opus for complex facets
- Step-by-step subagent prompts and validation rules
- Move the abbreviated README version (lines 111–124) into SKILL.md context, keep only 2–3 sentence overview in README

**Cost Discipline table (README lines 125–137)**
- Model selection rationale (haiku cheap, opus reserved)
- This is an internal budget constraint, not user documentation
- **Move entirely to SKILL.md** (it's already there, better to remove from README)

**Confidence Scoring (README lines 139–149)**
- Citation-density thresholds (>70% sourced → high, 40–70% → medium, <40% → low)
- Verification agent logic
- **Move entirely to SKILL.md** (already present in Step 5)

**Cite-and-Tag enforcement (README lines 273–280)**
- Multi-stage validation: subagent prompts, self-reported counts, orchestrator checks, bare-assertion audit
- How the skill guarantees citations (not just recommends them)
- Keep reference in README, move enforcement detail to SKILL.md

**Plugin Registry technical detail (lines 382–423)**
- Hint-based tool resolution and cross-platform portability
- File locations and resolution order (project vs user vs shipped)
- Substring matching for `match` field
- `--registry` subcommand prompting and MCP tool discovery
- **Remove or simplify from README** (lines 200–206); expand in SKILL.md only

**Common Mistakes checklist (lines 431–457)**
- Writing for experts when `--occ` implies beginner
- Raw subagent output without synthesis
- Ignoring `--time` in shaping
- Skipping dispatch table
- These are Claude implementation gotchas, not user documentation

---

## Duplicated Content (Needs Consolidation)

**Flags / Arguments**

| Location | Content | Issue |
|----------|---------|-------|
| README lines 59–70 | Flag table (7 flags, basic examples) | Abbreviated; good for user overview |
| SKILL.md lines 10–22 | Flag table (all flags with type, purpose, example) | Complete; needed for Claude |

**Recommendation:** Keep README's version (concise, user-oriented). Keep SKILL.md's version (exhaustive). No overlap problem, but README's version should link to SKILL.md for exhaustive docs.

**Config**

| Location | Content | Issue |
|----------|---------|-------|
| README lines 208–236 | User guide: how to create `~/.ketchup` and `<project>/.ketchup` with YAML examples | Good for users |
| SKILL.md lines 43–81 | Detailed config spec: resolution order, per-key replacement rules, defaults | Good for Claude |

**Recommendation:** Keep both. README is user-focused; SKILL.md is implementation-focused. No excision needed.

**Examples**

| Location | Content | Issue |
|----------|---------|-------|
| README lines 37–57 | 3 quick-start examples | Concise, ideal for users discovering the tool |
| SKILL.md lines 459–497 | 6 comprehensive examples covering all feature areas | Exhaustive, good for Claude reference |

**Recommendation:** Keep README's version (user-focused). Keep SKILL.md's version (comprehensive). SKILL.md examples are slightly repetitive but serve as a specification checklist for implementation.

**Perspective Shaping & Knowledge Staleness**

| Location | Content | Issue |
|----------|---------|-------|
| README lines 151–171 | Informal overview of 5 shaping lenses and staleness concept | Too detailed for users; belongs in SKILL.md |
| SKILL.md lines 116–142 | Formal specification of shaping lenses and staleness implementation | Correct placement |

**Recommendation:** **Remove from README entirely** (lines 151–171). Users don't need implementation detail; the Quick Start examples suffice. Reference SKILL.md section if needed.

---

## Specific Excisions

### README.md

1. **Remove "How It Works" (lines 111–162)**
   - Move the 1–2 sentence concept to the intro paragraph
   - Replace with: "Ketchup decomposes your topic into facets, researches each in parallel, and synthesizes a report shaped for your occupation and expertise level."
   - Remove all internal detail (Steps 1–7, Cost Discipline, Confidence Scoring)

2. **Remove "Perspective Shaping" section (lines 151–171)**
   - This is implementation detail; SKILL.md owns this
   - Users only need to know it happens, not how

3. **Simplify "Plugins" section (lines 173–198)**
   - Keep the table of built-in plugins
   - Move detailed registry management to footnote or link to SKILL.md
   - Remove lines 200–206 (file locations, plugin registry spec)

4. **Trim "Config" section (lines 208–236)**
   - Keep the user examples and resolution order (lines 210–229)
   - Remove lines 231–236 (detailed override rules) — move to SKILL.md if not already there

5. **Abbreviate "Cite and Tag" (lines 237–272)**
   - Keep: the 5 claim-type markers (lines 245–250) and tagging rules (lines 258–271)
   - Remove: enforcement detail (lines 273–280) — move to SKILL.md verification section

### SKILL.md

1. **Conditional: If Examples (lines 459–497) duplicate README's Quick Start**
   - Not a problem; SKILL.md examples are more exhaustive and serve as a specification
   - Keep them; they're useful as a checklist during implementation

2. **No removals needed** — SKILL.md is already appropriately detailed for Claude

---

## Rationale

**README should be:**
- Concise, example-driven, user-focused
- Install → Quick Start → Flags → Notebook feature → Plugins setup → Config
- Implementation detail distracts from "how do I use this?"

**SKILL.md should be:**
- Comprehensive, rule-driven, implementation-focused
- Every parsing rule, validation check, and agent prompt explicit
- Serves as the single source of truth for Claude to implement the skill correctly

**Outcome:**
- Users get a faster onboarding path (README ~200 lines)
- Claude gets complete specification without hunting across both files (SKILL.md ~500 lines)
- Both files are internally consistent with no conflicting guidance
