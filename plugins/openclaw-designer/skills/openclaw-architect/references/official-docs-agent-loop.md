# OpenClaw Official Docs: Agent Loop, Context Engine, Plugins, Security

Sources:
- https://docs.openclaw.ai/concepts/agent-loop
- https://docs.openclaw.ai/concepts/context-engine
- https://docs.openclaw.ai/tools/plugin
- https://docs.openclaw.ai/cli/security
- https://docs.openclaw.ai/concepts/oauth
- https://docs.openclaw.ai/nodes

---

## Agent Loop

Complete execution cycle: intake → context assembly → model inference → tool execution → streaming replies → persistence.

### Entry Points

- Gateway RPC: `agent` and `agent.wait` commands
- CLI: `agent` command

### Workflow

1. **RPC Validation** — validates params, resolves session, returns `{ runId, acceptedAt }`
2. **Agent Command** — resolves model, loads skills, calls Pi Agent runtime, emits lifecycle events
3. **Pi Agent Processing** — serializes runs per session key, resolves auth, enforces timeouts, returns usage metadata
4. **Event Bridging** — converts Pi events to OpenClaw streams (tool events, deltas, lifecycle phases)
5. **Completion** — `agent.wait` returns `{ status: ok|error|timeout, startedAt, endedAt, error? }`

### Concurrency

Runs serialized per session key (session lane), optionally through a global lane.

### Timeouts

- Agent runtime default: 172,800 seconds (48 hours)
- `agent.wait` default: 30 seconds

---

## Context Engine

Controls how OpenClaw builds model context for each run.

### Lifecycle Points

1. **Ingest** — processes new messages
2. **Assemble** — constructs ordered messages within token budget
3. **Compact** — summarizes older history when context fills
4. **After turn** — post-run operations (state persistence)

### Configuration

```json
plugins.slots.contextEngine: "engine-id"
```

`ownsCompaction: true` = engine fully manages compaction. `false` = Pi's built-in auto-compaction supplements.

---

## Plugin System

Expand capabilities across channels, model providers, tools, skills, speech, image generation.

### Two Formats

- **Native plugins** — `openclaw.plugin.json` + runtime module (in-process)
- **Bundle plugins** — Codex/Claude/Cursor-compatible layouts

### Registration Methods

- `registerProvider` — model providers
- `registerChannel` — chat channels
- `registerTool` — agent tools
- `registerHook` / `on()` — lifecycle hooks
- `registerSpeechProvider` — TTS/STT
- `registerContextEngine` — context engines
- `registerService` — background services
- Plus: media understanding, image generation, web search, HTTP routes, CLI commands

### Discovery Precedence

1. Config-specified paths (`plugins.load.paths`)
2. Workspace extensions (`.openclaw/extensions/`)
3. Global extensions (`~/.openclaw/extensions/`)
4. Bundled plugins

### Enablement

- Master toggle: `plugins.enabled`
- Allowlist/denylist: `plugins.allow` / `plugins.deny` (deny wins)
- Per-plugin: `plugins.entries.<id>.enabled`
- Workspace-origin plugins default to disabled

---

## Nodes (Remote Execution)

A node is a companion device (macOS/iOS/Android/headless) connecting to the Gateway WebSocket with `role: "node"`.

### Architecture

- **Gateway host**: receives messages, runs model, routes tool calls
- **Node host**: executes `system.run`/`system.which` remotely
- **Approvals**: enforced via `~/.openclaw/exec-approvals.json` on the node host

Nodes ≠ gateways. Nodes are peripherals. Messages from Telegram/WhatsApp arrive at gateway, not nodes.

### Node Commands

- Canvas operations (snapshot, navigate, eval)
- Camera (photo, video ≤60s)
- Screen recording
- Location (off by default)
- Android-specific (SMS, contacts, calendar, notifications)
- System (shell execution, notifications, approvals)

### Execution Binding

```
openclaw config set tools.exec.node "node-id-or-name"
```

---

## Security

### `openclaw security audit`

Identifies risks at configurable depth. Read-only by default.

Key flags:
- Multi-user shared access without `session.dmScope="per-channel-peer"`
- Small models (≤300B) without sandboxing
- Webhook token reuse
- Docker `host` network mode
- Unpinned plugin installations
- `gateway.auth.mode="none"`

`--fix` applies safe remediations (policy adjustments only, no credential rotation).

---

## OAuth / Auth

- Credentials stored per-agent at `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
- Multiple accounts: separate agents (isolated) or multiple profiles (same agent)
- Automatic token refresh via stored `expires` timestamp
- Supports: Anthropic API key, setup-token, local CLI login reuse, OpenAI OAuth/PKCE
