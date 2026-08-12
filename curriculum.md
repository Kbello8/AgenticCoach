# Agentic Engineering Curriculum — Course Reference

An 11-phase, self-paced curriculum for someone who already has real hands-on
experience building an AI agent system (even a rough one) and wants to formalize that
experience into interview-ready vocabulary, production-grade practices (evals,
observability, security), and comparative fluency across the major frameworks. This is
gap-filling and formalization, not ground-zero teaching — if you've never built
anything with an LLM before, this will move fast.

Primary tool bias: **Claude Code / Claude Agent SDK first.** Other frameworks
(LangGraph, CrewAI, OpenAI Agents SDK) are covered for interview fluency, not pushed as
a stack switch — adjust that framing if your own primary stack differs.

If you're running this via the paired `SKILL.md` coach, this file is the coach's
answer key / quiz bank — it should never dump raw content at you as a wall of text,
only use it to generate questions and verify your answers. Working through it solo
instead? Read it directly as a self-study reference.

Course/resource URLs were current as of August 2026 — this field moves fast, verify
links still resolve before relying on them.

Throughout, **"your own agent system"** is a placeholder for whatever real project you
bring — a side project, a work system, or a fresh one you build alongside this
curriculum specifically to have a concrete example.

---

## Phase 0 — Pattern vocabulary (no course, ~1 session)

**Goal:** name the patterns you're probably already using informally in your own agent
system, so you can cite them by name in interviews and in this curriculum's later
phases.

**Resource:** Anthropic engineering blog post "Building Effective Agents" (search
Anthropic's engineering/research blog directly — verify the current URL yourself
rather than trusting a stale link).

**Content to learn/quiz on (canonical pattern definitions):**
- **Prompt chaining** — decompose a task into sequential LLM calls, each step's output
  feeds the next. Fixed pipeline, not dynamic.
- **Routing** — classify input, dispatch to one of several specialized downstream
  prompts/agents. Single branch taken, not parallel.
- **Parallelization** — two flavors: *sectioning* (split a task into independent
  subtasks run concurrently) vs *voting* (run the same task N times, aggregate/vote on
  results for confidence).
- **Orchestrator-workers** — a central orchestrator LLM dynamically breaks down a task
  and delegates to worker LLMs, then synthesizes results. Unlike parallelization, the
  subtask breakdown isn't fixed in advance — the orchestrator decides it per input.
- **Evaluator-optimizer** — one LLM generates a response, a second LLM evaluates it
  against criteria and gives feedback, loop until acceptable.

**Quiz targets:**
- Can you map your own agent system's architecture onto 2+ of these pattern names
  unprompted?
- Can you articulate the difference between orchestrator-workers (dynamic breakdown)
  and plain parallelization/sectioning (fixed breakdown)? This is the most commonly
  confused pair.
- If your system has any kind of self-check or review step, can you explain WHY you'd
  want that (evaluator-optimizer) instead of trusting the first output — think about
  the cost of a false positive in your domain?

**Exit criteria:** you can name all 5 patterns from memory and correctly tag which
ones your own system implements, without looking at the code.

---

## Phase 1 — Prompt & Context Engineering

**Goal:** move from "prompts that work well enough" to explicit vocabulary and
deliberate technique for prompt design and token/context economy.

