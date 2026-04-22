# Hermes Meetup Presentation Notes

## 1) Opening
Hermes Agent is not just a coding assistant. It is an agent runtime that executes with tools, keeps continuity through memory, improves with skills, and runs across interfaces (CLI + messaging + automation).

Talk track:
- Most AI demos stop at “single conversation intelligence.”
- Hermes is built for operational intelligence across sessions.
- Goal of this talk: explain setup, architecture decisions, self-improvement loop, parallel execution, and knowledgebase integration with Obsidian.

---

## 2) What Hermes Actually Is
Core framing:
- Not a plain chatbot
- Not a single-provider wrapper
- Not stateless between runs

What it is:
- Tool-calling agent runtime
- Multi-provider model layer
- Persistent memory + reusable skills
- Multi-surface interface (terminal, Telegram, Discord, etc.)
- Automation hooks (cron, webhooks)

Why this matters:
- You can move from ad hoc prompts to repeatable systems.

---

## 3) Setup in 3 Minutes (practical)
Fast path:
1. Install Hermes
2. Run setup wizard
3. Choose model/provider
4. Run doctor check
5. Start chat

Key files:
- `~/.hermes/config.yaml` for behavior/config
- `~/.hermes/.env` for keys/secrets
- `~/.hermes/skills/` for procedural memory
- `~/.hermes/sessions/` for transcripts and continuity

Speaker point:
- Setup is intentionally easy. Hardening and optimization can come later.

---

## 4) Hermes vs OpenClaw
Use this framing respectfully:
- OpenClaw is strong for coding loops and single-session workflows.
- Hermes extends into “agent operations.”

Hermes differentiators:
- Persistent memory of user/environment
- Skill capture + patching loop
- Multi-platform gateway integrations
- Cron/webhooks/plugins/profiles
- Built-in dashboard and broader operations UX

Takeaway line:
- OpenClaw is excellent copilot behavior. Hermes targets copilot + runtime + operations.

---

## 4.5) Native Memory: How It Works (before self-improvement)
This section should appear before the self-improvement loop.

Built-in memory (default):
- `~/.hermes/memories/MEMORY.md` → environment/project facts
- `~/.hermes/memories/USER.md` → user profile/preferences

How injection works:
- These files are loaded into the system prompt as a frozen snapshot at session start.
- Mid-session writes persist immediately on disk but do not mutate the current prompt snapshot.
- Updates become active on the next session.

Why this design:
- Stable prompt prefix = better cache behavior and more predictable runs.
- Memory stays explicit and auditable (plain markdown files).

Important clarification:
- Built-in native memory is not default vector RAG.
- It's curated, file-backed memory with explicit writes.
- Vector/graph retrieval is optional via external memory providers.

---

## 5) Self-Improvement Loop (in-depth)
This is the core “why Hermes” section.

### 5.1 The Loop
1) Execute a real task with tools
- Hermes does actual work (commands, file edits, checks), not only text generation.
- Verification is part of completion: tests pass, outputs validated.

2) Extract reusable procedure
- If the task pattern is likely to recur, Hermes saves it as a skill.
- A good skill includes:
  - Trigger conditions (“use when X”)
  - Ordered steps
  - Common pitfalls
  - Verification checklist

3) Reuse on trigger
- On future tasks, Hermes loads the relevant skill and starts from a proven process.
- Outcome: fewer mistakes, less re-explaining, faster execution.

4) Patch when stale
- If reality changed (CLI flags, API behavior, OS quirks), patch the skill immediately.
- This prevents “knowledge rot” and keeps performance compounding.

### 5.2 Memory vs Skills (important distinction)
Memory stores facts:
- user preferences
- environment details
- stable conventions

Skills store procedures:
- repeatable workflows
- debugging playbooks
- setup/runbook logic

Simple line for audience:
- Memory is “what is true.” Skills are “how to do it.”

### 5.3 Why this compounds
Without this loop:
- Every session starts near zero.

With this loop:
- Session N+1 inherits a stronger operating baseline than session N.
- Errors become updates, not recurring failures.

### 5.4 Quality control in the loop
Compounding only works with discipline:
- Save skills after real success, not speculation.
- Include verification in every skill.
- Patch quickly when a skill fails.
- Keep memory compact and stable (no task junk).

