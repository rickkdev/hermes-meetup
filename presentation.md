# Hermes Meetup Presentation Notes

## 1) Self-Sovereign AI (start here)
Definition in plain language:
- Self-sovereign AI means you control where models run, where data is stored, and who can access logs.

What “private” can mean in practice:
- Self-hosted on your own hardware (Mac Mini, Windows PC, even an old spare machine)
- Private VPS in a jurisdiction you trust (example: Germany)
- Privacy-first provider setup on infrastructure you control/trust (example: Venice AI)

Why this matters:
- Lower risk of accidental data leaks to third-party logs
- Better compliance and auditability
- Less vendor lock-in when policy/security requirements change

Talk track opener:
- This meetup is about agent capability, but it starts with control.
- If your data path is not under your control, your AI stack is fragile.

---

## 2) Opening
Hermes Agent is not just a coding assistant. It is an agent runtime that executes with tools, keeps continuity through memory, improves with skills, and runs across interfaces (CLI + messaging + automation).

Talk track:
- Most AI demos stop at “single conversation intelligence.”
- Hermes is built for operational intelligence across sessions.
- Goal of this talk: explain setup, architecture decisions, self-improvement loop, parallel execution, and knowledgebase integration with Obsidian.

---

## 3) What Hermes Actually Is
Core framing:
- Not a plain chatbot
- Not a single-provider wrapper
- Not stateless between runs

What it is:
- Tool-calling agent runtime
- Webhook support for event-driven automation (most important runtime capability)
- Multi-provider model layer
- Persistent memory + reusable skills
- Multi-surface interface (terminal, Telegram, Discord, etc.)
- Automation hooks (cron, webhooks)

Why this matters:
- Webhooks let Hermes react to real events (deploys, incidents, form submissions, tickets) instead of waiting for manual prompts.
- You can move from ad hoc prompts to repeatable systems.

---

## 4) Setup in 3 Minutes (practical)
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

## 5) Hermes / OpenClaw Blueprints
Replace the prior text comparison slide with two visual slides in the same slot:
- Slide 1: show `hermes-agent-blueprint.png` full-screen.
- Slide 2: show `open claw.png` full-screen.

Speaker approach:
- Keep narration light and let the diagrams do the comparison.
- First orient the room on the Hermes blueprint, then advance and contrast it with OpenClaw.

---

## 5.5) Native Memory: How It Works (before self-improvement)
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

### 5.5.1) Exactly when embeddings are used (and when they are not)
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

## 5.6) RAG vs No-RAG (when to use vector extensions)
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

## 5.7) What Is LLM Wiki + Obsidian?
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
- In "Works as a two-layer system", show the Obsidian second-brain image (`obsidian-secondbrain.jpg`) to make the human layer concrete.
- Show the Karpathy tweet screenshot (`karpathy-llm-wiki-tweet.png`) as origin/inspiration.

---

## 5.8) Obsidian Integration (what we built)
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

## 5.9) How To Do This Yourself (with copy/paste prompts)
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

### 5.9.1 Step 2 — Initialize wiki with `llm-wiki` (not “skeleton”)
What this means:
- Ask Hermes to set up the wiki at `~/wiki` using the LLM Wiki convention.
- That includes `SCHEMA.md`, `index.md`, `log.md`, and standard folders.

Prompt to Hermes:
- “Use the `llm-wiki` skill. Initialize a new wiki at `~/wiki` for Hermes operations. Create `SCHEMA.md`, `index.md`, `log.md`, and folders `raw/`, `entities/`, `concepts/`, `comparisons/`, `queries/`. Then confirm files created.”

If someone asks “do I install from a link?”
- “No in most cases — this skill is built-in. Just call it in the prompt.”

### 5.9.2 Step 3 — Connect Obsidian with `obsidian` skill
What this means:
- `llm-wiki` manages the knowledge workflow.
- `obsidian` connects and operates the vault UX/sync side.

Prompt to Hermes:
- “Use the `obsidian` skill. Set vault path to `~/wiki`, run Obsidian Sync setup with `ob`, verify status, and run one sync.”

If you want both in one request:
- “Use `llm-wiki` + `obsidian`: keep knowledge curation in `~/wiki` and keep Obsidian Sync connected.”

Edge case if a skill is missing:
- “Run `/skills search <skill-name>`. If it appears, install with `/skills install <identifier>`. If not, update Hermes.”

### 5.9.3 Step 4 — Scheduled ingest/lint/sync (what maintenance actually is)
What this means:
- Ingest = add/update knowledge from new sources.
- Lint = quality checks (broken links, missing metadata, stale pages).
- Sync = publish latest vault state to Obsidian Sync.

Simple explanation for non-technical audience:
- “Scheduled maintenance” means Hermes auto-runs routine housekeeping on a timer.
- It’s a robot janitor+librarian: bring in new info, clean bad links/metadata, then sync.
- Why it exists: no manual babysitting, less drift, always-current wiki.