**Resources (real published material, not a formal certificate course — same tier of
authority as Phase 0's blog-post source):**
1. Anthropic's prompt engineering guide —
   https://docs.claude.com/en/docs/build-with-claude/prompt-engineering
   Covers: clear/direct instructions, multishot (few-shot) examples, chain-of-thought,
   XML tag structuring, system vs user prompt roles, prefilling, prompt chaining.
2. Anthropic engineering blog — "Effective context engineering for AI agents" (search
   Anthropic's engineering blog directly; verify the current URL yourself).
   Covers: context window as finite/precious resource, "context rot," just-in-time
   retrieval vs upfront loading, compaction, sub-agent context isolation, structured
   note-taking/memory.

**Content to quiz on:**

*Prompting technique:*
- Clear & direct instruction — why specificity reduces output variance.
- Multishot/few-shot — when examples outperform instructions alone.
- Chain-of-thought — explicit step-by-step reasoning before the answer; why it helps
  reasoning-heavy tasks specifically, and the cost tradeoff (more output tokens, more
  latency) that makes it wrong to apply everywhere.
- XML tag structuring — why tagging reduces ambiguity in multi-part prompts (Claude is
  specifically trained to respect tag boundaries).
- System prompt vs user turn — persona/role/constraints belong in system prompt, task
  specifics in the user turn.
- Prefilling — constraining output format by seeding the start of the assistant turn.

*Token/context efficiency:*
- "Context rot" — performance degrades as irrelevant/stale context accumulates in the
  window; distinct from simply hitting the token ceiling. Quiz: can you explain the
  difference unprompted?
- Just-in-time retrieval vs upfront-load-everything — fetch what's needed when needed.
  If you already do this informally (e.g. only reading the file sections you actually
  need instead of whole files), name that explicitly — you're already doing this
  instinctively, this phase just gives it a name.
- Compaction/summarization strategy for long-running context.
- Sub-agent context isolation — delegating work to keep the main thread's context
  clean.
- Prompt caching — reusing cached prefixes (system prompt, tool defs, static context)
  across calls to cut cost and latency.

**Quiz targets:**
- Context rot vs token-limit exhaustion — stated as two different failure modes,
  unprompted.
- Chain-of-thought's cost tradeoff — can you say when NOT to use it, not just when to?
- Can you map your OWN existing practices onto the formal vocabulary (just-in-time
  retrieval, sub-agent context isolation) without being told the mapping first?

**Practical task (mandatory):** audit one real prompt or system-prompt file you
already use and rewrite it applying at least 3 of the techniques above. Document
before/after and, for each change, whether it targets output quality or token cost
(not all techniques target both).

**Exit criteria:** you name the 6 prompting techniques and 5 context-efficiency
concepts from memory, correctly self-diagnose which your own prompts already use vs.
are missing, and complete the practical rewrite with documented before/after.

---

## Phase 2 — Claude Code / Agent SDK + MCP formal layer

**Goal:** close the gap between "used an AI coding tool heavily" and "can explain its
primitives and has built an MCP server, not just consumed one."

**Courses (Anthropic Academy, free, no account required):**
1. **Claude Code 101** — https://anthropic.skilljar.com
   Covers: installation (terminal/VS Code/JetBrains), the Explore-Plan-Code-Commit
   workflow, CLAUDE.md project memory, basic MCP integration.
2. **Claude Code in Action** — https://anthropic.skilljar.com
   Covers: applied workflows building on 101.
3. **Introduction to Model Context Protocol (MCP)** —
   https://anthropic.skilljar.com/introduction-to-model-context-protocol
   Covers: MCP architecture (host/client/server roles), the three primitives — tools,
   resources, prompts — and building a server with the Python SDK.

**Reference spec (not a course, but authoritative):**
https://modelcontextprotocol.io/specification/2025-11-25

**Content to quiz on:**
- Explore-Plan-Code-Commit workflow — what problem does explicit "Plan" mode solve
  vs. jumping straight to edits? (Answer shape: front-loads scope agreement, avoids
  wrong-direction work — same instinct Phase 10 drills for in general.)
- MCP's three primitives — tools (model-invoked actions), resources (app-controlled
  data the model can read), prompts (user-invoked templates). Common confusion:
  conflating "tools" and "resources" — tools DO things, resources ARE things. See the
  spec URL above for the primitive definitions if you want to double-check an answer.
- Host/client/server roles in MCP — host is the application (e.g. Claude Code), client
  is the connector instance per-server, server exposes the primitives. If you've only
  ever consumed MCP servers others built, this phase's practical task flips that.

**Practical task (mandatory, not optional):** build one minimal MCP server (Python SDK)
exposing at least one tool and one resource, and connect it to a real session. This
closes the "consumer only" gap directly.

**AGENTS.md standard (added 2026-08-12):** emerging cross-tool convention — a plain
markdown file (`AGENTS.md`) at repo root carrying project instructions for AI coding
agents, readable by Claude Code, Cursor, Copilot, Windsurf, and others, vs. a
tool-specific file (e.g. CLAUDE.md) which only one tool reads. Same content
(build/test commands, conventions, architecture notes), vendor-neutral format. Useful
answer if asked "how do you configure agent behavior across different tools" — some
teams maintain both, with the tool-specific file importing/pointing to AGENTS.md for
portability.

**Planning methodology (added 2026-08-12):** the Explore-Plan-Code-Commit workflow
above names the loop; this fills in what "Plan" should actually contain for
non-trivial features:
- **PRD writing** — a short product-requirements doc (problem, scope, acceptance
  criteria) written BEFORE entering Plan mode, so the plan is scoped against a fixed
  target instead of drifting.
- **Feature decomposition** — breaking a large feature into context-window-sized
  chunks before implementation starts, so no single agent turn has to hold the whole
  feature's scope at once. Direct extension of Phase 1's context-rot material.
- **Tracer-bullet validation** — build the thinnest possible end-to-end slice through a
  new architecture first (not a full vertical feature) to prove the approach connects
  before investing in the full build. Ties to Phase 10's "compute the target before
  iterating" habit — a tracer bullet IS that instinct applied to architecture
  validation specifically.

**Quiz target (added):** can you explain why a tracer bullet isn't the same as an MVP
(it's allowed to be throwaway/ugly — its only job is proving connectivity, not
shipping value)?

**Test-loop discipline (added 2026-08-12):**
- **Red-green-refactor with agents** — same TDD cycle, but the agent writes the
  failing test first, confirms it fails for the right reason, then implements until
  green, before refactoring. Prevents the agent from writing tests that trivially pass
  against whatever it just built.
- **CI pipeline integration** — agent-authored changes should run against the same CI
  gate a human PR would, not be trusted on the agent's self-report alone.
- **Pre-commit hooks as a safety mechanism** — catch bad output mechanically before it
  lands, don't rely on the agent (or reviewer) noticing. Same instinct as Phase 6's
  observability hooks, applied pre-commit instead of pre-tool-call.

**Autonomous agent loops & human-in-the-loop design (added 2026-08-12):**
- **Unattended agent loops** — an agent loop that runs against a task queue (e.g.
  GitHub Issues) for extended periods, self-correcting via its own test/CI feedback
  rather than a human reviewing each step. Distinct from interactive sessions.
- **HITL (human-in-the-loop) operational modes** — deliberate design of WHERE a human
  must approve before the agent proceeds (destructive actions, external sends,
  irreversible ops) vs. where the agent may proceed autonomously. Ties directly to
  Phase 8's least-privilege content: HITL gates are the runtime enforcement of a
  privilege boundary, not just a UX choice.
- **AFK autonomous deployment** — running an agent loop unattended for hours, which
  raises the stakes on both test-loop discipline (above) and HITL gating — an AFK loop
  with no gates and no CI is the failure mode to name explicitly if asked "what could
  go wrong."

**Quiz target (added):** can you state, unprompted, why an unattended agent loop is
unsafe WITHOUT both a CI/test gate and defined HITL boundaries — not just "unsafe," but
naming which of the two failure modes (bad code lands unattended vs. agent takes an
irreversible action unattended) each safeguard actually prevents?

**Exit criteria:** courses marked complete by self-report + working MCP server you can
demo (describe what it does, run/test it live). Planning methodology and
test-loop/autonomous-loop content above are quizzed as part of this phase's exit but
don't require a separate practical artifact — they inform HOW this phase's (and later
phases') practical tasks should be approached.

---

## Phase 3 — Agent Skills

**Goal:** name a pattern you may already rely on without formal vocabulary for it. If
you use Claude Code, your own environment's skill listing is a live example — most
learners have been consuming Skills constantly without a term for what they are or how
they differ from a system prompt or an MCP tool.

**Resources:**
1. Anthropic engineering blog — "Equipping agents for the real world with Agent
   Skills" (published ~Jun 2026 — verify the current URL yourself).
2. Google Cloud blog — "Level Up Your Agents: Announcing Google's Official Skills
   Repository" (Jun 2026) — confirms this is becoming a cross-vendor pattern, not
   Claude-only; good interview point.

**Content to quiz on:**
- What a Skill is: a folder/package of instructions (+ optional scripts/resources),
  loaded on-demand when relevant, not always resident in context — distinct from a
  system prompt (always loaded) or a single MCP tool (one callable action).
- Skill vs MCP tool: a Skill is "how to do X" (procedural instructions the model reads
  and follows, often referencing tools it should call and how); an MCP tool is "do X"
  (one executable action). Skills frequently wrap or orchestrate MCP tools rather than
  replacing them.
- Why Skills matter for token economy: only loaded when the task triggers them, so they
  don't bloat every system prompt the way a monolithic instructions file would — the
  same just-in-time-retrieval instinct from Phase 1, applied to capability loading
  instead of data retrieval.
- **Progressive disclosure (added 2026-08-12):** the design principle behind a
  well-authored Skill — the top-level file stays short (name + one-line trigger
  description), with detailed instructions/scripts/references loaded only once the
  skill actually triggers, and even then only the sections needed. A Skill with
  everything front-loaded into one giant always-read file defeats its own purpose.
  This is the authoring-side mirror of the token-economy point above — that point
  explains why Skills as a MECHANISM save tokens; this one explains how to author an
  individual Skill so it keeps saving them as it grows (nested reference docs, not one
  flat file).

**Quiz targets:**
- Can you name 2+ concrete differences between a Skill and a bare system prompt, and
  between a Skill and an MCP tool, unprompted?
- Can you point to a real Skill you use and describe its frontmatter/instructions
  shape from memory (not by opening the file)?

**Practical task (mandatory):** author one new Skill from scratch (proper frontmatter +
instructions) — either formalizing an existing informal habit into one, or building
one that plugs an actual gap in your own workflow. Demo it triggering correctly.

**Exit criteria:** you state the Skill vs system-prompt vs MCP-tool distinction from
memory, and have one working authored Skill to show.

---

## Phase 4 — RAG & Retrieval / Vector Databases

**Goal:** most learners have heard the term RAG without a working definition or any
hands-on build. Close that gap. If your own system already does API/DB-query lookups
(a ticketing system, docs search, source lookups), that already counts as RAG — but
you likely haven't built the embedding/vector-similarity flavor, which is the one more
commonly referenced in industry material.

**Resource:** no single course pinned — search for RAG fundamentals content directly
(Anthropic and OpenAI both publish RAG/retrieval design guidance on their engineering
blogs; verify the current URL yourself rather than trusting a stale link).

**Content to quiz on:**
- What RAG is: fetch relevant external data at query time and ground generation in it,
  rather than relying solely on training-time knowledge.
- Two retrieval flavors, both count as RAG: **structured/API-query retrieval** (call a
  defined API/DB query) vs **embedding/vector-similarity retrieval** (semantic search
  over an embedded corpus).
- Core embedding-RAG pipeline stages: chunking → embedding → indexing (vector DB) →
  retrieval (top-k/similarity search) → optional reranking → injection into context.
- Failure modes: retrieval irrelevance (bad chunking/embedding choice), stale index,
  context dilution from retrieving too much low-signal content (ties directly back to
  Phase 1's "context rot").

**Quiz targets:**
- Can you explain, unprompted, why your own system's existing API/DB-query retrieval
  (if you have any) already counts as RAG despite having no vector database?
- Can you name a case where embedding-based retrieval would beat structured-query
  retrieval (e.g. "find items semantically similar to this one" vs "find item by exact
  key")?

**Practical task (mandatory):** build one minimal embedding-based retrieval example —
even a small local vector search over a handful of real documents you have on hand
(old notes, tickets, whatever's available) — since structured-query retrieval, if you
already have it, doesn't need re-demonstrating.

**Exit criteria:** you name the embedding-RAG pipeline stages from memory and have run
one small embedding-based retrieval example end to end.

---

## Phase 5 — Eval design (the priority phase for most learners)

**Goal:** most self-built agent systems have a review/QA step that's pass/fail, not
measured — no known ground-truth accuracy, no false-positive/false-negative rate. This
phase produces both the vocabulary and a real artifact fixing that gap.

**Course:** DeepLearning.AI **"Agentic AI"** (Andrew Ng) —
https://www.deeplearning.ai/courses/agentic-ai/ (free audit, paid cert optional)

**Content to quiz on (the four design patterns as this course frames them):**
- **Reflection** — the agent (or a second agent) critiques its own output and
  iterates. This is the course's name for what Anthropic's "Building Effective Agents"
  post (Phase 0) calls "evaluator-optimizer" — same underlying pattern, different
  naming from different publishers, a good interview flex to note both.
- **Tool use** — calling external APIs/databases as part of reasoning.
- **Planning** — decomposing a goal into an ordered task list before execution.
- **Multi-agent** — specialist agents coordinating on subparts of a problem.

**Quiz targets:**
- Can you distinguish "reflection" (self/peer critique loop) from "evaluator-
  optimizer" as the same pattern under different naming, and explain why that naming
  inconsistency exists across publishers (no single standard vocabulary yet in this
  field as of 2026 — itself a useful thing to say in an interview)?
- Can you describe what a *ground-truth eval set* is and why "the reviewer agent
  approved it" is not the same claim as "the output was correct"?

**Practical task (mandatory — this is the capstone-feeding task):** retrofit your own
system's review/audit stage (or a scoped-down reproduction if you don't have one yet)
with a small ground-truth eval set — a set of cases with known-correct output — and
compute actual precision/recall or false-positive/false-negative rate instead of a
binary pass/fail per review. Don't let yourself skip straight to "the reviewer catches
most things" without a number.

**Exit criteria:** you produce an actual eval report (even a small one, 10-20 cases)
with a quantified accuracy metric, not a description of the review process.

---

## Phase 6 — Observability (no course — project-guided, use judgment not a quiz bank)

**No course exists for this phase.** Do not invent one or search for a generic
"observability course" — this phase is coached from your own prior work, not external
material.

**Coaching methodology for this phase (there's no fixed content to quiz — apply this
Socratically to whatever the learner brings):**

- **Look for an existing automated-guardrail habit first.** Ask: "do you already have
  something that catches a mistake automatically instead of relying on you noticing
  it?" — a lint rule, a pre-commit hook, a CI check, anything. If yes, name it
  explicitly and reframe this phase as "apply the same instinct to a new target"
  rather than teaching from zero. That reframe (an instinct they already have, pointed
  somewhere new) is the whole point of this phase.
- **Check for per-step visibility.** Most self-built systems track aggregate cost/
  latency per run but not per-step. Ask directly: is that true for you? If so, that's
  the gap this phase's practical task closes.
- **Watch for retrofit-after-pain.** Ask whether environment/tooling fixes in their
  own history have typically been built upfront or only after repeated pain. If it's
  the latter, flag the risk explicitly before they build tracing: push them to
  instrument before their next real change ships, not after they notice a problem.

**Practical task:** add per-agent-step tracing to your own system (or a reproduction)
— at minimum, per-step token count, latency, and cost, not just the run-level
aggregate. Agent SDK lifecycle hooks (pre/post-tool-call equivalents) are the natural
mechanism if you're on Claude Code/Agent SDK.

**Coaching mode for this phase:** no course to quiz against. Instead: Socratic review
of the tracing design before it's built ("what does this trace tell you that the
aggregate cost number doesn't?"), then code review of what gets built (coach does not
write the tracing code, only reviews and questions it — never write the answer for
them).

**Exit criteria:** a real run (or reproduction) produces a per-step trace they can
point to and say which specific agent/step is the cost or latency driver, not just the
total.

---

## Phase 7 — Model Routing & Cost-Tiering

**Goal:** deliberate model-tiering (route simple steps to a cheap/small model, complex
steps to a flagship model) is a direct cost lever — OpenAI's own guidance reports up to
5-10x reduction potential. Many learners already do an informal version of this (e.g.
using a cheaper model for simple subagent tasks) — this phase names the technique and
makes the routing decision deliberate rather than incidental.

**Resource:** OpenAI — "How agents are transforming work" (openai.com, Jun 2026) —
model-tiering guidance. No single dedicated course; resource-based like Phase 0.

**Content to quiz on:**
- Why not every step needs the flagship model — cost and latency scale with model
  size; classification, extraction, and simple-formatting steps rarely need top-tier
  reasoning.
- How to decide the routing boundary: task complexity AND error cost (what happens if
  the cheap model gets this specific step wrong?) — not "make everything cheaper
  everywhere" indiscriminately.
- Where this interacts with Phase 6 (observability): you cannot route intelligently
  without per-step cost/latency data telling you which steps are over-provisioned in
  the first place.

**Quiz targets:**
- Can you name one step in your own system that's arguably over- or under-provisioned,
  and justify the call unprompted?

**Practical task (mandatory):** apply deliberate model-tier routing to one real
multi-step task in your own system and document the before/after cost number — not a
theoretical estimate.

**Exit criteria:** you show one real before/after cost number from an actual routing
change you made.

---

## Phase 8 — Security & Access Governance

**Goal:** Phase 10's habit-correction content covers tool-output prompt injection and
general guardrails; this phase adds the piece that's usually entirely absent from
personal-scale projects — access control and identity plumbing (what can an agent act
as, and what can it touch). This is what enterprise-tier senior postings gate on
directly, and most side/portfolio projects have never needed to formalize.

**Resource:** no single course pinned; reference OWASP's LLM security guidance (search
"OWASP Top 10 for LLM Applications") plus the MCP spec's own auth guidance (already
cited in Phase 2 — MCP servers commonly gate tool access via OAuth/API keys, a direct
tie-in).

**Content to quiz on:**
- Principle of least privilege applied to agent tool access: an agent's available
  tool-set should be scoped to what a given task needs, not everything the system could
  technically call.
- Identity model: does the agent act as itself (service identity) or "on behalf of" the
  invoking user (delegated identity)? Different blast radius, different audit trail.
- Common access-control mechanisms in this space: API keys/tokens, OAuth2/OIDC flows,
  policy engines (e.g. OPA) gating which tools/data a given agent invocation can reach.
- Where this meets Phase 10's tool-output-injection drill: injection is a content-layer
  risk; access governance is the blast-radius-limiting layer underneath it — scoped
  permissions cap the damage even if the agent gets tricked.

**Quiz targets:**
- Can you explain why "least privilege for agents" differs from classic app-level
  least privilege — the caller (the LLM itself) can be tricked into requesting things a
  human operator never would?

**Practical task (mandatory):** audit your own system's (or one practical task's)
tool access list and write down, per tool, what it could be misused for if the agent
were tricked into misusing it — then note what scoping, if any, currently limits that.

