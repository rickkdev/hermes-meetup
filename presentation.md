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

## 5.8) How To Do This Yourself (with copy/paste prompts)
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

### 5.8.1 Step 2 — Initialize wiki with `llm-wiki` (not “skeleton”)
What this means:
- Ask Hermes to set up the wiki at `~/wiki` using the LLM Wiki convention.
- That includes `SCHEMA.md`, `index.md`, `log.md`, and standard folders.

Prompt to Hermes:
- “Use the `llm-wiki` skill. Initialize a new wiki at `~/wiki` for Hermes operations. Create `SCHEMA.md`, `index.md`, `log.md`, and folders `raw/`, `entities/`, `concepts/`, `comparisons/`, `queries/`. Then confirm files created.”

If someone asks “do I install from a link?”
- “No in most cases — this skill is built-in. Just call it in the prompt.”

### 5.8.2 Step 3 — Connect Obsidian with `obsidian` skill
What this means:
- `llm-wiki` manages the knowledge workflow.
- `obsidian` connects and operates the vault UX/sync side.

Prompt to Hermes:
- “Use the `obsidian` skill. Set vault path to `~/wiki`, run Obsidian Sync setup with `ob`, verify status, and run one sync.”

If you want both in one request:
- “Use `llm-wiki` + `obsidian`: keep knowledge curation in `~/wiki` and keep Obsidian Sync connected.”

Edge case if a skill is missing:
- “Run `/skills search <skill-name>`. If it appears, install with `/skills install <identifier>`. If not, update Hermes.”

### 5.8.3 Step 4 — Scheduled ingest/lint/sync (what maintenance actually is)
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

### 5.8.4 Step 5 — Profile mirror automation (what gets mirrored)
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

### 5.8.5 Step 6 — No-op silent runs (how it works)
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

## 7) Subagent Spawn Mechanism (diagram)
Use the full-screen image: `how-subagent-works.png`.

Talk track:
- Parent LLM sees available tools and may emit a `delegate_task` tool call.
- Hermes runtime executes that call with limits/guardrails.
- Child agent(s) run isolated and return summaries.
- Parent merges outputs and verifies final result.

---

## 8) Ralph Loop (what it is)
Use this right after the subagent diagram.

Slide layout:
- Left side: explain Ralph loop mechanism.
- Right side: photo of Geoffrey Huntley (`ghuntley.jpg`) + links.

Definition in your terms:
- A Ralph loop is a scripted fresh-session coding loop.
- Geoffrey Huntley website: https://ghuntley.com/
- Ralph tool: https://github.com/rickkdev/ralph
- Inputs are usually:
  - `PRD.json` (project/stories)
  - one or more agent instruction `.md` files (coding principles/rules)
  - `progress.txt` (story progress ledger)

Execution pattern:
1) Launcher opens a fresh Codex/Claude session.
2) Agent reads rules + PRD + progress.
3) Agent picks next unfinished story.
4) Agent implements + verifies.
5) Agent writes status back to `progress.txt`.
6) Session closes.
7) Script runs again for next story.

Why teams like it:
- predictable progress tracking
- clean context every story
- easy to run long multi-story builds with less drift

Bridge line:
- “Ralph loop externalizes planning files; Hermes can do similar decomposition internally via subagents.”

---

## 9) Coding Workflow Choices (expanded comparison)
Use this section for audiences already using Claude Code, Codex, Open Code, or Cursor and asking where Hermes fits.

### 9.1 One-liner positioning
- Claude Code / Codex / Open Code are coding specialists inside a repo.
- Cursor is an in-editor pair programmer.
- Ralph loops are for bigger one-shot epic runs and overnight execution.
- Hermes is a self-improving autonomous agent that lives outside all of them.
- Most power users run 2-3 together, not one.

