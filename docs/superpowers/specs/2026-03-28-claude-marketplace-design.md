# Claude Marketplace Design

## Overview

Transform the existing `openclaw-designer-plugin` single-plugin project into `claude-marketplace` — a monorepo-style registry/catalog where multiple Claude Code plugins live side by side, each fully self-contained and installable directly.

## Directory Structure

```
claude-marketplace/
├── marketplace.json              # Machine-readable catalog of all plugins
├── README.md                     # Human-readable browsable index with plugin table
├── plugins/
│   └── openclaw-designer/        # First plugin (existing, relocated)
│       ├── .claude-plugin/
│       │   └── plugin.json
│       ├── agents/
│       │   └── openclaw-agent-designer.md
│       └── skills/
│           └── openclaw-architect/
│               ├── SKILL.md
│               └── references/
│                   ├── file-quick-reference.md
│                   ├── official-docs-agent-loop.md
│                   ├── official-docs-automation.md
│                   ├── official-docs-memory.md
│                   ├── official-docs-multi-agent.md
│                   ├── official-docs-subagents.md
│                   └── official-docs-system-prompt.md
```

## Marketplace Manifest (`marketplace.json`)

```json
{
  "name": "claude-marketplace",
  "version": "1.0.0",
  "description": "A curated collection of Claude Code plugins",
  "plugins": [
    {
      "name": "openclaw-designer",
      "version": "0.1.0",
      "description": "OpenClaw configuration plugin for Claude Code",
      "path": "plugins/openclaw-designer"
    }
  ]
}
```

Each plugin entry contains:
- `name` — plugin identifier
- `version` — semver version matching the plugin's own `plugin.json`
- `description` — short description
- `path` — relative path from repo root to the plugin directory

Each plugin directory contains a standard `.claude-plugin/plugin.json` so Claude Code can install directly by pointing at the plugin path within the repository.

## README

A top-level `README.md` with:
- Marketplace title and description
- Table of all plugins (name, description, version, link to directory)
- Brief installation instructions

## Design Decisions

- **Plugins live under `plugins/`** — clean separation from marketplace-level files
- **Each plugin is fully self-contained** — own `plugin.json`, agents, skills, references; no cross-dependencies
- **No shared resources yet** — structure doesn't preclude adding a `shared/` directory later
- **Flat listing** — no categories or tags; just name, version, description, path
- **Both JSON and README indexes** — JSON for machine consumption/installation, README for human browsing

## Migration

The existing project files move into `plugins/openclaw-designer/`:
- `.claude-plugin/` -> `plugins/openclaw-designer/.claude-plugin/`
- `agents/` -> `plugins/openclaw-designer/agents/`
- `skills/` -> `plugins/openclaw-designer/skills/`

Files that stay at the marketplace root:
- `docs/` — marketplace-level specs and documentation, not part of any individual plugin

New files created at repo root:
- `marketplace.json`
- `README.md`
