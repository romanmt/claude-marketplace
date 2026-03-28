# OpenClaw Official Docs: System Prompt & Workspace Files

Source: https://docs.openclaw.ai/concepts/system-prompt

---

## System Prompt Construction

OpenClaw constructs a custom system prompt for each agent execution. The framework assembles and injects this prompt into every run.

### Fixed Sections

- **Tooling**: Available tools with brief descriptions
- **Safety**: Guardrails discouraging power-seeking or oversight circumvention
- **Skills**: Instructions for loading skill files on demand
- **Self-Update**: Procedures for `config.apply` and `update.run`
- **Workspace**: Working directory reference
- **Documentation**: Local docs path and usage guidance
- **Workspace Files**: Bootstrap file indicators
- **Sandbox**: Runtime sandbox details (when enabled)
- **Current Date & Time**: User timezone and format
- **Reply Tags**: Provider-specific syntax options
- **Heartbeats**: Heartbeat prompts and acknowledgment behavior
- **Runtime**: Host, OS, Node version, model, repo root, thinking level
- **Reasoning**: Current visibility and toggle hints

### Prompt Modes

**Full** (default): Complete prompt with all sections.

**Minimal** (sub-agents): Excludes Skills, Memory Recall, Self-Update, Model Aliases, User Identity, Reply Tags, Messaging, Silent Replies, and Heartbeats. Retains Tooling, Safety, Workspace, Sandbox, Date/Time, Runtime, and context.

**None**: Returns only base identity information.

---

## Bootstrap File Injection

These workspace files inject automatically on every turn:

- `AGENTS.md`
- `SOUL.md`
- `TOOLS.md`
- `IDENTITY.md`
- `USER.md`
- `HEARTBEAT.md`
- `BOOTSTRAP.md` (new workspaces only)
- `MEMORY.md` or lowercase `memory.md`

### Configuration Controls

- `agents.defaults.bootstrapMaxChars` — default: 20,000 per file
- `agents.defaults.bootstrapTotalMaxChars` — default: 150,000 total
- `agents.defaults.bootstrapPromptTruncationWarning` — `off`, `once`, `always`

### Sub-Agent Bootstrap

Sub-agents receive only `AGENTS.md` and `TOOLS.md` (not SOUL, IDENTITY, USER, MEMORY, or HEARTBEAT).
