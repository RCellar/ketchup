# CLI Parsing Rules Audit

## Current State
The ketchup skill defines 16 numbered Parsing Rules (lines 24–41). Analysis reveals substantial duplication and collapsible patterns that obscure the core schema.

## Duplicates & Collapsible Rules

### Value Extraction (Rules 1–2)
**Duplicate pattern in different words:**
- Rule 1: Quoted values → `"Windows Systems Engineer"` extracts content between quotes
- Rule 2: Unquoted values → Consume from flag to next `--` or EOF

**Collapse into:** Single rule describing both modes as orthogonal extraction strategies. Flag order (Rule 6) and alias handling (Rule 3) are independent concerns.

### Validation (Rules 8–9)
**Parallel structure, different constraints:**
- Rule 8: `--time` type validation (non-negative integer, 0–50 range)
- Rule 9: `--tgt` type validation (non-empty string, ~30-word limit warning)

**Collapse into:** Single "validation schema" rule defining per-flag constraints and error messages.

### Enum Validation (Rules 12–13)
**Identical structure, different enums:**
- Rule 12: `--fmt` ∈ {obsidian, notebook}
- Rule 13: `--kernel` ∈ {python, bash} AND requires `--fmt notebook`

**Collapse into:** Single "enum flag" rule covering: allowed values, error on invalid, cross-flag dependencies.

### Storage Semantics (Rules 15–16)
**Opposite behaviors, not inverse:** 
- Rule 15: `--fmt` never stored (per-invocation only)
- Rule 16: `--kernel` may be stored in config

**Collapse into:** Single "persistence model" rule defining which flags persist and which are per-invocation.

### Config & Replacement (Rules 5, 4, 10)
**Intertwined logic:**
- Rule 5: Check `.ketchup` config if no CLI flag; prompt if no config
- Rule 4: Last value wins; confirm update
- Rule 10: CLI `--plugin` replaces config `plugins`; confirm with before/after

**Collapse into:** Single rule for config resolution order (CLI → config → prompt) with replacement confirmation protocol.

## Default Behaviors (Not Rules)

- **Rule 3 (single-dash aliases):** Standard CLI parsing. Remove as explicit rule; implement as parser detail.
- **Rule 6 (flag order irrelevant):** Automatic consequence of hash-based flag storage. Document in implementation, not parsing rules.

## Essential Distinctions Retained

1. **Standalone mode** (Rule 14): `--annotate` rejects combination with `--tgt`, `--occ`, `--fmt`, `--kernel`. Non-negotiable; core behavior.
2. **Reset semantics** (Rule 11): Clears session state, not full restart. Distinct from Rule 5 prompt behavior.
3. **`--plugin` normalization** (Rule 7): Substring → normalized (lowercase, hyphens). Distinct from general validation.

## Proposed 5–8 Rule Replacement

1. **Value Extraction** — Quoted values extract from `"..."`. Unquoted values consume to next flag or EOI. Parse both uniformly; flag order is irrelevant post-parse.

2. **Flag Aliases** — Single-dash forms (`-occ`, `-tgt`, etc.) accepted; normalized internally to double-dash equivalents.

3. **Validation Schema** — `--time`: non-negative integer (0–50 typical); `--tgt`: non-empty string, warn if >~30 words; `--plugin`: comma-separated, normalize to lowercase. Invalid values trigger specific error messages per flag.

4. **Enum Flags & Dependencies** — `--fmt` ∈ {obsidian, notebook}; `--kernel` ∈ {python, bash} AND requires `--fmt notebook`. Invalid values rejected with specific error; dependency violation trapped before execution.

5. **Config Resolution & Replacement** — CLI flags override config files; config overrides defaults. If no CLI flag and no config, prompt user. Replacing a prior/config value confirmed with old → new visualization (e.g., "Perspective updated to: {new}").

6. **Persistence Model** — Per-invocation flags (`--fmt`, `--tgt`, `--annotate`) never persist. Config-storable flags (`--occ`, `--time`, `--plugin`, `--kernel`) stored in `.ketchup`; CLI overrides config. Resolution order explicit in config section (already well-documented).

7. **Standalone Mode** — `--annotate` is standalone. Rejects combination with `--tgt`, `--occ`, `--fmt`, `--kernel`; inherits perspective from notebook metadata. Error on violation.

8. **Reset Behavior** — `--reset` clears session state (`--occ`, `--time`, `--kernel`, active plugins) without restart. Confirm: "Session state cleared. Set `--occ` to begin."

## Correctness Preserved

- Validation and error messages remain identical or more precise
- Config fallback logic unchanged (just de-duplicated verbally)
- Dependency enforcement (`--kernel` requires `--fmt notebook`) explicit in Rule 4
- Standalone mode semantics (Rule 7) distinct from general parsing (Rules 1–3)
- Reset vs. fallback (Rule 8 vs. Rule 5) unambiguous

## Reduction Summary

**From 16 to 8 rules** by:
- Merging value extraction (1+2) → Rule 1
- Merging validation (8+9) → Rule 3
- Merging enum validation (12+13) → Rule 4
- Merging storage (15+16) → Rule 6
- Merging config/replacement (5+4+10) → Rule 5
- Removing default behaviors (Rules 3, 6) → Implementation detail
- Preserving distinct concerns (Rules 11, 14) → Rules 7–8
- Keeping aliases, dependencies, fallback logic

**Net result:** Tighter schema, clearer precedence, easier to implement and test.