Good one-liner:
- “Hermes improves by operationalizing lessons, not by hoping the model remembers.”

### 5.5 Prompt → Result → Improvement (decision flow)
Direct answer to the audience question: yes, improvement is mostly custom skills + targeted memory updates.

Flow:
1) Prompt
- User request includes goal + constraints + quality bar.

2) Result
- Hermes executes with tools and verifies output.

3) Improvement gate
- Ask three checks: reusable? stable? verified?
- If any check fails, do not write durable learning.

4) Write learning (only if gate passes)
- Write/patch a custom skill for repeatable procedure.
- Write memory only for stable facts (preferences, environment, conventions).

What should NOT be written:
- One-off outputs with no repeat value.
- Unverified guesses or temporary hacks.
- Temporary task progress (that belongs in session history, not memory).

---

## 6) Parallel Sessions and Multi-Agent Execution
Two parallel modes:
- Lightweight: delegated subtasks (quick and isolated)
- Heavyweight: independent Hermes processes (long-running)

Typical coding split:
- Agent A: backend
- Agent B: frontend
- Agent C: tests/CI
- Merge validated outputs

Why worktree mode matters:
- Parallel coding agents can conflict in git.
- Isolated worktrees reduce branch collisions and broken state.

---

## 7) Coding Workflow vs Codex/Claude Code Sessions
Common ground:
- Terminal-centric
- Edit/test/iterate loop
- Tool-assisted coding

Hermes-specific layer:
- Toolset controls and approval gates
- Session branching and recovery
- Skills + memory persistence
- Cross-platform continuity (same agent reachable in messaging)

Framing:
- Codex/Claude Code style = strong execution sessions.
- Hermes = execution sessions + durable operational memory + orchestration.

---

## 8) Knowledgebase by Default (Files + DB context assembly)
Explain this concretely:

System prompt context layers:
1. Identity file
- `~/.hermes/SOUL.md`

2. Project context file (priority, first match wins)
- `.hermes.md` (or `HERMES.md`) from cwd up to git root
- else `AGENTS.md`
- else `CLAUDE.md`
- else `.cursorrules` / `.cursor/rules/*.mdc`

3. Native memory snapshot
- `~/.hermes/memories/MEMORY.md`
- `~/.hermes/memories/USER.md`

4. Skills index
- Built from `~/.hermes/skills/` (plus optional external skill dirs)

5. Session recall path (on demand, tool-driven)
- `~/.hermes/state.db` (SQLite) stores session/chat history and powers FTS5 recall for `session_search`

Direct answer on “RAG?”
- Default is file-first prompt assembly + SQLite full-text recall.
- Not always-on vector RAG by default.
- Vector/semantic retrieval is optional via external memory plugins.

---

## 9) Obsidian Integration (what we built)
Practical setup:
- Use `~/wiki` as the working knowledgebase path.
- Keep it markdown-native so both humans and agent can operate on it.
- Sync with Obsidian (`ob sync --path ~/wiki`).

Recommended architecture:
- Hermes curates and updates wiki pages.
- Obsidian provides graph/backlinks/human editing UX.
- Automation keeps state current with low-noise no-op runs.

Special pattern from our build:
- Mirror local Hermes profile snapshots into wiki notes.
- Sync only on actual change.
- Optional watcher + cron fallback for reliability.

---

## 10) DIY Blueprint (replication checklist)
1. Install Hermes and choose provider/model
2. Create wiki structure (schema/index/log)
3. Connect Obsidian or obsidian-headless sync
4. Add scheduled ingest/lint/sync tasks
5. Add local profile mirror automation
6. Enforce silent no-op behavior

Closing line:
- “We moved from prompt interactions to a compounding agent system.”

---

## Optional Q&A bullets
Q: Is this just prompt engineering with extra steps?
A: No. The differentiator is executable workflows + persistent procedural capture + patching.

Q: Does self-improvement mean model fine-tuning?
A: Not required. Most gains come from better process memory (skills) and stable context memory.

Q: What fails most often?
A: Stale skills and weak verification. Patch quickly and keep verification explicit.

Q: Why Obsidian?
A: It keeps the knowledgebase transparent, editable, linked, and human-auditable while still machine-operable.
