# Coding Workflow Choices — Where Hermes Fits

Slide 14 of 26

## One-liner
Claude Code is an in-repo coding agent. Cursor is an in-editor pair programmer. OpenClaw is a configuration-driven task runner. Hermes is a self-improving autonomous agent that lives outside all of them.

Most power users run two or three together, not one.

## Comparison at a glance
| Feature | Claude Code | Cursor | OpenClaw | Hermes Agent |
|---|---|---|---|---|
| Primary surface | CLI in a repo | IDE | CLI + configs | CLI + chat + cron + Telegram |
| Persistent memory | Session-scoped | Project-scoped | Config/plugin based | Cross-session bounded (`MEMORY.md`, `USER.md`) |
| Learning | None | None | None (default) | Auto-generates skills after repeated patterns |
| Channels | Terminal | Editor | Terminal | Terminal, Telegram, Discord, email, webhooks |
| Scheduled jobs | No | No | No | Built-in cron |
| Self-improvement | No | No | No | Yes (skills + memory) |
| Model lock-in | Anthropic | Multiple | Multiple | 18+ providers, swap with one command |
| Best for | Coding in a repo | Inline code work | Shell workflow orchestration | Long-running autonomous tasks that compound |

## When to pick which
- Pick Claude Code when you want deep in-repo read/edit/test/commit loops.
- Pick Cursor when you want real-time editor completions and fast inline fixes.
- Pick OpenClaw when you want declarative, configuration-first task running.
- Pick Hermes when you want:
  - cross-session continuity,
  - multi-channel operation,
  - and compounding improvement over recurring work.

## Practical stack recommendation
- Repo-heavy build loop: Claude Code + Hermes
- IDE-heavy coding loop: Cursor + Hermes
- Ops/workflow automation: OpenClaw or Hermes
- If your priority is long-term compounding leverage, include Hermes in the stack.