### 9.2 Comparison at a glance
| Feature | Claude Code / Codex / Open Code | Cursor | Ralph loops | Hermes Agent |
|---|---|---|---|---|
| Primary surface | CLI in a repo | IDE | Loop scripts + terminal | CLI + chat + cron + Telegram |
| Persistent memory | Mostly session-scoped | Project/editor-scoped | File/process-driven | Cross-session, bounded (`MEMORY.md`, `USER.md`) |
| Learning | None | None | None | Auto-generates skills after repeated patterns |
| Channels | Terminal | Editor | Terminal / scheduled runs | Terminal, Telegram, Discord, email, webhooks |
| Scheduled jobs | No | No | Yes (overnight loops) | Built-in cron |
| Self-improvement | No | No | No | Yes (skills + memory loop) |
| Best for | Read/edit/test/commit loops | Writing/fixing code inline | Bigger epics + overnight task runs | Long-running autonomous tasks that compound |

### 9.3 When to pick which
- Pick Claude Code/ Codex/ Open Code when you're inside a repo and want the agent to read code, edit code, run tests, commit. It's a coding specialist.
- Pick Cursor when you want AI completions and fixes inside your editor in real time.
- Pick Ralph loops for bigger epics you want to one shot and generall tasks you want to run over night.
- Pick Hermes when you want an agent that (a) lives beyond any single session, (b) talks to you from any channel, (c) gets better at your recurring work over time.

### 9.4 Practical stack recommendation
- Repo-heavy dev loop: Claude Code/Codex/Open Code + Hermes
- IDE-heavy daily coding: Cursor + Hermes
- Big epic overnight execution: Ralph loops (+ Hermes for memory + multi-channel control)
- For compounding leverage over weeks/months, Hermes should be one of the layers.

## 9.5) Team Profiles: Shared Hermes Agent
Core question:
- “Can a team share one Hermes agent?”
- Yes, via profiles.

Simple explanation:
- A profile is an isolated Hermes instance.
- Each profile has its own:
  - memory (`MEMORY.md`, `USER.md`)
  - sessions/history
  - skills
  - cron jobs/automation state
  - config/auth scope

How to run this for a team (single slide flow):
1. Create a shared profile on a shared VPS (example: `team-core`).
2. Run Hermes gateway for that profile.
3. Pair/allowlist team members in the messaging gateway.
4. Everyone messages the same team agent endpoint.
5. All outcomes accumulate in shared profile memory + sessions.

Why it matters:
- You get one team “agent brain” without mixing with personal agents.
- Team automation and memory compound in one place.
- Personal profile remains private and separate.

Practical operating pattern:
- Keep two profiles:
  - `personal` for private work
  - `team-core` for shared team operations

---

## 10) Next Section: Hermes + Webhooks
Purpose: transition from coding workflow into event-driven agent operations.

Suggested slide copy:
- “Hermes + Webhooks: unlocking real agentic workflow potential.”
- “Stop babysitting your agent — let the world trigger it.”

Talking points:
- Shift framing from chat-first to trigger-first automation.
- External systems should initiate work: deploy events, incident alerts, tickets, form submissions, API callbacks.
- This is where agent workflows stop being demos and become operational systems.

## 10.1) Hermes Integrations: Webhook Triggers
Goal: show concrete apps that can trigger Hermes with webhook events.

Core idea:
- Each integration is basically a webhook source.
- Event happens in external tool -> Hermes receives POST -> agent run starts.

Suggested integrations to mention:
- Typeform: new response -> qualify lead, route to owner, draft reply.
- ConvertKit (or Beehiiv): new subscriber/campaign event -> tag/enrich/onboarding actions.
- Stripe: payment/subscription/invoice event -> CRM update, receipt/support workflow.
- GitHub: push/PR/issue event -> summarize diff, triage, create follow-up tasks.
- CRM (Attio or HubSpot): lead/contact changes -> enrich, assign owner, trigger next follow-up.
- Extra compatible options: Linear, Jira, Notion, Slack (event-driven triggers).

