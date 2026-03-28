# OpenClaw Official Docs: Sub-Agents

Source: https://docs.openclaw.ai/tools/subagents

---

## Overview

Sub-agents are background runs spawned from an existing agent session. They operate in isolated sessions (`agent:<agentId>:subagent:<uuid>`) and announce results back to the requester channel upon completion.

## Spawning

Via `/subagents spawn` command or `sessions_spawn` tool (programmatic).

- **Non-blocking**: Returns run ID immediately
- **Announcement**: On completion, sub-agent posts summary to requester channel

### sessions_spawn Parameters

- `task` (required) — the work to perform
- `label?` — human-readable identifier
- `agentId?` — spawn under a different agent if permitted
- `model?` — override sub-agent model
- `thinking?` — override thinking level
- `runTimeoutSeconds?` — abort after N seconds (default 0 = no timeout)
- `thread?` (default `false`) — request channel thread binding
- `mode?` (`run` | `session`, default `run`) — `session` requires `thread: true`
- `cleanup?` (`delete` | `keep`, default `keep`) — immediate archive or retention
- `sandbox?` (`inherit` | `require`, default `inherit`)

### Model & Thinking Inheritance Chain

Explicit parameter > agent-specific config > global default > caller's model/thinking

## Sub-Agent Bootstrap Context

**Sub-agents receive ONLY `AGENTS.md` + `TOOLS.md`.**

Not injected: `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`, `MEMORY.md`

This is critical for workspace design — any information sub-agents need must be in AGENTS.md or TOOLS.md, or passed explicitly in the task prompt.

## Nested Sub-Agents (Orchestrator Pattern)

Default: `maxSpawnDepth: 1` (sub-agents cannot spawn children).

Set `maxSpawnDepth: 2` to enable orchestrator pattern (main → orchestrator → workers).

### Depth Levels

| Depth | Session Key | Role | Can Spawn? |
|-------|------------|------|-----------|
| 0 | `agent:<id>:main` | Main agent | Always |
| 1 | `agent:<id>:subagent:<uuid>` | Orchestrator (if depth ≥ 2) | Only if maxSpawnDepth ≥ 2 |
| 2 | `agent:<id>:subagent:<uuid>:subagent:<uuid>` | Leaf worker | Never |

Maximum nesting: depth 5 (range 1–5; depth 2 recommended).

### Announce Chain

- Depth-2 worker → announces to parent orchestrator
- Orchestrator synthesizes → announces to main
- Main delivers to user
- Each level only sees announces from direct children

### Configuration

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2,
        maxChildrenPerAgent: 5,    // max active children per session (range 1-20)
        maxConcurrent: 8,          // global concurrency cap
        runTimeoutSeconds: 900,    // default timeout
      },
    },
  },
}
```

## Tool Access by Depth

**Default: sub-agents get all tools EXCEPT session and system tools.**

Tools DENIED by default:
- `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`
- System tools (`gateway`, etc.)

**Exception:** Depth-1 orchestrators (when maxSpawnDepth ≥ 2) additionally receive:
- `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history`

### Override

```json5
{
  tools: {
    subagents: {
      tools: {
        deny: ["gateway", "cron"],
        // allow: ["read", "exec", "process"]  // allow-only mode; deny still wins
      },
    },
  },
}
```

## Cross-Agent Spawning

- `agentId` parameter targets a different registered agent
- Controlled by `agents.list[].subagents.allowAgents` (default: only requester agent)
- Set `["*"]` to allow any agent
- Sandbox guard: sandboxed requesters cannot spawn unsandboxed targets

## Authentication

- Auth resolved by agent ID, not session type
- Main agent's auth profiles merge as fallback; agent-specific profiles override on conflict
- Fully isolated per-agent auth not yet supported

## Concurrency & Lifecycle

- Lane: `subagent` (dedicated queue)
- Concurrency: `maxConcurrent` default 8
- Auto-archive: after `archiveAfterMinutes` (default 60)
- `cleanup: "delete"` archives immediately after announce
- `/stop` cascades to all spawned sub-agents
- `/subagents kill <id>` stops specific + children
- `/subagents kill all` stops all + cascades

## Thread-Bound Sessions (Discord)

Discord is currently the only supported channel for persistent thread-bound sub-agent sessions.

Commands:
- `/focus <target>` — bind thread to sub-agent
- `/unfocus` — remove binding
- `/agents` — list active runs with binding state
- `/session idle <duration|off>` — auto-unfocus inactivity window
- `/session max-age <duration|off>` — hard cap on session age

## Announce Behavior

- If sub-agent replies `ANNOUNCE_SKIP`, nothing is posted
- Completion context includes: result, status, runtime, token stats, session key
- Announce payload meant for orchestration; user-facing replies should be rewritten in normal voice
- Best-effort; pending announces lost on gateway restart

## Slash Commands

- `/subagents list` — view runs
- `/subagents kill <id|#|all>` — terminate
- `/subagents log <id|#> [limit] [tools]` — inspect logs
- `/subagents info <id|#>` — metadata
- `/subagents send <id|#> <message>` — message sub-agent
- `/subagents steer <id|#> <message>` — direct behavior
- `/subagents spawn <agentId> <task> [--model <model>] [--thinking <level>]` — launch