Prompt to Hermes:
- “Create recurring jobs for `~/wiki`: ingest every 2 hours, lint daily at 09:00, and `ob sync --path ~/wiki` every 30 minutes. Keep output concise and include job IDs.”

### 5.9.4 Step 5 — Profile mirror automation (what gets mirrored)
What this means:
- Keep human-readable snapshots of core Hermes profile files inside the wiki.
- Typical mapping:
  - `~/.hermes/SOUL.md` → `~/wiki/hermes/soul.md`
  - `~/.hermes/memories/USER.md` → `~/wiki/hermes/user-profile-live.md`

Simple explanation for non-technical audience:
- “Snapshot mirror” means Hermes automatically copies important internal files into wiki notes.
- Think: camera snapshots of Hermes brain/config files, saved where humans can read them.
- Why it exists: transparency and audit trail (“what changed and when”) in Obsidian.

Prompt to Hermes:
- “Use `mirror-hermes-local-to-obsidian`. Set up automatic mirroring from `~/.hermes/SOUL.md` and `~/.hermes/memories/USER.md` into `~/wiki/hermes/` snapshot notes, updating only the `## current contents` section.”

### 5.9.5 Step 6 — No-op silent runs (how it works)
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

## 6) Self-Improvement Loop (in-depth)
This is the core “why Hermes” section.

### 6.1 The Loop
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

### 6.2 Memory vs Skills (important distinction)
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

### 6.3 Why this compounds
Without this loop:
- Every session starts near zero.

With this loop:
- Session N+1 inherits a stronger operating baseline than session N.
- Errors become updates, not recurring failures.

### 6.4 Quality control in the loop
Compounding only works with discipline:
- Save skills after real success, not speculation.
- Include verification in every skill.
- Patch quickly when a skill fails.
- Keep memory compact and stable (no task junk).

Good one-liner:
- “Hermes improves by operationalizing lessons, not by hoping the model remembers.”

### 6.5 Prompt → Result → Improvement (decision flow)
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

Concrete custom-skill example (use on slide):
- Example skill: `x-bookmarks-export-to-wiki` (or equivalent custom formatter skill)
- Prompt: "Get my X/Twitter bookmarks, fix inconsistent fields, and output them in the exact format I prefer."
- What happened in the run:
  1. Hermes retrieved/imported the bookmark export
  2. Detected inconsistent source fields (missing/variant metadata)
  3. Normalized records into a consistent schema
  4. Rendered output in the user’s preferred structure
- What got saved as a skill:
  - repeatable normalization rules
  - target output template
  - validation checks before final output
- Why this is a skill (not memory): it is a reusable transformation workflow, not a single fact.

What should NOT be written:
- One-off outputs with no repeat value.
- Unverified guesses or temporary hacks.
- Temporary task progress (that belongs in session history, not memory).

---

## 6.9) How a Session Works (bridge before parallel)
Use this as the transition slide before “Parallel Sessions”.

Core definition:
- A session is one live short-term working context (one “whiteboard”).
- The token meter (example: `45K / 400K`) shows current load vs context capacity.

How it behaves:
- Every turn adds to the live context window.
- Near threshold, Hermes compresses older context instead of hard-failing.
- Recent turns are protected; older turns are summarized.

What resets the live session:
- `/new` or `/clear`
- closing/reopening CLI and starting a new chat
- (unless explicitly resumed)

What persists beyond session reset:
- history in `~/.hermes/state.db`
- durable memory facts (`MEMORY.md` / `USER.md`)
- skills
- filesystem changes

What Telegram adds (important):
- Telegram is handled by the gateway service, not your current CLI tab.
- So closing one CLI session does not stop Telegram handling if gateway is still running.
- Telegram has its own live session context(s), but writes to the same long-term stores (`state.db`, memory, skills).
- If the host machine/WSL is down, gateway is down too (unless you run it on a VPS/server).

Bridge line into next slide:
- “Parallel sessions are separate live whiteboards that share the same long-term notebook.”

---

## 7) Subagent Spawn Mechanism (deep dive)
This replaces the old “How it actually works” parallel overview.

Core definition:
- A subagent is an isolated short-lived worker spawned via `delegate_task`.
- The parent sends a scoped packet: `goal` + `context` + `toolsets` (+ limits).
- The child runs in fresh context and returns a final summary (not full transcript).

### 7.1 The actual mechanism (plain engineering view)
Yes — your intuition is right. There is no separate magical scheduler function with hidden planning logic.

What actually happens inside the normal agent loop:
1) Parent session receives your prompt.
2) Parent calls the main LLM with:
   - current conversation context
   - system/developer instructions
   - available tool schemas (including `delegate_task`)
3) The LLM chooses one of two actions:
   - continue in the parent session, or
   - emit a tool call to `delegate_task`.
4) If it emits `delegate_task`, Hermes runtime executes that tool call:
   - creates one or more child agents in isolated contexts
   - passes explicit payload (`goal`, `context`, `toolsets`, limits)
   - each child runs independently and returns summary output only
5) Parent receives child summaries as tool results, then continues reasoning:
   - merge findings
   - choose final action
   - run verification/tools
   - return final answer

