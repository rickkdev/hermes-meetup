# Self-Sovereign AI — Hermes vs OpenClaw (researched)

Slide 1 of 26

Focus
- Platform posture

Core differentiation
- Both are capable agent platforms; the strongest practical difference is how memory/automation is packaged and configured by default.

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
- Hermes in this repo is more opinionated for immediate continuity; OpenClaw is more explicitly plugin-composable.

Evidence URLs
- https://docs.openclaw.ai/plugins/memory-wiki
- https://docs.openclaw.ai/concepts/session
- https://docs.openclaw.ai/concepts/session-tool
- https://docs.openclaw.ai/tools
- https://docs.openclaw.ai/plugins/webhooks
- https://docs.openclaw.ai/automation/hooks

Note
- This comparison is now grounded in OpenClaw docs and avoids the earlier oversimplification that OpenClaw lacks webhook/memory features.
