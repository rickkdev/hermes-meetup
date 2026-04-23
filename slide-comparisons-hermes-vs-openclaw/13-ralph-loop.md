# Ralph Loop — Hermes vs OpenClaw (researched)

Slide 13 of 26

Focus
- Agent orchestration

Core differentiation
- Both support session-based orchestration/sub-agent style flows; Hermes emphasizes persistent skills/memory compounding in this deck context.

What is confirmed about OpenClaw
- OpenClaw does not expose one single hardcoded memory engine; memory is plugin-based.
- Docs explicitly describe an "active memory plugin" owning recall, promotion, indexing, and dreaming.
- Memory Wiki is a bundled plugin that adds a compiled knowledge vault and does not replace the active memory plugin.
- Session/history state is managed in the gateway state directory (OPENCLAW_STATE_DIR) and accessed through session tooling.

What Hermes does in this setup
- In this Hermes setup, stable user/profile facts are stored in markdown memory files and injected each session.
- Conversation history is stored in a state DB and retrieved on demand via session search.
- So Hermes presents an explicit two-lane mental model by default: always-on memory vs searchable history.

Bottom line for this slide
- Choose by operating style: modular gateway/plugin ecosystem (OpenClaw) vs tighter built-in memory+skills loop (Hermes).

Evidence URLs
- https://docs.openclaw.ai/plugins/memory-wiki
- https://docs.openclaw.ai/concepts/session
- https://docs.openclaw.ai/concepts/session-tool
- https://docs.openclaw.ai/tools
- https://docs.openclaw.ai/plugins/webhooks
- https://docs.openclaw.ai/automation/hooks

Note
- This comparison is now grounded in OpenClaw docs and avoids the earlier oversimplification that OpenClaw lacks webhook/memory features.