Bridge note:
- If a tool has weak webhook support, bridge with Zapier or Make and still trigger Hermes.

## 10.2) Webhook Adapter: Hermes as Receiver
Goal: explain incoming webhook flow in plain language, including HMAC and payload rendering.

Important clarification:
- Step 01 is NOT Hermes sending a webhook.
- Step 01 is another service sending a webhook TO Hermes.

Core framing:
- Hermes webhook adapter is an HTTP gateway.
- Like Telegram/Slack adapters receive chat messages, this adapter receives event messages.

Flow (easy version):
1. External app sends POST to `/webhooks/<route-name>`
2. Hermes checks signature (HMAC) to verify sender authenticity
3. Hermes converts event JSON into a readable prompt via template
4. Hermes runs a fresh agent turn with that prompt as user input
5. Hermes sends output to a configured destination (chat/tool/API endpoint)

What is HMAC signature?
- HMAC is a cryptographic signature made with a shared secret.
- Sender signs the payload; Hermes recalculates and compares.
- If signatures mismatch, Hermes rejects request.
- Purpose: stop spoofed/fake webhook calls.

What does “render prompt from payload” mean?
- Payload = JSON body sent by the external service.
- Template = text with placeholders like `{payload.repository}` or `{payload.alert_title}`.
- Rendering = replacing placeholders with real values from the JSON.
- Result = a clean prompt the agent can reason on.

Concrete example:
- Incoming payload: `{ "repo": "hermes-agent", "branch": "main", "author": "rick" }`
- Template: `Summarize this push to {payload.repo} on {payload.branch} by {payload.author}.`
- Rendered prompt: `Summarize this push to hermes-agent on main by rick.`

Speaker line:
- “This is just message translation: event JSON in, trusted + templated, then treated as a user message for the agent.”

## 10.3) Webhook Adapter: Hermes as Sender
Goal: explain outbound posting from Hermes in plain terms.

Meaning of “as sender”:
- Hermes initiates HTTPS calls to your endpoints.
- This is outbound from Hermes.
- No inbound webhook is required for these sends.

Two practical mechanisms:
1. Cron jobs (scheduled)
2. Terminal tool (on-demand during agent run)

### 10.3.1 Scheduled agent posts (cron jobs)
How it works:
- A cron schedule triggers a job (every 30m, hourly, etc.).
- The job runs an agent prompt in a fresh session.
- Agent composes payload and sends to endpoint.
- Send method can be `curl` command or Python `httpx` call.

Example idea:
- Every hour: summarize system state and POST JSON to dashboard webhook.

### 10.3.2 Terminal tool explained (what it is)
- Terminal tool lets the agent run shell commands on the machine.
- Typical use: run scripts, call APIs with curl, execute checks, parse outputs.
- For webhook sending, agent can run:
  - `curl -X POST https://api.example.com/hook -H 'Content-Type: application/json' -d '{...}'`

### 10.3.3 What is HTTPX?
- HTTPX is a Python HTTP client library (like `requests`, but modern + async support).
- If you know curl: HTTPX is “curl-style HTTP calls in Python code”.
- Useful when payload creation/logic is easier in Python than in one long shell command.

Minimal Python example:
```python
import httpx
payload = {"status": "ok", "source": "hermes"}
httpx.post("https://api.example.com/hook", json=payload, timeout=10)
```

### 10.3.4 No inbound needed after start (runtime hooks)
Goal: explain how Hermes can keep posting updates during a run.

Plain-language translation of confusing terms:
- `session_start` = “a new run just began”
- `agent_step` = “one unit of agent work completed”
- `get_webhooks` = “load configured endpoints/targets”

How hook-based posting works:
1. A run starts (or a step finishes)
2. Hook code is triggered at that moment
3. Hook builds a payload (status, result, metadata)
4. Hermes POSTs to your endpoint
5. Your receiver processes update (CLI, gateway, internal API, etc.)

