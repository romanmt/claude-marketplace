---
name: openclaw-architect
description: Use when creating, auditing, or improving OpenClaw workspace configurations — agent definitions, multi-agent routing, workspace files (SOUL/IDENTITY/AGENTS/BOOTSTRAP/USER/TOOLS/HEARTBEAT/MEMORY), satellite workspaces, standing orders, cron/heartbeat setup, or evaluating architecture decisions
---

# Skill: OpenClaw Architect

Expert-level guidance for designing, auditing, and improving OpenClaw multi-agent configurations. Grounded in official OpenClaw documentation.

---

## Reference Files

**Official docs (read before any operation):**

- `references/official-docs-system-prompt.md` — Bootstrap files, prompt modes, injection rules
- `references/official-docs-multi-agent.md` — Agent isolation, routing, bindings, config structure
- `references/official-docs-memory.md` — Memory files, vector search, daily logs
- `references/official-docs-automation.md` — Cron, heartbeats, hooks, standing orders
- `references/official-docs-agent-loop.md` — Agent loop, context engine, plugins, nodes, security
- `references/official-docs-subagents.md` — Sub-agent spawning, orchestrator pattern, tool access, announce

**Workspace context (read for current state):**

- `IDENTITY.md` — Current agent roster
- `SOUL.md` — Core philosophy
- `AGENTS.md` — Session startup procedures
- `brain/reference/working-genius-agent-reference.md` — Working Genius framework

---

## How OpenClaw Works (Official)

### Agent Isolation

Each agent is an **isolated entity** with its own:
- Workspace directory (set via `openclaw agents add <name> --workspace <path>`)
- State directory (`~/.openclaw/agents/<agentId>/`)
- Session store, auth profiles, model registry, and per-agent config

**Critical:** Never reuse `agentDir` across agents — causes auth/session collisions.

**Default single-agent mode:** `agentId` = `"main"`, workspace at `~/.openclaw/workspace`.

### Bootstrap File Injection

OpenClaw automatically injects these workspace files into the system prompt on **every turn**:

| File | Purpose | Injected |
|------|---------|----------|
| `AGENTS.md` | Session startup, procedures, safety | Every turn (full + sub-agent) |
| `SOUL.md` | Personality, values, boundaries | Every turn (full only) |
| `TOOLS.md` | Environment-specific local config | Every turn (full + sub-agent) |
| `IDENTITY.md` | Agent identity, profile | Every turn (full only) |
| `USER.md` | Human context | Every turn (full only) |
| `HEARTBEAT.md` | Proactive background tasks | Every turn (full only) |
| `BOOTSTRAP.md` | First-run onboarding (new workspaces) | Every turn (full only) |
| `MEMORY.md` | Curated long-term memory | Every turn (full only) |

**Sub-agents receive only `AGENTS.md` and `TOOLS.md`.** Not SOUL, IDENTITY, USER, MEMORY, or HEARTBEAT.

**Size limits:**
- Per file: `agents.defaults.bootstrapMaxChars` (default: 20,000)
- Total: `agents.defaults.bootstrapTotalMaxChars` (default: 150,000)

### Prompt Modes

- **Full** (default): All sections
- **Minimal** (sub-agents): Tooling, Safety, Workspace, Sandbox, Date/Time, Runtime + context only
- **None**: Base identity only

---

## Workspace File Reference

### IDENTITY.md — Who the agent is

**Purpose:** Agent identity — name, personality, visual identity. Auto-injected every turn.

**Contains:** Name, role, creature metaphor, vibe, emoji, avatar, model tier, Working Genius profile.

**Does NOT contain:** Job instructions, skills, schedules, file paths, operational procedures.

**CLI alternative:** `openclaw agents set-identity --agent <id> --name "Name" --emoji "🌑" --avatar path`

**Shared workspace:** One IDENTITY.md defines all agents on the team.
**Satellite workspace:** Each agent gets its own IDENTITY.md (or loads via `--from-identity`).

