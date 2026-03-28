# OpenClaw Official Docs: Multi-Agent Routing

Source: https://docs.openclaw.ai/concepts/multi-agent

---

## Core Concept

An "agent" is an isolated entity with its own:
- Workspace directory
- State directory (`agentDir` at `~/.openclaw/agents/<agentId>/`)
- Session store (`~/.openclaw/agents/<agentId>/sessions`)
- Auth profiles (`~/.openclaw/agents/<agentId>/agent/auth-profiles.json`)
- Model registry
- Per-agent config

**Critical:** Never reuse `agentDir` across agents — causes auth/session collisions.

## Default (Single-Agent) Mode

- `agentId` defaults to `"main"`
- Sessions keyed as `agent:main:<mainKey>`
- Workspace at `~/.openclaw/workspace`

## Multi-Agent Setup Steps

1. Create agent workspaces: `openclaw agents add <name>`
2. Create channel accounts (Discord, Telegram, WhatsApp, etc.)
3. Configure agents, accounts, and bindings in config file
4. Restart gateway, verify: `openclaw agents list --bindings`

## Agent Management Commands

- `openclaw agents add work --workspace ~/.openclaw/workspace-work` — create isolated agent
- `openclaw agents delete work` — remove agent
- `openclaw agents list` — list agents
- `openclaw agents bindings` — view all bindings
- `openclaw agents bind --agent work --bind telegram:ops` — add binding
- `openclaw agents unbind --agent work --bind telegram:ops` — remove binding
- `openclaw agents set-identity --agent main --name "Nox" --emoji "🌑"` — set identity
- `openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity` — load from workspace

## Routing Rules (Priority Order — Most Specific Wins)

1. Exact peer match (DM/group/channel ID)
2. Parent peer match (thread inheritance)
3. Guild ID + roles (Discord)
4. Guild ID only (Discord)
5. Team ID (Slack)
6. Account ID match
7. Channel-level match with wildcard
8. Fallback to default agent

When multiple match fields exist in a binding, ALL are required (AND semantics).

## Configuration Structure

Multi-agent setups require entries under:
- `agents.list` — agent definitions
- `channels.<channel>.accounts` — channel account configs
- `bindings` — route channel/account/peer to agent

## Multi-Account Patterns

Channels supporting multiple accounts (WhatsApp, Telegram, Discord, Slack, Signal, iMessage, IRC, LINE) use `accountId` to identify each login.

- Binding omitting `accountId` matches only the default account
- Use `accountId: "*"` for channel-wide fallback across all accounts
- One server can host multiple phone numbers without mixing sessions

## WhatsApp Single-Number Routing

One WhatsApp account can route different DMs to different agents by matching sender E.164 numbers with `peer.kind: "direct"`. True isolation requires one agent per person (DMs collapse to agent's main session key).

## Per-Agent Sandbox & Tool Configuration

Each agent supports independent:
- `sandbox.mode`: `"off"` or `"all"` (always sandboxed)
- `sandbox.scope`: `"agent"` (one container per agent)
- `tools.allow` and `tools.deny` lists
- `sandbox.docker.setupCommand` — runs once on container creation

**Note:** `tools.elevated` remains global and sender-based, not per-agent.

## Skills

- Per-agent via workspace `skills/` folders
- Shared skills from `~/.openclaw/skills` (global)
- Skills in workspace take precedence over global skills