Important point:
- After the first trigger, Hermes can keep sending HTTPS updates itself.
- You do not need additional inbound events for each update.

Simple example:
- Session starts for "daily build monitor"
- Each agent step posts progress to `https://ops.example.com/agent-progress`
- Final step posts summary + success/failure to `https://ops.example.com/agent-final`

Speaker line:
- “Inbound starts the process; hooks make Hermes proactively report and integrate while it runs.”

## 10.4) Webhooks Setup (No-Code) + config.yaml explained
Goal: explain a beginner-safe webhook setup flow for Telegram-only users.

### 10.4.1 Beginner flow (no Python, minimal terminal)
1. Enable webhook adapter (`WEBHOOK_ENABLED=true`).
2. Set listener port (`WEBHOOK_PORT=8644`).
3. Set auth secret (`WEBHOOK_SECRET=...`) as global fallback.
4. Expose URL:
   - public VPS with HTTPS -> use domain directly, or
   - local/private host -> use ngrok tunnel.
5. Create route (`config.yaml` or `hermes webhook subscribe`).
6. Paste webhook URL in external app and test delivery.

### 10.4.2 What env vars mean
- `WEBHOOK_ENABLED=true`
  - turns on webhook HTTP server in Hermes gateway.
- `WEBHOOK_PORT=8644`
  - local port Hermes listens on for incoming POSTs.
- `WEBHOOK_SECRET=<strong-secret>`
  - global fallback secret for signature validation.
  - best practice: set per-route `secret` too.

### 10.4.3 Why security matters here
- Webhook endpoints are public URLs.
- Anyone can hit URL unless signature validation blocks them.
- If secret is missing or weak, fake events can trigger agent runs.
- Keep secrets local (`~/.hermes/.env`), never in chat logs.
- Rotate leaked secrets immediately.

### 10.4.4 What ngrok is (and when you need it)
- ngrok = temporary public HTTPS tunnel to local port.
- Example: `http://localhost:8644` -> `https://abc123.ngrok-free.app`
- Needed when Hermes is not publicly reachable.
- Not needed when VPS already has public HTTPS domain.
- Free ngrok URLs change on restart; update webhook URL when that happens.

### 10.4.5 How `config.yaml` route works (simple mental model)
One route = 5 blocks:
1. `events` (what to accept)
2. `secret` (auth check)
3. `prompt` (payload -> agent input)
4. `skills` (optional preload)
5. `deliver` + `deliver_extra` (where result goes)

Minimal route example:
```yaml
platforms:
  webhook:
    enabled: true
    extra:
      port: 8644
      routes:
        github-pr:
          events: ["pull_request"]
          secret: "set-a-strong-secret-here"
          prompt: |
            Review PR #{number} in {repository.full_name}
            Title: {pull_request.title}
            URL: {pull_request.html_url}
          skills: ["github-code-review"]
          deliver: "github_comment"
          deliver_extra:
            repo: "{repository.full_name}"
            pr_number: "{number}"
```

Template notes:
- `{field.path}` = insert value from webhook JSON payload.
- `{__raw__}` = dump full payload while discovering field names.

## 10.5) Telegram Chat vs VPS Tasks (who does what)
Goal: set clear boundary so non-technical users know what must be done manually.

Can be done just by texting Hermes (Telegram):
- choose service + event type + route behavior
- draft prompt templates and delivery mapping
- explain payload fields and fix template mistakes
- debug webhook failures from logs/errors you paste

Requires VPS login / host access:
- add real secrets to `~/.hermes/.env`
- restart gateway service
- make endpoint publicly reachable (TLS/domain or ngrok)
- firewall/reverse proxy only if you host endpoint directly on public VPS

Still manual outside Hermes:
- create webhook in external service UI (GitHub/Stripe/Typeform/etc.)
- paste Hermes endpoint + matching secret there

Speaker line:
- “Think chat for logic, VPS for infrastructure, external UI for registration.”