---

### SOUL.md — How the agent thinks and behaves

**Purpose:** Core values, personality, behavioral boundaries, Working Genius behavioral descriptions. Auto-injected every turn (full mode only, not sub-agents).

**Contains:** Core truths, WG genius/competency/frustration behaviors (role-specific), communication style, domain boundaries, shared workspace paths.

**Does NOT contain:** Step-by-step procedures, skill instructions, task lists.

**Critical for satellites:** This is where per-agent differentiation happens. Each satellite SOUL.md defines what the agent does and doesn't touch.

---

### AGENTS.md — What to do each session

**Purpose:** Session startup procedure, workspace orientation, operational rules. Auto-injected every turn (including sub-agents).

**Contains:** Session startup sequence, memory system instructions, safety rules, group chat etiquette, heartbeat/cron guidance, standing orders.

**Does NOT contain:** Agent personality (SOUL.md), agent identity (IDENTITY.md), job procedures (skills).

**Standing orders belong here** per official docs: "Stored in workspace files, preferably AGENTS.md for guaranteed session availability."

**Sub-agent note:** Since AGENTS.md is one of only two files sub-agents receive, keep it focused on universal procedures — not personality content.

---

### BOOTSTRAP.md — First-run onboarding

**Purpose:** One-time self-discovery script for fresh workspaces. Injected only on new workspaces. Agent deletes it when done.

**Contains:** Conversational onboarding flow (name, creature, vibe, emoji), instructions to populate IDENTITY.md and USER.md, prompt to discuss SOUL.md values.

**Self-destructs.** If it still exists after onboarding, it's a bug.

---

### USER.md — Who the human is

**Purpose:** Human context for the agent to tailor behavior. Auto-injected every turn (full mode only).

**Contains:** Name, pronouns, timezone, background, business constraints, anti-interests, current projects.

**Single source of truth.** In multi-agent setups, USER.md lives in one place. Satellites reference it via path in AGENTS.md or SOUL.md.

---

### TOOLS.md — Environment-specific notes

**Purpose:** Local configuration unique to the runtime environment. Auto-injected every turn (including sub-agents).

**Contains:** Device names, SSH hosts, voice preferences, credentials references, environment-specific values.