**Exit criteria:** you produce one real tool-access audit with documented
blast-radius reasoning per tool.

---

## Phase 9 — Comparative literacy (LangGraph / CrewAI / OpenAI Agents SDK, interview fluency only)

**Goal:** map your primary tool's subagent/skill model onto the vocabulary of the other
major frameworks, so "have you used LangGraph?" (or CrewAI, or OpenAI's SDK) has a real
answer. This is explicitly NOT a stack migration — keep your primary stack primary.

**Courses:**
1. **LangGraph Academy — "Introduction to LangGraph"** —
   https://academy.langchain.com/courses/intro-to-langgraph
   Covers: state, memory, human-in-the-loop, supervisor pattern, agent handoffs,
   structured state graphs.
2. **DeepLearning.AI — "Multi AI Agent Systems with CrewAI"** (taught by João Moura,
   CrewAI co-founder) — https://www.deeplearning.ai/courses/multi-ai-agent-systems-with-crewai/
   Covers: memory, tools (including MCP server integration), guardrails, execution
   hooks, Flows, coordination strategies, production deployment.
3. **OpenAI Agents SDK — official documentation** (openai.com/docs; SDK shipped Jun
   2026) — no structured "course," but the canonical reference for OpenAI's
   agent-orchestration primitives (Responses API, deterministic tool-heavy
   orchestration, background processing, Workload Identity Federation for enterprise
   auth). Verify the current doc URL yourself.

**Content to quiz on:**
- LangGraph's **supervisor pattern** — a central node routes to specialist nodes, but
  LangGraph makes the state graph and transitions explicit/inspectable as a
  first-class object. Compare: does your own system's orchestration logic live in
  explicit graph state, or implicitly in an orchestrator's prompt/control flow? This
  distinction — explicit state graph vs prompt-driven control flow — is the single
  most interview-useful fact from this phase.
- CrewAI's "Flows" vs "Crews" distinction — Crews are autonomous multi-agent
  collaboration (agents decide), Flows are structured, event-driven, more deterministic
  control (developer decides the sequence). Map this onto your own system: is it
  closer to a Crew or a Flow?
- OpenAI Agents SDK's orchestration model — deterministic, tool-heavy, no separate
  explicit "graph" abstraction like LangGraph's; closer to plain prompt-chained/
  tool-calling control flow with SDK-level session/background-task management layered
  on top.
- Workload Identity Federation (OpenAI's enterprise-auth answer to "what identity does
  this agent run as") — ties directly to Phase 8's access-governance content.

**Quiz targets:** can you, unprompted, explain what your primary tool's subagent
delegation model is analogous to and different from in LangGraph, CrewAI, AND OpenAI's
Agents SDK? This is the actual interview-readiness test — not "did you finish the
course" but "can you translate."

**A2A (Agent-to-Agent Protocol) — no dedicated course, awareness-level:**
Google-originated open standard for agent-to-agent communication — distinct from MCP,
which is agent-to-tool. MCP: an agent discovers and calls a tool/resource. A2A: one
agent discovers and negotiates with another autonomous agent (capability discovery,
task delegation across agent boundaries, not just within one orchestrator's workers).
CrewAI and Microsoft Agent Framework (formerly AutoGen/Semantic Kernel) both added
native A2A support in early 2026. Less commonly deployed than MCP as of Aug 2026, but
rising — flag it as "the protocol for cross-vendor/cross-org agent handoff" the way
MCP is "the protocol for tool access." Quiz target: can you state the MCP vs A2A
distinction in one sentence (tool access vs agent-to-agent handoff) without conflating
them?

**Exit criteria:** you can describe your own system's architecture correctly using
LangGraph vocabulary, CrewAI vocabulary, AND OpenAI Agents SDK vocabulary in under 2
minutes each, without notes.

---

## Extra — Computer Use (no course, awareness-level only)

**Not a numbered phase — no quiz gate, no mandatory practical, no exit criteria.**
Bonus-content only, covered if time allows or you ask for it.

Computer use = agent capability to directly control a browser or desktop UI (click,
type, screenshot-read) rather than acting only through defined tool/API calls. Anthropic
ships this in the Claude Agent SDK. As of Aug 2026, industry framing is "emerging/niche"
— relevant to browser-automation and legacy-UI-without-API use cases (e.g. customer
support tooling, QA automation, workflows where no API exists) but not yet a baseline
expectation the way MCP or evals are.

**Note:** if you use Claude Code, its Browser pane / preview tools are a
computer-use-adjacent surface (screenshot-read, click, form-fill loops) — a concrete,
low-effort way to make the term interview-legible without a dedicated project.

---

## Phase 10 — Habit correction (no course — tied to your own actual session history)

**No course exists for this phase — it's entirely derived from reviewing your own past
AI-assisted coding sessions.** This is not generic advice; it only works if applied to
documented patterns from your real history, not hypotheticals.

**Coaching methodology (there's no fixed answer key — apply this at the start of
coaching, and revisit across every practical task):**

Review the learner's own available session history if accessible, or ask them to
self-report. Look for these categories:

*Positive habits to watch for and reinforce (name them, don't re-teach):*
- Self-audits metrics before publishing or shipping them, rather than trusting the
  first number.
- Generalizes one-off fixes into reusable process (a script, a hook, a documented
  playbook) instead of a private one-time fix.
- Documents rejected alternatives, not just the chosen approach — a record of *why*
  something was ruled out, not just silence about it.
- Matches tooling to task size — knows when NOT to reach for heavier tooling (e.g.
  multi-agent fan-out) because the task doesn't warrant the overhead. This is good
  judgment, not tool-avoidance — don't coach toward over-delegating by default.

*Negative habits/gaps to probe for (this is the actual drill material):*
- **Repeats environment mistakes across sessions instead of fixing root cause once.**
  A tooling/environment snag that recurs across multiple sessions before getting a
  permanent, automatic fix (a hook, a config change) instead of being caught the first
  time. Drill: when the learner hits ANY environment/tooling snag in a session, ask
  "is this the first time, or should this already be a guardrail?" before moving on.
  Make them answer, don't just note it.
- **Heavy manual trial-and-error instead of computing the target constraint upfront.**
  Repeated iterate-and-check cycles nudging something by feel until it works, rather
  than calculating the target once (a budget, a size, a threshold) and applying it
  directly. This is the same failure mode that shows up as "iterate on agent
  cost/behavior until it feels okay" instead of "compute a target budget before
  building." Drill: whenever the learner is about to start iterating on a Phase 2-8
  practical task by trial and error, stop them and ask for the target number FIRST
  (expected cost, expected latency, expected token budget) before they write anything.
- **Guardrails get retrofitted after pain, not built upfront.** Same root pattern as
  the environment-mistake habit above, generalized: process/permission rules added
  after friction rather than before it. Drill: for the Phase 5 eval harness and Phase
  6 tracing tasks specifically, ask the learner to write down what they're
  instrumenting for BEFORE they build it, not after they notice a problem.

*Additional technical gaps (folded in as habit-correction drill material —
knowledge/design gaps rather than pure behavioral habits, but drilled the same way:
caught during practical-task review, not lectured upfront):*
- **Cost/latency SLA design** — extends the trial-and-error gap above from "guess a
  target number" to "write an explicit SLA" (max $/run, max latency, max token budget)
  before building, not just a rough conversational estimate. Drill: for any Phase 2-8
  practical task, ask for the SLA in writing before the learner opens an editor.
- **Failure-mode blindness** — agent-specific failure modes (infinite tool-call loops,
  tool-call hallucination, silent partial failure, context poisoning from bad tool
  output) aren't designed for upfront, only discovered after they happen in prod.
  Drill: for any practical task, ask "what happens when step N fails, hangs, or
  returns garbage?" before the learner starts building, not after a bug surfaces.
- **State/idempotency blindness** — no consideration of what happens if a multi-step
  agent run crashes mid-sequence and retries (duplicate side effects: double-sent
  message, double-charged API call, duplicate record). Drill: for any practical task
  with external side effects (API calls, sends, writes), ask "what happens if this
  step runs twice?" before build.
- **Tool-output prompt-injection blindness** — treating fetched tool/doc/webpage
  content as trusted context rather than untrusted input, distinct from classic
  user-input prompt injection. Drill: when a practical task ingests external content
  via a tool (fetched page, ticket body, doc), ask whether that content is isolated
  from instruction-following context with the same rigor applied to user input.
- **Model/version drift blindness** — no habit of pinning model versions or
  re-evaluating after a provider version bump. Drill: ask what model version string
  the practical task pins to, and whether a re-eval trigger exists for version
  changes.

**Coaching mode for this phase:** this isn't a standalone session so much as a lens
applied across Phases 2-8's practical tasks. Call it out explicitly at the start of
each practical task ("before you start — target number first, per the Phase 10
habit"), and log any recurrence in the progress tracker's "recurring gaps" section.

**Exit criteria:** across at least 2 of the Phase 2-8 practical tasks, the learner
states a target/budget number BEFORE building, unprompted by the coach.

---

## Capstone

Extend your own agent system (or build a second one) that closes the two most common
gaps directly: a quantified eval harness (Phase 5 output) and per-agent tracing (Phase
6 output), combined into one system. This becomes your next resume bullet.
