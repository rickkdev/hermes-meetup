# Coding Workflow Choices — Where Hermes Fits

Slide 14 of 26

## One-liner
Claude Code / Codex / Open Code are coding specialists inside a repo. Cursor is an in-editor pair programmer. Ralph loops are for bigger one-shot epic runs and overnight execution. Hermes is a self-improving autonomous agent that lives outside all of them.

Most power users run two or three together, not one.

## Comparison at a glance
| Feature | Claude Code / Codex / Open Code | Cursor | Ralph loops | Hermes Agent |
|---|---|---|---|---|
| Primary surface | CLI in a repo | IDE | Loop scripts + terminal | CLI + chat + cron + Telegram |
| Persistent memory | Mostly session-scoped | Project/editor-scoped | File/process-driven | Cross-session bounded (`MEMORY.md`, `USER.md`) |
| Learning | None | None | None | Auto-generates skills after repeated patterns |
| Channels | Terminal | Editor | Terminal / scheduled runs | Terminal, Telegram, Discord, email, webhooks |
| Scheduled jobs | No | No | Yes (overnight loops) | Built-in cron |
| Self-improvement | No | No | No | Yes (skills + memory) |
| Best for | Read/edit/test/commit loops | Writing/fixing code inline | Bigger epics + overnight task runs | Long-running autonomous tasks that compound |

## When to pick which
- Pick Claude Code/ Codex/ Open Code when you're inside a repo and want the agent to read code, edit code, run tests, commit. It's a coding specialist.
- Pick Cursor when you want AI completions and fixes inside your editor in real time.
- Pick Ralph loops for bigger epics you want to one shot and generall tasks you want to run over night.
- Pick Hermes when you want an agent that (a) lives beyond any single session, (b) talks to you from any channel, (c) gets better at your recurring work over time.

## Practical stack recommendation
- Repo-heavy build loop: Claude Code/Codex/Open Code + Hermes
- IDE-heavy coding loop: Cursor + Hermes
- Big epic overnight execution: Ralph loops (+ Hermes for memory + multi-channel control)
- If your priority is long-term compounding leverage, include Hermes in the stack.
