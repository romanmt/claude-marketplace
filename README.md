# Claude Marketplace

A curated collection of Claude Code plugins.

## Available Plugins

| Plugin | Description | Version |
|--------|-------------|---------|
| [openclaw-designer](plugins/openclaw-designer) | OpenClaw configuration plugin for Claude Code | 0.1.0 |

## Installation

To install a plugin from this marketplace, point Claude Code at the plugin's directory within this repository:

```bash
claude plugin add /path/to/claude-marketplace/plugins/<plugin-name>
```

For example:

```bash
claude plugin add /path/to/claude-marketplace/plugins/openclaw-designer
```

## Adding a Plugin

1. Create a new directory under `plugins/` with your plugin name
2. Include a `.claude-plugin/plugin.json` manifest inside it
3. Add your skills, agents, and other plugin resources
4. Add an entry to `marketplace.json`
5. Update the table in this README
