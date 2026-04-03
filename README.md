<p align="center">
  <img src="assets/banner.svg" alt="ketchup — Perspective-Shaped Technical Research" width="700"/>
</p>

<p align="center">
  <strong>Catch up on anything. From where you stand.</strong><br/>
  A Claude Code skill that generates in-depth technical research reports<br/>
  shaped for your professional background.
</p>

<p align="center">
  <a href="#install">Install</a> &middot;
  <a href="#quick-start">Quick Start</a> &middot;
  <a href="#flags">Flags</a> &middot;
  <a href="#plugins">Plugins</a> &middot;
  <a href="#config">Config</a>
</p>

---

## Install

```bash
claude install-skill github:dmickles/ketchup
```

## Quick Start

**Set your perspective, then research anything:**

```bash
# Tell ketchup who you are
/ketchup --occ "Windows Systems Engineer"

# Research a topic — shaped for you
/ketchup --tgt "SELinux Administration"
```

That's it. Ketchup decomposes the topic into facets, dispatches parallel research agents, and synthesizes an Obsidian-ready report that explains SELinux through the lens of Windows security concepts you already know — GPO, integrity levels, ACLs, Event Viewer, and more.

**One-liner with everything:**

```bash
/ketchup --occ "Windows Systems Engineer" --tgt "SELinux Administration" --plugin "Context7" --time 6
```

This tells ketchup: *"I'm a Windows sysadmin who hasn't been deep in the weeds for 6 years. Use Context7 docs. Catch me up on SELinux."*

## Flags

| Flag | What it does | Example |
|------|-------------|---------|
| `--occ` | Your occupation — shapes vocabulary, analogies, assumed knowledge | `--occ "DBA"` |
| `--tgt` | Topic to research | `--tgt "Kubernetes"` |
| `--plugin` | MCP data sources for pre-fetch (comma-separated) | `--plugin "Context7"` |
| `--time` | Years since your last deep engagement | `--time 6` |
| `--registry` | Manage plugins: `list`, `add`, `remove` | `--registry list` |

## How It Works

```
/ketchup --occ "Frontend Dev" --tgt "PostgreSQL" --plugin "Context7"
```

1. **Scope & decompose** — Breaks the topic into 2-6 facets tuned to what matters for a frontend dev learning Postgres (not a DBA learning Postgres — different facets entirely)
2. **Plugin pre-fetch** — Orchestrator queries Context7 for current Postgres docs, tags results per facet
3. **Parallel research** — One Sonnet agent per facet, each with your occupation context, staleness window, and pre-fetched docs
4. **Validate & synthesize** — Citation audit, contradiction check, deduplicate, restore your perspective across all sections
5. **Obsidian output** — Formatted report with frontmatter, callouts, Mermaid diagrams, and proper tags

## Perspective Shaping

Ketchup doesn't just add analogies. It restructures content through five lenses:

| Lens | What it does |
|------|-------------|
| **Vocabulary bridging** | Maps concepts to equivalents you know |
| **Assumed knowledge** | Skips what your role implies you know |
| **Practical framing** | Leads with why this matters to *you* |
| **Risk calibration** | Matches warning intensity to your operational stakes |
| **Misconception correction** | Preempts wrong mental models your background creates |

### Knowledge Staleness (`--time`)

Haven't been in the weeds for a while?

```bash
/ketchup --occ "Java Developer" --tgt "Modern JavaScript" --time 8
```

Ketchup bridges from 2018-era JS (ES6 was still new, Webpack was king, class components everywhere) to today. Every major shift gets flagged: *"You may remember X — since then, Y replaced it."*

## Plugins

Plugins are opt-in MCP data sources that run at the pre-fetch layer. The orchestrator queries them *before* dispatching research agents, injecting authoritative docs into each subagent's context.

**Built-in plugins:**

| Plugin | Match | What it fetches |
|--------|-------|----------------|
| Context7 | `context7` | Current library/framework/SDK documentation |
| Microsoft Docs | `microsoft` | Official Microsoft and Azure documentation + code samples |

**Manage plugins:**

```bash
# List all registered plugins
/ketchup --registry list

# Add a new plugin (interactive — prompts for MCP tool workflow)
/ketchup --registry add "My Custom Docs"

# Add to the global registry instead of project-level
/ketchup --registry add "My Custom Docs" --global

# Remove a plugin
/ketchup --registry remove "My Custom Docs"
```

**Plugin registry files:**

| File | Scope |
|------|-------|
| `skills/ketchup/plugin-registry.yaml` | Ships with ketchup (global defaults) |
| `<project>/.ketchup-plugins.yaml` | Project-level additions/overrides |

## Config

Persist your defaults in a `.ketchup` YAML file so you don't have to type flags every time.

```yaml
# ~/.ketchup (global defaults)
occ: "Windows Systems Engineer"
time: 6
plugins:
  - context7
```

```yaml
# <project>/.ketchup (project override — replaces per-key, not deep-merged)
occ: "Cloud Infrastructure Engineer"
plugins:
  - context7
  - microsoft-docs
```

**Resolution order:** CLI flags > project `.ketchup` > global `~/.ketchup` > prompt user

| Override | How |
|----------|-----|
| Use a different occupation for one report | `--occ "New Role"` on the CLI |
| Disable staleness for one report | `--time 0` |
| Use different plugins for one report | `--plugin "Other"` (replaces config list) |

## Citation Standards

Every ketchup report follows strict citation rules:

| Claim type | Format |
|-----------|--------|
| Fact with URL | Inline hyperlink: `claim ([source](url))` |
| Fact, no URL | `_(unsourced)_` |
| Inference | `_(~inferred: brief basis)_` |
| Unverified link | `_(link unverified)_` |

No bare assertions. No "intuition." Every claim is sourced, flagged, or explicitly marked as inference.

## Output Format

Reports are Obsidian-flavored markdown with:

- YAML frontmatter (`topic`, `perspective`, `date`, `confidence`, `source_count`, `tags`)
- `> [!abstract]`, `> [!tip]`, `> [!warning]` callouts
- Mermaid diagrams where helpful
- Tags in frontmatter (not blockquotes) for proper Obsidian indexing

## File Structure

```
ketchup/
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata
├── skills/
│   └── ketchup/
│       ├── SKILL.md          # Main skill definition
│       ├── cite-and-tag.md   # Citation rules reference
│       └── plugin-registry.yaml  # MCP plugin registry
├── docs/
│   └── superpowers/
│       ├── specs/            # Design specs
│       └── plans/            # Implementation plans
├── assets/
│   └── banner.svg
├── LICENSE
└── README.md
```

## License

MIT
