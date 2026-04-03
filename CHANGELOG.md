# Changelog

All notable changes to the ketchup skill are documented here.

---

## [1.1.0] - 2026-04-03

### Added
- **Two-pass research pipeline** — Pass 1 uses haiku for cheap, fast fact extraction via web search. Pass 2 uses sonnet to reshape facts for the reader's occupation and staleness window. No re-research in Pass 2 — only perspective shaping on Pass 1 output.
- **Dispatch table** — Authoritative model selection table governs every agent call. Haiku for extraction and verification, sonnet for shaping, opus only for complex cross-domain facets (>3 subtopics).
- **Step 5: Verification phase** — Dedicated haiku agent checks frontmatter completeness, section presence, citation compliance, link resolution, and confidence scoring independently after the report is drafted.
- **Confidence scoring** with defined thresholds — high (>70% sourced), medium (40-70%), low (<40%). Verification agent sets the `confidence` field in frontmatter automatically.
- **`--reset` flag** — Clears session state (occ, time, plugins) mid-conversation without restarting.
- **`--time` input validation** — Rejects negative numbers, decimals, and non-numeric values with a clear error message.
- **Orchestration red flags** in Common Mistakes table — cost discipline anti-patterns: don't use sonnet for extraction, don't skip the dispatch table, don't verify inline, don't default to opus, don't re-research in Pass 2.

### Changed
- **Cite-and-tag.md rewritten** — Removed blockquote tagging (inapplicable to ketchup's Obsidian output). Now covers only citation rules and frontmatter tagging.
- **Config walk algorithm** — Defined termination condition: walks up to git root (if in a repo) or filesystem root (if not). Previously underspecified.
- **Plugin registry `--global` write target** — Now writes to `~/.ketchup-plugins.yaml` (user-owned, survives plugin updates) instead of the shipped `plugin-registry.yaml` (overwritten on update).
- **Plugin `match` field** — Explicitly documented as case-insensitive substring matching.
- **SVG banner** — Replaced fragile double-rect accent line workaround with a proper clipPath.

### Fixed
- **Flowchart missing `--registry` path** — Added early branch node for `--registry` before the `--occ` check.
- **Citation rule drift** — Pass 1 prompt template and cite-and-tag.md now include the same rules (URL verification, training-data-only flagging).
- **Plugin name normalization** — Added space-to-hyphen replacement to the normalization step.
- **Config type conversion** — Documented that config `plugins` (YAML list) and `--plugin` (comma-separated string) resolve to the same internal list.
- **`--registry add` offline fallback** — Users can manually enter tool names when no MCP tools are available.

---

## [1.0.0] - 2026-04-03

### Added
- **Core skill** — `/ketchup` with `--occ` (occupation), `--tgt` (topic), `--plugin` (MCP data sources), `--time` (knowledge staleness), and `--registry` (plugin management).
- **Plugin system** — Opt-in MCP data source prefetch with registry-based resolution, discovery fallback for unregistered plugins, and project-level overrides via `.ketchup-plugins.yaml`.
- **`.ketchup` config files** — Persistent defaults for `--occ`, `--plugin`, and `--time`. Global (`~/.ketchup`) with project-level override. Per-key replacement, not deep-merged.
- **Plugin registry** — `plugin-registry.yaml` ships with Context7 and Microsoft Docs entries. Managed via `--registry list/add/remove`.
- **Cite-and-tag system** — All outputs enforce citation discipline: sourced facts get URLs, unsourced facts get `_(unsourced)_`, inferences get `_(~inferred: basis)_`. No bare assertions.
- **5 perspective lenses** — Vocabulary bridging, assumed knowledge, practical framing, risk & consequence calibration, preemptive misconception correction.
- **Knowledge staleness (`--time`)** — Anchors vocabulary to the reader's era, flags what changed, calibrates explanation density.
- **Obsidian-flavored output** — YAML frontmatter with topic/perspective/date/confidence/source_count/tags, callouts, Mermaid diagrams, frontmatter-only tags.
- **Multi-agent research pipeline** — Parallel subagent dispatch per facet, orchestrator-level plugin prefetch (subagents can't access MCP), 3-phase validation (validate, audit, synthesize).
- **Facet decomposition menu** — 8 standard lenses to select from, with scope checking for narrow/broad topics.
- **Marketplace distribution** — `.claude-plugin/plugin.json` + `marketplace.json` for installation via `/plugin marketplace add RCellar/ketchup`.
- **Banner and README** — Lab Report variant SVG banner, comprehensive documentation with install instructions, flag reference, examples.

---

## Format

Based on [Keep a Changelog](https://keepachangelog.com/).