Important: the “decomposition” is mostly the model deciding to call `delegate_task` with specific child tasks. Hermes runtime then enforces guardrails (iteration limits, spawn depth, allowed toolsets, concurrency limits).

Practical line:
- “Subagent spawning is an LLM tool-call decision plus runtime guardrails: decide → dispatch → summarize → merge → verify.”

### 7.2 Concrete example (bugfix workflow)
Example prompt to parent:
- “Fix flaky login tests. Use subagents where helpful.”

What the parent LLM may emit (conceptually):
- `delegate_task(tasks=[{goal: "trace failing test path"}, {goal: "inspect auth middleware"}, {goal: "review recent auth commits"}])`

This is the key point:
- The parent does not manually fork OS processes itself in your prompt.
- It asks the Hermes runtime to execute the `delegate_task` tool with structured arguments.
- Runtime then spawns isolated child agent runs and returns their summaries to the parent.

What parent spawns:
- Child A: trace failing test path + stack traces
- Child B: inspect auth middleware/session handling
- Child C: review recent commits touching login/auth

What comes back:
- A: failure trigger location
- B: likely state/race bug in middleware
- C: suspect regression commit range

What parent does next:
- chooses fix strategy
- applies patch
- runs full test suite
- returns one merged explanation + verification result

### 7.3 When NOT to spawn
- one linear tool call is faster in parent
- tasks requiring constant shared in-memory state
- high-frequency user clarification loops

### 7.4 Parallel terminals reminder
- Multiple terminal windows are separate live sessions.
- Same profile shares long-term state (DB/memory/skills/files), not live turn-by-turn thoughts.
- Use worktrees when concurrent workers touch same repo.

---

## 8) Coding Workflow: the context-bloat objection (start here)
Main concern to lead with:
- “If I code in Hermes continuously, won’t context bloat beat me vs starting fresh Claude Code loops per story?”

Short honest answer:
- Yes, bloat is real if you keep one giant thread forever.
- No, if you operate Hermes correctly: fresh execution scopes + selective persistence.

### 8.1 Fresh-loop model vs Hermes model
Fresh-loop model (Claude/Codex style per story):
- Pro: very clean context window per task.
- Con: repeated onboarding every single time (repo conventions, test commands, architecture constraints).

Hermes model:
- Keep execution sessions scoped (don’t run one mega-session forever).
- Persist only durable value (skills + memory), not transient chatter.
- Result: less repetitive setup while keeping context quality high.

How to reset scope in Hermes (say this explicitly):
- End the current story with a concise summary.
- Start a new Hermes session (new terminal/tab and run Hermes again).
- Paste only a compact handoff brief (goal, constraints, changed files, open risks).

Stage one-liner:
- “Hermes done right is not one infinite context; it’s many fresh contexts plus a durable brain.”

### 8.2 When fresh loops are genuinely better
Use fresh loops when:
- tiny one-off changes with no continuity value
- throwaway experiments
- quick probes in unknown codebases where setup knowledge won’t be reused

Reset-scope version inside Hermes:
- Treat each one-off as a separate Hermes story session anyway.
- Do not continue from yesterday’s unrelated coding thread just because it exists.

### 8.3 When Hermes is better
Use Hermes when:
- project standards repeat (tests, lint flow, release steps)
- similar bugs/feature patterns recur
- you want the agent to stop relearning your stack every day

Reset-scope rule here too:
- Reuse durable project knowledge, but still start each story in a fresh Hermes session.

### 8.4 Anti-bloat operating pattern (practical)
1) Run one session per story or milestone.
2) End with a clean summary (what changed, what remains, risks).
3) Save stable facts to memory (preferences, environment truths).
4) Save repeatable procedures as skills (not ad-hoc chat logs).
5) Start the next story fresh, loading only what matters.

5-line handoff brief when you reset scope:
- Story goal
- Constraints / acceptance criteria
- Files touched or areas in scope
- Current status (done vs pending)
- Known risks/questions

This gives you:
- fresh local context quality
- less repeated prompting
- better long-term throughput

### 8.5 Prompt patterns to teach audience
Pattern A (scoped story):
- “Treat this as one isolated story. Do not drag prior irrelevant context. Complete implementation + tests + concise wrap-up.”

Pattern B (capture only durable value):
- “After finishing, save only durable lessons: one memory entry for stable project facts and one skill patch for repeatable workflow improvements.”

Pattern C (parallel for complex stories):
- “Split this story into backend/frontend/tests workers, keep contexts isolated, then merge outputs with integration risks and final checklist.”

Pattern D (explicit scope reset command to self):
- “Close this story now. Produce a 5-line handoff brief. I will start a fresh Hermes session for the next story.”

### 8.6 Bottom-line framing
- Fresh Claude loop every time optimizes for local cleanliness.
- Hermes optimizes for cleanliness plus cumulative operational learning.
- The control knob is scope resets: one story, one session.
- If you treat Hermes like a never-ending chat, you lose.
- If you treat it like scoped runs + curated persistence, you win.

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
