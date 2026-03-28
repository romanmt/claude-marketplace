# OpenClaw Official Docs: Memory System

Source: https://docs.openclaw.ai/concepts/memory

---

## Core Principle

"OpenClaw memory is plain Markdown in the agent workspace. The files are the source of truth; the model only 'remembers' what gets written to disk."

## Memory File Structure

Two layers:

1. **Daily logs** (`memory/YYYY-MM-DD.md`): Append-only. Read at session start (today + yesterday).
2. **Long-term memory** (`MEMORY.md`): Optional curated storage. Loaded only in private sessions.

Both reside in the workspace directory (default: `~/.openclaw/workspace`).

## Agent Tools

- `memory_search` — Semantic recall over indexed snippets
- `memory_get` — Targeted file/line-range reads (graceful degradation when files don't exist)

## When to Write Memory

- Recording decisions, preferences, permanent facts → `MEMORY.md`
- Logging day-to-day notes → `memory/YYYY-MM-DD.md`
- Someone explicitly requests remembering something
- Information that must persist beyond the session

## Automatic Memory Flush

Before context compaction, OpenClaw triggers a silent agent turn reminding the model to preserve durable memories. Controlled via config with a soft threshold of 4,000 tokens.

## Vector Memory Search

Semantic indexing across memory files using embeddings from OpenAI, Gemini, Voyage, Mistral, Ollama, or GGUF models. Optional hybrid (BM25 + vector) search.

## CLI Commands

- `openclaw memory status` — display memory system status
- `openclaw memory index` — perform semantic indexing
- `openclaw memory search <query>` — query the memory index
- Flags: `--agent <id>`, `--force` (full reindex), `--deep`, `--json`
