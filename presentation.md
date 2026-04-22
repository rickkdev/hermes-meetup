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

Start with this explicit line for the audience:
- Yes, Hermes has a DB: `~/.hermes/state.db` (SQLite) for chat/session history.

Two memory lanes (default):
- Curated facts in files:
  - `~/.hermes/memories/MEMORY.md` → environment/project facts
  - `~/.hermes/memories/USER.md` → user profile/preferences
- Full conversation history in DB:
  - `~/.hermes/state.db` (SQLite) stores sessions + messages
  - FTS5 index powers `session_search` over past transcripts

When each lane is used:
- File memory lane:
  - loaded into prompt at session start (baseline context)
  - use for stable truths needed often
  - write via `memory` tool when fact should persist broadly
- DB history lane:
  - queried on demand via `session_search` (not always injected)
  - use for historical details tied to prior chats
  - best when user asks “what did we do before?” or references old work

Decision rule:
- Always-needed stable truth → file memory
- Historical details from old sessions → DB recall (`session_search`)

Why this design:
- Stable prompt prefix = better cache behavior and predictable runs.
- Knowledge remains auditable (plain files + inspectable SQLite DB).

Important clarification:
- Default is file snapshot + SQLite full-text recall.
- Not always-on vector RAG by default.
- Vector/graph retrieval is optional via external memory providers.

### 4.5.1) Exactly when embeddings are used (and when they are not)
This is the part people usually misunderstand.

By default (no external memory provider enabled):
- No embedding pipeline runs for native memory files.
- `MEMORY.md` and `USER.md` are plain text blocks injected at session start.
- `session_search` uses SQLite FTS5 keyword matching on `state.db`.
- So recall is lexical/full-text + LLM summarization, not vector similarity.

What happens when `session_search` runs (default path):
1. Query is matched against `messages_fts` in `~/.hermes/state.db` (FTS5).
2. Top matching sessions/messages are selected.
3. Relevant transcript windows are assembled.
4. An auxiliary model summarizes those sessions for the current task.
5. That summary is returned to the main agent as context.

This is retrieval, but not embedding-RAG.

When embeddings/semantic retrieval enter the system:
- Only after enabling an external memory provider plugin.
- Typical providers expose semantic/vector retrieval (for example Mem0, Honcho, Hindsight, Supermemory, etc.).
- In that mode, retrieval behavior is provider-specific (some mix semantic + keyword + graph/entity logic).

Practical trigger rule for the talk:
- "Out of the box: file memory + SQLite FTS recall."
- "Embeddings appear only when you explicitly turn on a semantic memory provider."

---

## 4.6) RAG vs No-RAG (when to use vector extensions)
Goal: give a non-technical decision framework for both individuals and employee workflows.

Simple framing:
- No-RAG is not “worse.” It is the simpler default and works great for many cases.
- Vector RAG is a scale/retrieval upgrade, not a mandatory baseline.

No-RAG default (Hermes out of the box):
- Uses file memory + SQLite FTS recall (`state.db`) + summarization.
- Benefits:
  - lower complexity/cost
  - easier to audit and explain
  - less infra maintenance
- Best for:
  - personal assistants
  - small teams with moderate docs
  - environments where explainability matters more than max semantic recall

RAG/vector extension:
- Adds semantic similarity retrieval (via external memory providers/plugins).
- Benefits:
  - better recall across large, messy, synonym-heavy corpora
  - stronger retrieval when exact keywords are unknown
- Costs:
  - extra setup/ops (indexing, tuning, monitoring)
  - more moving parts to debug
- Best for:
  - employee knowledge systems across many teams and documents
  - large policy/wiki/support corpora
  - recurring “we know this exists but nobody can find it quickly” complaints

Decision matrix (easy to say on stage):
- If your problem is “keep context stable and practical” → stay No-RAG.
- If your problem is “find the right answer in huge, scattered docs” → enable vector extension.

Non-technical one-liner:
- No-RAG = “organized notebook + search.”
- RAG = “semantic librarian for very large libraries.”

Rollout recommendation:
1. Start with No-RAG.
2. Measure misses (how often users fail to find known answers).
3. Enable vector extension only when misses become a business problem.

---

## 4.7) What Is LLM Wiki + Obsidian?
Simple framing:
- LLM Wiki is the agent-side knowledge workflow.
- Obsidian is the human-side interface on the same markdown files.

Explain as two layers:
- Layer 1 (`llm-wiki`): ingest sources, synthesize pages, enforce schema, maintain links, run lint.
- Layer 2 (`obsidian` + Sync): browse graph, backlinks, edit manually, keep devices in sync.

Why this combo works:
- Agent does heavy curation and consistency work.
- Humans keep transparency and editorial control.
- No lock-in database: plain markdown vault.

On this slide:
- Show the Karpathy tweet screenshot (`karpathy-llm-wiki-tweet.png`) as origin/inspiration.

---

## 4.8) Obsidian Integration (what we built)
Practical setup:
- Use `~/wiki` as the working knowledgebase path.
- Keep it markdown-native so both humans and agent can operate on it.
- Sync with Obsidian (`ob sync --path ~/wiki`).