## 10.6) Webhook Setup Master Prompt: Public VPS
Goal: give a production-safe reference pattern.

Architecture chain:
- External service -> Cloudflare/WAF -> Nginx/Caddy (HTTPS 443) -> Hermes webhook adapter (`:8644`) -> HMAC verification -> agent run.

Key security rules:
- Do NOT expose `:8644` directly to internet.
- Keep `:8644` internal; only expose 443.
- Enforce per-route secrets + signature validation.
- Apply rate limiting at WAF/proxy + route-level limits in Hermes.

Master Prompt #1 (Public VPS):
```text
Set up production webhook mode on this VPS: enable webhook adapter, bind local port 8644, configure reverse-proxy/TLS on 443, keep 8644 internal-only, add per-route HMAC secrets, apply rate limits, verify /health and one signed test route, then output final webhook URL + security checklist.
```

## 10.7) Webhook Setup Master Prompt: Local/Private + ngrok
Goal: give a safe dev/testing pattern when host is not public.

Architecture chain:
- External service -> ngrok HTTPS URL -> local Hermes webhook adapter (`:8644`) -> HMAC verification -> agent run.

Key security rules:
- ngrok provides reachability, not authentication.
- Keep per-route secret required.
- Free ngrok URLs rotate; update webhook target on restart.

Master Prompt #2 (Local/Private):
```text
Set up secure local webhook mode with ngrok: enable webhook adapter on 8644, create one route with per-route HMAC secret, start ngrok tunnel to 8644, return public webhook URL, run signed test POST, and output what I must paste into external service webhook settings (URL, secret, event type).
```

## 10.8) Takeaways
Goal: end with one visual mental model, not a text wall.

Slide structure (3 cards):
- 01 — STOP POLLING
  - Webhooks are push events.
  - Polling = repeatedly asking “anything new yet?”
- 02 — TWO-WAY FLOW
  - Trigger comes in.
  - Hermes can also push updates/results out.
- 03 — UNIVERSAL CONNECTOR
  - Any webhook-capable tool can start work.
  - Any API/webhook endpoint can receive results.

Anchor line on slide:
- “trigger in → Hermes works → result out.”

Speaker line:
- “If you remember one thing: stop building ‘check every minute’ loops. Let events fire the workflow.”

## 10.9) Closing / Speaker Handoff
Slide copy:
- “This is the end for Hermes Agent.”
- “Next up: Georg and Roman on local AI setups, GPUs, and the best models to host locally at home.”

Speaker line:
- “Thanks everyone — handing over to Georg and Roman for the local AI stack deep dive.”

## 10.10) Sponsors
Goal: thank sponsors and give attendees a concrete next step.

Slide structure:
- Two cards: Nebius + Mem0
- Each card includes logo, website, offer, and QR code

Content:
- Nebius
  - Website: nebius.com
  - Offer: $50 promo code for attendees
- Mem0
  - Website: mem0.ai
  - Offer: join/signup via QR/link

Speaker line:
- “Huge thanks to Nebius and Mem0 for sponsoring this meetup — scan either QR now and claim the offer.”

## 10.11) Final CTA — Join our Local AI WhatsApp Group
Goal: convert audience attention into ongoing community participation.

Slide content:
- Invite everyone: beginners, builders, teams, founders, CEOs, employees.
- Say what they get in the group:
  - live Hermes Agent updates,
  - new tips & tricks,
  - setup videos for absolute beginners,
  - FAQ support (people ask, we answer),
  - latest model discussions (especially smaller home-runnable models),
  - hardware guidance (what to buy for LLMs, what to avoid, and why).
- Mention practical examples:
  - home GPU discussion around 3090 / 4090 / 5090,
  - small model picks like Qwen2.5 14B and Llama 3.1 8B.
- Strong CTA: “Scan now and join before you leave.”

Speaker line:
- “If you want the real follow-up after today, this WhatsApp group is where we share everything — updates, fixes, model picks, and buying advice.”

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