**Not how tools work** (that's skills). Just the specific values this agent needs.

---

### HEARTBEAT.md — Proactive background checklist

**Purpose:** Instructions for heartbeat polls (default every 30 min in main session). Auto-injected every turn (full mode only).

**Empty = skip heartbeat work.** Keeps API costs down when there's nothing to check.

---

### MEMORY.md — Curated long-term memory

**Purpose:** Distilled insights from daily logs. Auto-injected every turn (full mode only).

**Security:** Contains personal context. Only load in private/main sessions. The official docs note daily logs (`memory/YYYY-MM-DD.md`) read today + yesterday at session start.

**Vector search available:** `openclaw memory search <query>` for semantic recall across indexed memory files.

---

## Configuration Architectures

### 1. Single Agent

One agent, one workspace. Default mode.

```
~/.openclaw/workspace/          # agentId: "main"
├── IDENTITY.md
├── SOUL.md
├── AGENTS.md
├── USER.md
├── TOOLS.md
├── HEARTBEAT.md
├── MEMORY.md
├── skills/
└── memory/
```

**Setup:** `openclaw setup --workspace ~/.openclaw/workspace`

**When to use:** Solo projects, getting started, one personality for all work.

---

### 2. Shared Workspace (Multi-Agent, Same Workspace)

Multiple agents point to the same workspace directory. All agents share the same bootstrap files, skills, and memory.

```
~/.openclaw/workspace/          # Shared by all agents
├── IDENTITY.md                # All agents defined here
├── SOUL.md                    # Shared philosophy
├── AGENTS.md                  # Shared procedures
├── USER.md
├── skills/
├── brain/
└── memory/
```

**Setup:**
```bash
openclaw agents add agent-b --workspace ~/.openclaw/workspace
# agent-b shares the same workspace as the default "main" agent
```

**Routing:** Bind each agent to different channels/accounts via bindings config.

**When to use:** Small teams (2-3 agents) doing closely related work. Simplest multi-agent setup.

**Pros:** Zero coordination overhead, single config location, skills shared automatically.

**Cons:** No per-agent SOUL customization, no domain isolation, shared memory (context leakage), all agents see the same IDENTITY.md and get the same personality injection.

---

### 3. Isolated Workspaces (Multi-Agent, Separate Workspaces)

Each agent has its own workspace directory with its own bootstrap files. The official architecture for agent isolation.

```
~/.openclaw/workspace/           # Agent: main (hub/coordinator)
├── IDENTITY.md
├── SOUL.md
├── AGENTS.md
├── USER.md
├── skills/                     # Can be shared via symlink or global skills
├── brain/
└── memory/

~/.openclaw/workspace-coda/      # Agent: coda
├── IDENTITY.md                 # Coda-specific identity
├── SOUL.md                     # Coda-specific personality + boundaries
├── AGENTS.md                   # Coda-specific startup + skill paths
├── TOOLS.md
├── skills/                     # Agent-specific skills (workspace takes precedence)
└── memory/

~/.openclaw/workspace-rook/      # Agent: rook
├── ...
```

**Setup:**
```bash
openclaw agents add coda --workspace ~/.openclaw/workspace-coda
openclaw agents add rook --workspace ~/.openclaw/workspace-rook
openclaw agents add vale --workspace ~/.openclaw/workspace-vale
```

**When to use:** Teams of 3+ agents with distinct roles, when agents need different personalities, when domain boundaries matter.

**Pros:**
- Per-agent SOUL.md (personality, WG behaviors, boundaries)
- Per-agent memory (no context leakage)
- Per-agent model, sandbox, and tool configuration
- Domain isolation via SOUL.md boundary rules
- Each agent gets exactly the bootstrap context it needs

**Cons:**
- More files to maintain
- Skills need explicit sharing (symlinks, global skills at `~/.openclaw/skills`, or absolute paths)
- Path management across workspaces
- Configuration drift between workspaces

### Skill Sharing in Isolated Workspaces

Three approaches (per official docs):

1. **Global skills directory:** `~/.openclaw/skills/` — available to all agents automatically
2. **Workspace skills:** `<workspace>/skills/` — per-agent, takes precedence over global
3. **Symlinks:** Symlink satellite `skills/` to hub `skills/` for shared access

### Hub-and-Satellite Pattern (Hybrid)

A practical combination: one "hub" workspace holds shared resources (brain, projects, standards), while satellite workspaces reference the hub via absolute paths in their SOUL.md and AGENTS.md.

**Key design decisions:**
- Hub holds canonical IDENTITY.md (team roster), shared skills, brain, projects
- Satellites have their own SOUL.md, AGENTS.md, optionally IDENTITY.md
- Satellites reference hub paths absolutely (e.g., `/home/user/.openclaw/workspace/brain/`)
- Domain boundaries enforced in each satellite's SOUL.md
- Inter-agent communication through shared hub directories
- Skills: either symlinked or referenced via global `~/.openclaw/skills/`

---

## Multi-Agent Routing (Official)

### Setup Steps

1. Create agent workspaces: `openclaw agents add <name> --workspace <path>`
2. Create channel accounts (Discord, Telegram, WhatsApp, etc.)
3. Configure agents, accounts, and bindings
4. Restart gateway: verify with `openclaw agents list --bindings`

### Routing Priority (Most-Specific Wins)

1. Exact peer match (DM/group/channel ID)
2. Parent peer match (thread inheritance)
3. Guild ID + roles (Discord)
4. Guild ID only (Discord)
5. Team ID (Slack)
6. Account ID match
7. Channel-level match with wildcard
8. Fallback to default agent

Multiple match fields = AND semantics (all must match).

### Binding Commands

```bash
openclaw agents bind --agent coda --bind telegram:product-bot
openclaw agents unbind --agent coda --bind telegram:product-bot
openclaw agents bindings                     # view all
openclaw agents bindings --agent coda        # view specific
```

### Per-Agent Configuration

Each agent supports independent:
- `sandbox.mode` / `sandbox.scope`
- `tools.allow` / `tools.deny`
- Model selection
- Thinking level

---

## Automation Integration

### Standing Orders in AGENTS.md

Per official docs, standing orders define permanent operating authority:

```markdown
## Standing Orders

### Program: Daily SEO Scan
- **Scope:** Run barboard-seo-audit skill, write report to brain/reports/seo/
- **Trigger:** Cron job, daily at 7:00 AM ET
- **Approval:** None required for report writing
- **Escalation:** Flag findings scoring >8 impact to Matt via Telegram
- **What NOT to do:** Don't modify any code or backlog items
```

### Heartbeat vs Cron Decision

| Factor | Heartbeat | Cron |
|--------|-----------|------|
| Timing | ~30 min intervals (approximate) | Exact (cron expression + timezone) |
| Session | Main session (context-aware) | Isolated or main session |
| Model | Main session model | Can override per-job |
| Cost | One turn per interval | Full agent turn per job |
| Delivery | Within main session | Can announce to channels |
| Best for | Batched monitoring, context-aware checks | Precise timing, isolated tasks, different models |

### Cron Setup

```bash
# Recurring isolated job
openclaw cron add --name "Morning brief" --cron "0 7 * * *" \
  --tz "America/New_York" --session isolated \
  --message "Run barboard-seo-audit skill" \
  --announce --channel telegram --to "user:12345"

# One-shot reminder
openclaw cron add --name "Reminder" --at "2026-04-01T16:00:00Z" \
  --session main --system-event "Check PR status" --delete-after-run
```

---

## Sub-Agent Architecture

Sub-agents are background runs spawned from an existing agent session. Critical for multi-agent orchestration.

### Bootstrap Context (Critical)

**Sub-agents receive ONLY `AGENTS.md` + `TOOLS.md`.** Everything else (SOUL.md, IDENTITY.md, USER.md, HEARTBEAT.md, MEMORY.md) is NOT injected.

**Design implication:** Any information sub-agents need must be either:
1. In AGENTS.md or TOOLS.md
2. Passed explicitly in the `task` parameter when spawning
3. Available via file paths the sub-agent can read

### Spawning

```bash
# Via slash command
/subagents spawn <agentId> <task> [--model <model>] [--thinking <level>]

# Programmatic (sessions_spawn tool)
sessions_spawn({ task: "...", model: "opus", thinking: "medium" })
```

Non-blocking. Returns run ID immediately. Sub-agent announces results back on completion.

### Orchestrator Pattern (Nested Sub-Agents)

Default: `maxSpawnDepth: 1` (sub-agents can't spawn children).

Set `maxSpawnDepth: 2` for main → orchestrator → workers:

| Depth | Role | Can Spawn? | Gets Session Tools? |
|-------|------|-----------|-------------------|
| 0 | Main agent | Always | Yes |
| 1 | Orchestrator | If maxSpawnDepth ≥ 2 | Yes (spawn, list, history) |
| 2 | Leaf worker | Never | No |

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2,
        maxChildrenPerAgent: 5,    // range 1-20
        maxConcurrent: 8,          // global cap
        runTimeoutSeconds: 900,
      },
    },
  },
}
```

### Cross-Agent Spawning

Agents can spawn sub-agents under a different registered agent:
- Controlled by `agents.list[].subagents.allowAgents`
- Default: only requester's own agent ID
- `["*"]` allows any agent
- Sandbox guard: sandboxed requesters can't spawn unsandboxed targets

### Model Inheritance

Explicit param > agent config > global default > caller's model

### Tool Access

Sub-agents get all tools EXCEPT session/system tools by default. Override via:

```json5
{ tools: { subagents: { tools: { deny: ["gateway", "cron"] } } } }
```

---

## Operations

### MODE: AUDIT

Evaluate an existing configuration for correctness against official OpenClaw documentation.

#### Step 1 — Inventory

Read all workspace files across all agent workspaces. Build a table:

```
| File | main | coda | rook | vale | Status |
|------|------|------|------|------|--------|
| IDENTITY.md | ✅ | ✅ | ✅ | ❌ | Incomplete |
| SOUL.md | ✅ | ✅ | ✅ | ✅ | OK |
| AGENTS.md | ✅ | ✅ | ✅ | ⚠️ | Needs update |
| BOOTSTRAP.md | — | ✅! | — | ✅! | Should be deleted |
```

#### Step 2 — Bootstrap File Compliance

Check each file against official injection rules:
- Are files within the 20,000 char per-file limit?
- Is total bootstrap under 150,000 chars?
- Is content in the correct file? (personality in SOUL, not AGENTS; procedures in AGENTS, not SOUL)
- Do sub-agent-facing files (AGENTS.md, TOOLS.md) avoid personality content?
- Are standing orders in AGENTS.md (guaranteed availability)?

#### Step 3 — Multi-Agent Compliance

For isolated workspace setups:
- Is each agent registered? (`openclaw agents list`)
- Are bindings configured? (`openclaw agents bindings`)
- Do absolute paths in satellite files resolve to real hub files?
- Are domain boundaries defined in each satellite SOUL.md?
- Is skill sharing set up correctly (global, symlink, or per-workspace)?
- Are there stale BOOTSTRAP.md files?
- Does each agent have its own `agentDir`? (Never shared)

#### Step 4 — Automation Check

- Are standing orders defined in AGENTS.md (not SOUL.md or other files)?
- Are heartbeat tasks in HEARTBEAT.md (or empty if not needed)?
- Are cron jobs using appropriate session types (isolated vs main)?
- Is MEMORY.md only loaded in private sessions?

#### Step 5 — Report

Prioritized findings:
1. **Critical** — Broken paths, missing agent registrations, shared agentDirs, files exceeding size limits
2. **Important** — Content in wrong files, stale BOOTSTRAP.md, missing boundaries, outdated skill references
3. **Advisory** — Formatting, missing optional fields, optimization opportunities

---

### MODE: IMPROVE

Iteratively improve configuration based on audit findings or user goals.

#### Step 1 — Scope

Confirm focus area with user:
- Fix audit findings?
- Add a new agent workspace?
- Restructure (shared → isolated)?
- Update personalities or Working Genius profiles?
- Add/improve standing orders?
- Set up or optimize cron/heartbeat?

#### Step 2 — Propose Changes

For each change, show:
- **File:** Which file
- **Before:** Current content (relevant section)
- **After:** Proposed content
- **Why:** Rationale, citing official docs where applicable

Wait for approval before writing anything.

#### Step 3 — Apply & Verify

Apply approved changes. Re-run relevant audit checks to confirm consistency.

---

### MODE: SCAFFOLD

Create a new agent workspace from scratch.

#### Step 1 — Register Agent

```bash
openclaw agents add <name> --workspace ~/.openclaw/workspace-<name>
```

#### Step 2 — Create Bootstrap Files

Write SOUL.md, AGENTS.md, and optionally IDENTITY.md, TOOLS.md, BOOTSTRAP.md.

Pull identity from hub IDENTITY.md if using hub-and-satellite pattern. Expand Working Genius into role-specific behavioral descriptions.

#### Step 3 — Configure Skills

Choose approach:
- Symlink to hub: `ln -s /path/to/hub/skills /path/to/satellite/skills`
- Use global: place skills in `~/.openclaw/skills/`
- Copy specific skills to workspace `skills/` directory

#### Step 4 — Set Up Routing

```bash
openclaw agents bind --agent <name> --bind <channel>:<account>
openclaw agents set-identity --agent <name> --from-identity
```

#### Step 5 — Verify

- Confirm agent appears in `openclaw agents list`
- Confirm bindings with `openclaw agents bindings --agent <name>`
- Confirm all paths in SOUL.md and AGENTS.md resolve
- If BOOTSTRAP.md exists, confirm it's intentional (agent not yet onboarded)

---

### MODE: MIGRATE

Convert shared workspace to isolated workspace architecture.

#### Step 1 — Analyze

Read shared workspace. Identify agents in IDENTITY.md, content that should be per-agent in SOUL.md, skill ownership, directory write-ownership.

#### Step 2 — Design

For each agent, determine: satellite SOUL.md content, AGENTS.md content, skill sharing strategy, domain boundaries, hub paths to reference.

#### Step 3 — Plan

Present full migration plan. Include:
- Agent registration commands
- Files to create per satellite
- Content splits (what moves from shared SOUL to satellite SOULs)
- Skill sharing approach
- Binding configuration
- What stays in hub

#### Step 4 — Execute

After approval:
1. Create satellite directories and files
2. Register agents: `openclaw agents add <name> --workspace <path>`
3. Configure bindings
4. Set identities
5. Do NOT modify hub — satellites are additive
6. Restart gateway

---

## File Separation Rules

```
IDENTITY.md  = WHO    → name, personality, WG profile, visual identity
SOUL.md      = HOW    → values, behaviors, boundaries, WG behavioral descriptions
AGENTS.md    = WHAT   → startup procedures, skills list, safety, standing orders
TOOLS.md     = WHERE  → environment-specific local config
BOOTSTRAP.md = FIRST  → one-time onboarding, self-destructs
USER.md      = FOR    → human context, preferences, constraints
HEARTBEAT.md = WHEN   → periodic background check instructions
MEMORY.md    = RECALL → curated long-term memory (private sessions only)
Skills       = JOB    → step-by-step job procedures (separate SKILL.md files)
```

**Sub-agents only see:** AGENTS.md + TOOLS.md. Keep these two files universally useful.

---

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Job instructions in IDENTITY.md | IDENTITY is WHO, not WHAT | Move to skills or AGENTS.md |
| Personality content in AGENTS.md | Sub-agents receive AGENTS.md — personality bleeds | Move to SOUL.md |
| Standing orders in SOUL.md | Not guaranteed for sub-agents; SOUL is character | Move to AGENTS.md |
| Shared `agentDir` across agents | Causes auth/session collisions (per official docs) | Each agent needs unique agentDir |
| Relative paths in satellite configs | Satellites have different working directories | Use absolute paths |
| BOOTSTRAP.md after onboarding | Wastes tokens every turn, confuses agent | Delete it |
| No domain boundaries in SOUL.md | Agents write to each other's directories | Add explicit "don't write to X" rules |
| MEMORY.md in group chats | Security-sensitive personal context leaks | Only load in main/private sessions |
| Bootstrap files exceeding 20K chars | Truncated silently, agent misses content | Trim or move to skills/references |
| Skills duplicated into satellites | Maintenance burden, version drift | Use global skills dir or symlinks |

---

## Guardrails

- **User approves all changes.** Never write to configuration files without explicit approval.
- **Official docs are source of truth.** Reference `references/official-docs-*.md` files, not assumptions.
- **Each agent gets unique agentDir.** Never share state directories between agents.
- **Paths must resolve.** Before writing a config that references a path, verify it exists.
- **Respect file size limits.** 20K per file, 150K total bootstrap. Check before writing large content.
- **SOUL.md is sacred.** Changes should be surfaced and discussed, not silently applied.
- **Sub-agent awareness.** AGENTS.md and TOOLS.md are the only files sub-agents see — design accordingly.
