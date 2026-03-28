# OpenClaw Official Docs: Automation (Cron, Heartbeats, Hooks, Standing Orders)

Sources:
- https://docs.openclaw.ai/automation/cron-jobs
- https://docs.openclaw.ai/automation/cron-vs-heartbeat
- https://docs.openclaw.ai/automation/hooks
- https://docs.openclaw.ai/automation/standing-orders

---

## Cron Jobs

Built-in Gateway scheduler. Jobs persist at `~/.openclaw/cron/` and survive restarts.

### Two Execution Styles

- **Main session**: enqueue system events, run on next heartbeat
- **Isolated**: dedicated agent turn in `cron:<jobId>` or custom session with optional delivery

### Three Schedule Kinds

- `at` — one-shot timestamp (ISO 8601)
- `every` — fixed interval in milliseconds
- `cron` — 5 or 6-field cron expression with optional IANA timezone

### Job Structure

- Schedule (when)
- Payload (what — `systemEvent` or `agentTurn`)
- Delivery mode (`announce`, `webhook`, `none`)
- Optional agent binding (`agentId`)

One-shot jobs auto-delete after success. Set `deleteAfterRun: false` to retain.

### Payload Fields (agentTurn)

- `message` — required text prompt
- `model` / `thinking` — optional overrides
- `timeoutSeconds` — optional timeout
- `lightContext` — optional lightweight bootstrap mode

### Delivery Modes

- **announce** — posts summary to target channel and main session
- **webhook** — posts to HTTP(S) URL
- **none** — internal only

### Storage

- Job store: `~/.openclaw/cron/jobs.json`
- Run history: `~/.openclaw/cron/runs/<jobId>.jsonl` (auto-pruned)
- Session retention: `cron.sessionRetention` (default `24h`)

### Retry Policy

- **Transient errors (retried):** Rate limits, provider overload, network errors, 5xx, Cloudflare
- **Permanent errors (no retry):** Auth failures, config/validation errors
- One-shot: up to 3 retries with exponential backoff (30s → 1m → 5m)
- Recurring: exponential backoff (30s → 1m → 5m → 15m → 60m) before next scheduled run

### Config

```json5
{
  cron: {
    enabled: true,
    store: "~/.openclaw/cron/jobs.json",
    maxConcurrentRuns: 1,
    sessionRetention: "24h",
    runLog: { maxBytes: "2mb", keepLines: 2000 }
  }
}
```

---

## Cron vs Heartbeat

### Heartbeats

- Run periodically in main session (default every 30 min)
- Batch multiple checks together
- Context-aware (leverage main session history)
- Cost: one turn per interval
- Returns `HEARTBEAT_OK` when nothing needs attention

**Use for:** Bundled monitoring (inbox, calendar, notifications), context-aware decisions, routine checks.

### Cron

- Execute at precise times
- Can use isolated sessions or main session
- Can use different models or thinking levels
- Session isolation prevents history clutter
- Immediate delivery via announce mode

**Use for:** Exact timing ("9:00 AM sharp"), different models/heavy analysis, one-shot reminders, noisy/frequent operations.

### Best Practice

Combine both: heartbeat for routine monitoring, cron for precisely-timed or isolated tasks.

---

## Hooks

Event-driven automation through scripts that execute in response to agent commands and lifecycle events.

### Two Categories

- **Gateway Hooks** — execute inside the Gateway on events (`/new`, `/reset`, `/stop`)
- **Webhooks** — external HTTP endpoints triggering work in OpenClaw

### Discovery Hierarchy (precedence order)

1. Bundled hooks (lowest)
2. Plugin hooks
3. Managed hooks (`~/.openclaw/hooks/`)
4. Workspace hooks (`<workspace>/hooks/`, highest priority)

### Bundled Hooks

1. **session-memory** — preserves session context on `/new` or `/reset`
2. **bootstrap-extra-files** — injects additional workspace bootstrap files
3. **command-logger** — logs commands to `~/.openclaw/logs/commands.log`
4. **boot-md** — executes `BOOT.md` at gateway startup

### Supported Event Types

- **Command**: `command:new`, `command:reset`, `command:stop`
- **Session**: `session:compact:before`, `session:compact:after`, `session:patch`
- **Agent**: `agent:bootstrap`
- **Gateway**: `gateway:startup`
- **Message**: `message:received`, `message:transcribed`, `message:preprocessed`, `message:sent`

### Hook Structure

Each hook requires:
- `HOOK.md` — YAML frontmatter metadata + markdown docs
- `handler.ts` — TypeScript handler function

---

## Standing Orders

Grant agents permanent operating authority for defined programs. Agents execute autonomously within boundaries.

### Key Concepts

- Stored in workspace files, preferably `AGENTS.md` for guaranteed session availability
- Each program specifies: scope, triggers, approval gates, escalation rules
- Execution follows "Execute-Verify-Report" pattern — no exceptions
- Work alongside cron: standing orders define authorization, cron enforces timing

### Best Practices

- Start narrow, expand with demonstrated reliability
- Include explicit escalation criteria and "What NOT to do" sections
- Use separate programs for distinct domains
- Maintain as living documents with regular review
