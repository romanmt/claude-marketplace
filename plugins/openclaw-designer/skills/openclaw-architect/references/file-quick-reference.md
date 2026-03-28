# OpenClaw Configuration — Quick Reference

Based on official OpenClaw documentation at docs.openclaw.ai.

---

## Bootstrap File Injection (Official)

All files live in the agent's workspace directory. Injected into system prompt automatically.

| File | Injected In | Sub-Agents? | Max Size | Purpose |
|------|------------|-------------|----------|---------|
| `AGENTS.md` | Every turn | YES | 20K chars | Startup, procedures, safety, standing orders |
| `TOOLS.md` | Every turn | YES | 20K chars | Environment-specific local config |
| `SOUL.md` | Every turn | NO | 20K chars | Personality, values, boundaries |
| `IDENTITY.md` | Every turn | NO | 20K chars | Name, role, profile, visual identity |
| `USER.md` | Every turn | NO | 20K chars | Human context |
| `HEARTBEAT.md` | Every turn | NO | 20K chars | Proactive background tasks |
| `BOOTSTRAP.md` | New workspaces | NO | 20K chars | First-run onboarding (delete after) |
| `MEMORY.md` | Every turn | NO | 20K chars | Curated long-term memory |

**Total bootstrap limit:** 150,000 chars across all files.

---

## Architecture Decision Matrix

| Factor | Single Agent | Shared Workspace | Isolated Workspaces |
|--------|-------------|-----------------|-------------------|
| Agents | 1 | 2+ | 2+ |
| Workspaces | 1 | 1 (shared) | 1 per agent |
| SOUL per agent | No (one SOUL) | No (shared) | YES |
| Memory isolation | N/A | No (shared) | YES |
| Skill sharing | Built-in | Built-in | Global dir, symlinks, or per-workspace |
| Domain boundaries | N/A | No enforcement | Via SOUL.md boundary rules |
| Model per agent | N/A | Via config | Via config |
| Complexity | Low | Medium | High |
| Setup command | `openclaw setup` | `openclaw agents add` | `openclaw agents add --workspace` |

---

## Agent Management Commands

```bash
# Create
openclaw agents add <name> --workspace <path>

# List & inspect
openclaw agents list
openclaw agents list --bindings
openclaw agents bindings --agent <name>

# Identity
openclaw agents set-identity --agent <name> --name "Name" --emoji "🌑"
openclaw agents set-identity --agent <name> --from-identity

# Routing
openclaw agents bind --agent <name> --bind <channel>:<account>
openclaw agents unbind --agent <name> --bind <channel>:<account>

# Remove
openclaw agents delete <name>
```

---

## Satellite SOUL.md Template

```markdown
# SOUL.md - Who You Are

You are {Name} — {Project}'s {role}.

_{Creature description}_

## Core Truths

**Lead with {Genius1} and {Genius2}.** Your Working Genius is {CODE}.
{2-3 more core truths derived from the agent's role}

## Working Genius: {CODE} — {Archetype Name}

**Your Geniuses (lead with these):**
- **{G1}:** {Role-specific behavioral description}
- **{G2}:** {Role-specific behavioral description}

**Your Competencies (capable, not primary):**
- **{C1}:** {What agent can do but defers on, and to whom}
- **{C2}:** {What agent can do but defers on, and to whom}

**Your Frustrations (recognize and hand off):**
- **{F1}:** {What triggers handoff, to which agent}
- **{F2}:** {What triggers handoff, to which agent}

## Shared Project Space

All project files live at `{absolute-hub-path}`. Key paths:
- {Primary input}: `{absolute-path}`
- {Primary output}: `{absolute-path}`
- Team identity: `{absolute-hub-path}/IDENTITY.md`

Always use absolute paths when reading from or writing to the shared project space.

## Boundaries

- Don't write to `{other-agent-domain}` — that's {Agent}'s domain.
- Don't send external communications without approval.
- {Role-specific boundaries}
```

---

## Satellite AGENTS.md Template

```markdown
# AGENTS.md - {Name}'s Workspace

## Every Session

Before doing anything else:
1. Read `SOUL.md` — this is who you are
2. Check `{absolute-hub-path}/USER.md` — this is who you're helping

## Your Role

You are {Name}, {Project}'s {role}. Your primary outputs:
- {Output 1}
- {Output 2}

## Key Paths

- **Repo:** `{absolute-path}`
- **Backlog:** `{absolute-hub-path}/projects/{project}/backlog/`
- **Standards:** `{absolute-hub-path}/brain/standards/`

## Skills

- **{skill-name}** — {brief purpose}
  `{absolute-hub-path}/skills/{skill-name}/SKILL.md`

Read the relevant skill before starting work in its domain.

## Standing Orders

### Program: {Program Name}
- **Scope:** {Authorized actions}
- **Trigger:** {When this runs}
- **Approval:** {What needs human approval}
- **Escalation:** {When to flag the human}
- **What NOT to do:** {Explicit exclusions}

## Safety

- {Safety rule 1}
- {Safety rule 2}
- Don't send external communications without approval.
```

---

## Cron vs Heartbeat Quick Decision

```
Need exact timing?           → Cron
Need different model?        → Cron (isolated session)
Batching multiple checks?    → Heartbeat
Need main session context?   → Heartbeat
One-shot reminder?           → Cron with --at
Noisy/frequent task?         → Cron (isolated, won't clutter history)
```

---

## Skill Sharing Strategies

| Strategy | How | Best For |
|----------|-----|----------|
| Global directory | Place in `~/.openclaw/skills/` | Skills every agent needs |
| Workspace skills | Place in `<workspace>/skills/` | Agent-specific skills |
| Symlink | `ln -s /hub/skills /satellite/skills` | Hub-and-satellite with shared library |
| Absolute path refs | Reference in AGENTS.md | When agents need different subsets |

**Precedence:** Workspace skills > Global skills. Per-agent overrides are possible.