Native Hermes support worth saying explicitly:
- Hermes has a built-in `obsidian` skill (read/search/create notes in vault).
- LLM Wiki markdown format is Obsidian-vault compatible out of the box (`[[wikilinks]]`, frontmatter, assets).
- Set `OBSIDIAN_VAULT_PATH=~/wiki` so skill operations and wiki automation point to the same place.

Recommended architecture:
- Hermes curates and updates wiki pages.
- Obsidian provides graph/backlinks/human editing UX.
- Automation keeps state current with low-noise no-op runs.

Special pattern from our build:
- Mirror local Hermes profile snapshots into wiki notes.
- Sync only on actual change.
- Optional watcher + cron fallback for reliability.

---

## 4.9) How To Do This Yourself (with copy/paste prompts)
1. Install Hermes and choose provider/model
2. Run `llm-wiki` to initialize `~/wiki` (schema/index/log)
3. Run `obsidian` to connect `~/wiki` to Obsidian Sync
4. Schedule upkeep jobs (ingest, lint, sync)
5. Automate profile snapshot mirroring into wiki notes
6. Enforce quiet no-change runs (`[SILENT]`)

Skill availability (important):
- `llm-wiki` and `obsidian` are official built-in Hermes skills (normally no external URL needed).
- Verify with `/skills` or `hermes skills list`.
- If missing, update Hermes first, then re-check.

### 4.9.1 Step 2 — Initialize wiki with `llm-wiki` (not “skeleton”)
What this means:
- Ask Hermes to set up the wiki at `~/wiki` using the LLM Wiki convention.
- That includes `SCHEMA.md`, `index.md`, `log.md`, and standard folders.

Prompt to Hermes:
- “Use the `llm-wiki` skill. Initialize a new wiki at `~/wiki` for Hermes operations. Create `SCHEMA.md`, `index.md`, `log.md`, and folders `raw/`, `entities/`, `concepts/`, `comparisons/`, `queries/`. Then confirm files created.”

If someone asks “do I install from a link?”
- “No in most cases — this skill is built-in. Just call it in the prompt.”

### 4.9.2 Step 3 — Connect Obsidian with `obsidian` skill
What this means:
- `llm-wiki` manages the knowledge workflow.
- `obsidian` connects and operates the vault UX/sync side.

Prompt to Hermes:
- “Use the `obsidian` skill. Set vault path to `~/wiki`, run Obsidian Sync setup with `ob`, verify status, and run one sync.”

If you want both in one request:
- “Use `llm-wiki` + `obsidian`: keep knowledge curation in `~/wiki` and keep Obsidian Sync connected.”

Edge case if a skill is missing:
- “Run `/skills search <skill-name>`. If it appears, install with `/skills install <identifier>`. If not, update Hermes.”

### 4.9.3 Step 4 — Scheduled ingest/lint/sync (what maintenance actually is)
What this means:
- Ingest = add/update knowledge from new sources.
- Lint = quality checks (broken links, missing metadata, stale pages).
- Sync = publish latest vault state to Obsidian Sync.

Prompt to Hermes:
- “Create recurring jobs for `~/wiki`: ingest every 2 hours, lint daily at 09:00, and `ob sync --path ~/wiki` every 30 minutes. Keep output concise and include job IDs.”

### 4.9.4 Step 5 — Profile mirror automation (what gets mirrored)
What this means:
- Keep human-readable snapshots of core Hermes profile files inside the wiki.
- Typical mapping:
  - `~/.hermes/SOUL.md` → `~/wiki/hermes/soul.md`
  - `~/.hermes/memories/USER.md` → `~/wiki/hermes/user-profile-live.md`

Prompt to Hermes:
- “Use `mirror-hermes-local-to-obsidian`. Set up automatic mirroring from `~/.hermes/SOUL.md` and `~/.hermes/memories/USER.md` into `~/wiki/hermes/` snapshot notes, updating only the `## current contents` section.”

### 4.9.5 Step 6 — No-op silent runs (how it works)
What this means:
- If a scheduled run detects no content change, it should not spam.
- Behavior on no-change:
  - skip log update
  - skip sync
  - return exactly `[SILENT]`

Prompt to Hermes:
- “For all wiki maintenance jobs, enforce no-op silence: if nothing changed, return exactly `[SILENT]`, do not append logs, and do not run sync.”

Transition line into next section:
- “Now that knowledge is connected, here’s how Hermes compounds capability over time.”

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

## Optional Q&A bullets
Q: Is this just prompt engineering with extra steps?
A: No. The differentiator is executable workflows + persistent procedural capture + patching.

Q: Does self-improvement mean model fine-tuning?
A: Not required. Most gains come from better process memory (skills) and stable context memory.

Q: What fails most often?
A: Stale skills and weak verification. Patch quickly and keep verification explicit.

Q: Why Obsidian?
A: It keeps the knowledgebase transparent, editable, linked, and human-auditable while still machine-operable.
