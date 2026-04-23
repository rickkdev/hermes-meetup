# RAG vs No-RAG — Hermes vs OpenClaw

Slide 6 of 26

Core difference
- Hermes separates always-on memory from searchable history; OpenClaw workflows are typically more run-centric with less explicit two-lane memory architecture.

Hermes on this slide
- Clear split: MEMORY/USER for stable facts + state.db/session search for on-demand recall.

OpenClaw on this slide
- Commonly relies on current run context plus repo/files; long-term recall is often externalized or less opinionated.

Talk track (one-liner)
- Hermes gives you a cleaner mental model: write stable facts once, retrieve transient context only when needed.

Assumption
- This comparison treats OpenClaw as a coding-first agent setup without Hermes-specific memory/cron/webhook stack enabled.
