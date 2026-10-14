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

Course/resource URLs and model/API claims were last verified 2026-10-13 — this field
moves fast, verify links still resolve and pages carry no deprecation banner before
relying on them.

Throughout, **"your own agent system"** is a placeholder for whatever real project you
bring — a side project, a work system, or a fresh one you build alongside this
curriculum specifically to have a concrete example.

---

## Phase 0 — Pattern vocabulary (no course, ~1 session)

**Goal:** name the patterns you're probably already using informally in your own agent
system, so you can cite them by name in interviews and in this curriculum's later
phases.

**Resource:** Anthropic engineering blog post "Building Effective Agents" (Dec 2024) —
https://www.anthropic.com/engineering/building-effective-agents. The page now carries a
banner saying the tooling landscape it describes has changed; the banner scopes the
tooling, not the five patterns, which still stand. (Verified 2026-10-13.)

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

**Worked example (added 2026-08-12):** orchestrator-workers vs. parallelization/
sectioning, the most-confused pair, walked through on a generic support-ticket
system. If the orchestrator reads each incoming ticket and DECIDES, per-ticket, which
lookups are relevant (skip a vendor-dependency check if none is mentioned in the
ticket) — that decision-per-input is orchestrator-workers. Compare: if the system
instead ALWAYS ran all lookups on every ticket regardless of content, fixed in
advance, that's parallelization/sectioning instead, even though the code shape (fan
out, run concurrently, merge) looks nearly identical. The tell: is the subtask list
fixed at design time, or decided at run time per input?

**Rubric (added 2026-08-12):**
- **Pass:** names all 5 patterns unprompted, correctly tags 2+ onto your own system,
  and can redo the orchestrator-workers vs. parallelization distinction on a NEW
  example (not the one taught) without prompting.
- **Partial:** names the patterns and taps the right ones for your system, but needs a
  hint to apply the orchestrator-workers/parallelization distinction to a new example.
- **Fail:** cannot name 4+ patterns from memory, or misassigns your system's
  architecture.

**Exit criteria:** you can name all 5 patterns from memory and correctly tag which
ones your own system implements, without looking at the code.

---

## Phase 1 — Prompt & Context Engineering

**Goal:** move from "prompts that work well enough" to explicit vocabulary and
deliberate technique for prompt design and token/context economy.

**Resources (real published material, not a formal certificate course — same tier of
authority as Phase 0's blog-post source):**
1. Anthropic's "Prompting best practices" (the docs call it "the living reference") —
   https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
   Covers: clear/direct instructions, multishot (few-shot) examples, XML tag
   structuring, role/system prompts, thinking, prompt chaining. (Verified 2026-10-13.)
2. Anthropic engineering blog — "Effective context engineering for AI agents" (Sep 2025)
   — https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
   Covers: context window as finite/precious resource, "context rot," just-in-time
   retrieval vs upfront loading, compaction, sub-agent context isolation, structured
   note-taking/memory.

**Content to quiz on:**

*Prompting technique:*
- Clear & direct instruction — why specificity reduces output variance.
- Multishot/few-shot — when examples outperform instructions alone.
- Chain-of-thought — explicit step-by-step reasoning before the answer; why it helps
  reasoning-heavy tasks specifically, and the cost tradeoff (more output tokens, more
  latency) that makes it wrong to apply everywhere. On current models (Claude 4.6+)
  this is adaptive thinking — the model decides how much to think and you steer cost
  with the `effort` parameter, not a manual `budget_tokens` cap (that returns a 400 on
  4.7+). Prompted step-by-step reasoning remains the technique for thinking-off setups.
- XML tag structuring — why tagging reduces ambiguity in multi-part prompts.
- System prompt vs user turn — persona/role/constraints belong in system prompt, task
  specifics in the user turn.
- Output format control — structured outputs (schema-enforced, see below), explicit
  format instructions, XML-tagged output sections. (Prefilling the assistant turn is
  not supported on Claude 4.6+ models — a 400 error — so it is not taught here.)

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

**Worked example (added 2026-08-12):** context rot vs. token-limit exhaustion, the
most commonly conflated pair. Token-limit exhaustion: a long session hits the hard
context ceiling, the tool errors or truncates — a capacity problem, fixed by clearing/
compacting. Context rot: a session well under its token budget still degrades — e.g.
a dozen stale file reads from an abandoned earlier approach are still sitting in
context, and the model starts referencing that dead approach or missing the actual
current instruction buried under it — a signal-to-noise problem, not a capacity
problem. Fix for rot is pruning/re-scoping, not "there's room left so it's fine."

**Rubric (added 2026-08-12):**
- **Pass:** states the rot-vs-exhaustion distinction unprompted using a concrete
  example (not just definitions), and correctly maps 3+ of your own existing practices
  onto the formal vocabulary without being told the mapping.
- **Partial:** can state the distinction when asked directly but doesn't offer a
  concrete example unprompted; maps 1-2 of your own practices only with a hint.
- **Fail:** conflates rot with hitting the token ceiling, or cannot name 4+ of the 6
  prompting techniques from memory.

**Practical task (mandatory):** audit one real prompt or system-prompt file you
already use and rewrite it applying at least 3 of the techniques above. Document
before/after and, for each change, whether it targets output quality or token cost
(not all techniques target both).

**Structured outputs (added 2026-09-09):**
The reliability primitive most self-taught agent builders have invented without naming.
A model asked for prose and parsed with a regex is a system with an unbounded failure
mode; a model given a schema and validated against it fails loudly, in one place.

- **Tool / function schemas** — the JSON Schema attached to a tool definition is not
  documentation, it is the contract the model is constrained by. Ambiguous field names
  and missing enums are prompt bugs that surface as tool-call bugs.
- **Structured output vs parsing** — asking for JSON in the prompt and hoping is not
  structured output. The distinction is whether the format is *enforced* by the API or
  *requested* in the text.
- **Validation at the boundary** — parse, validate and reject at the edge of the agent,
  not three layers in. A validation failure is a retry signal, and the retry should
  carry the validation error back to the model.
- **Check your own system first.** If you have a fixed return format or a confidence
  scale you enforce by convention, you already built this without the vocabulary. It is
  also what PydanticAI is built around and what LangGraph's typed state object gives you.

**Quiz target:** why is "return JSON" in a prompt weaker than a schema-constrained tool
call, and what failure does each one produce under load?

**Exit criteria:** you name the 6 prompting techniques and 5 context-efficiency
concepts from memory, correctly self-diagnose which your own prompts already use vs.
are missing, and complete the practical rewrite with documented before/after.

---

## Phase 2 — Claude Code / Agent SDK + MCP formal layer

**Goal:** close the gap between "used an AI coding tool heavily" and "can explain its
primitives and has built an MCP server, not just consumed one."

**Courses (Claude Academy, free; sign-in only needed to save progress). Catalogue:
https://academy.claude.com/courses — all links verified 2026-10-13.** Older
`anthropic.skilljar.com` links still resolve but are the legacy surface.
1. **Claude Code 101** — https://academy.claude.com/courses/claude-code-101
   12 lessons, ~1.5 hr. Covers: installation (terminal/VS Code/JetBrains/Desktop/web),
   the Explore-Plan-Code-Commit workflow, CLAUDE.md project memory, basic MCP
   integration. Following along needs a Claude plan or API key.
2. **Claude Code in Action** — https://academy.claude.com/courses/claude-code-in-action
   9 lessons, ~1 hr. Covers: steering long sessions, CLAUDE.md, verification skills,
   permission modes, hooks, routines/headless runs, GitHub Actions and Code Review,
   verifying unsupervised runs, plugins — maps onto 2b's test-loop and HITL material.
3. **Introduction to Model Context Protocol (MCP)** —
   https://academy.claude.com/courses/introduction-to-model-context-protocol
   10 lessons, ~1 hr. Covers: MCP architecture (host/client/server roles), the three
   primitives — tools, resources, prompts — and building a server and client with the
   Python SDK.
4. *(optional)* **Model Context Protocol: Advanced topics** —
   https://academy.claude.com/courses/model-context-protocol-advanced-topics
   (11 lessons, ~1.5 hr; catalogue listing verified, contents not reviewed.)

**Reference spec (not a course, but authoritative):**
https://modelcontextprotocol.io/specification/2026-07-28 — current revision as of
2026-10-13; the version list lives at
https://modelcontextprotocol.io/specification/versioning.

**Spec revision note (2026-07-28):** MCP is now stateless. No initialize handshake —
each request carries its protocol version and client capabilities in `_meta`, and
servers implement `server/discover`. `subscriptions/listen` replaces the old subscribe
stream. Roots, Sampling, Logging, HTTP+SSE and dynamic client registration are
deprecated; tasks moved to an extension. The three server primitives are unchanged.

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

**Worked example (added 2026-08-12):** MCP tools vs. resources, the common confusion.
A `create_ticket(project, summary, description)` tool DOES something — model invokes
it, side effect happens, a ticket gets created. A `ticket://PROJ-123` resource IS
something — the model reads it to get the current ticket content, same as reading a
file, no side effect. A read-only lookup is resource-shaped; an action that posts a
comment or changes state back to the source system is tool-shaped. The tell: does
invoking it change external state, or just return a view of existing state?

**Rubric (added 2026-08-12):**
- **Pass:** correctly classifies a NEW example (not the one taught) as tool or
  resource unprompted, and states the host/client/server role split from memory.
- **Partial:** classifies correctly only when given the "does it change state"
  question as a hint.
- **Fail:** conflates tools and resources, or cannot name the 3 MCP primitives.

**Practical task (mandatory, not optional):** build one minimal MCP server (Python SDK)
exposing at least one tool and one resource, and connect it to a real session. This
closes the "consumer only" gap directly. The Python SDK is on v2 (`mcp` 2.x on PyPI,
verified 2026-10-13), which renamed `FastMCP` to `MCPServer` — v1 tutorials won't
import. Pin the version and follow the SDK's own migration page
(https://py.sdk.modelcontextprotocol.io/migration/) rather than an old tutorial.

**AGENTS.md standard (added 2026-08-12, verified 2026-10-13):** an open, vendor-neutral
markdown format (https://agents.md, stewarded by the Agentic AI Foundation under the
Linux Foundation) at repo root carrying project instructions for AI coding agents —
read by Cursor, Copilot's coding agent, Windsurf, Codex, Gemini CLI and others, vs. a
tool-specific file (e.g. CLAUDE.md) which only one tool reads. Same content
(build/test commands, conventions, architecture notes). Claude Code also reads it: with
no CLAUDE.md present it reads AGENTS.md by default; if both exist it reads only
CLAUDE.md unless CLAUDE.md imports AGENTS.md. So the portable setup — the answer to
"how do you configure agent behavior across different tools" — is one AGENTS.md plus a
CLAUDE.md that imports it.

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

**Worked example (added 2026-08-12, covers Phase 2's planning/loop content):** tracer
bullet vs. MVP, the most-confused pair in the planning material above. A tracer bullet
for a new "post results to Slack" feature would be: hardcode a single fake result,
wire it through the thinnest possible path to an actual Slack message landing in a
test channel — ugly, no real logic, throwaway. Its only job is proving the connection
works before investing further. An MVP of the same feature would be the smallest REAL
version — actual logic, meant to ship and be used, not thrown away.

**Rubric (added 2026-08-12, covers Phase 2's planning/loop content):**
- **Pass:** states tracer-bullet vs. MVP unprompted with a concrete example, AND names
  which of the two unattended-loop failure modes (bad code lands vs. irreversible
  action taken) each of CI-gate and HITL-boundary specifically prevents.
- **Partial:** gets one of the two distinctions right without prompting, needs a hint
  for the other.
- **Fail:** conflates tracer bullet with MVP, or says "unattended loops are unsafe"
  without naming which safeguard prevents which failure mode.

**Exit criteria:** courses marked complete by self-report + working MCP server you can
demo (describe what it does, run/test it live). Planning methodology and
test-loop/autonomous-loop content above are quizzed as part of this phase's exit but
don't require a separate practical artifact — they inform HOW this phase's (and later
phases') practical tasks should be approached.

---

## Phase 2c — Claude Projects (the claude.ai surface)

**Added 2026-09-03.** Same phase number as 2a/2b (integers only, no decimal phases) — a
third sitting under Phase 2's "Claude platform surfaces" territory, gated like the
others. It depends on Phase 2's MCP content and Phase 3's Skills content, so run it
after both if you can.

**Goal:** everything else in this curriculum is Claude Code / Agent SDK — the developer
surface. **Claude Projects** is a different Anthropic product surface (claude.ai web,
including Team and Enterprise), and it's the one non-engineer stakeholders actually
deploy on, so it turns up by name in job descriptions for "wrap Claude around our
existing data" roles. The goal here is not to learn a UI. It's to be able to say
precisely which Phase 1-3 primitives carry over to that surface and which ones quietly
do not — because three of them don't, and assuming they do is the mistake this
sub-phase exists to prevent.

**Resources (no course exists — resource-based, same precedent as Phases 0 and 7. URLs
re-verified resolving 2026-10-13; this surface moves fast, re-check before relying on any
number below):**
1. "What are projects?" —
   https://support.claude.com/en/articles/9517075-what-are-projects
   (Banner seen 2026-10-13: a new version of projects is in beta, starting with Claude
   Code. Everything below describes the current chat projects, which it says are
   unchanged.)
2. "Retrieval augmented generation (RAG) for projects" —
   https://support.claude.com/en/articles/11473015-retrieval-augmented-generation-rag-for-projects
3. "Upload files to Claude" —
   https://support.claude.com/en/articles/8241126-upload-files-to-claude
4. "Use connectors to extend Claude's capabilities" —
   https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities
5. "How to create custom skills" —
   https://support.claude.com/en/articles/12512198-how-to-create-custom-skills
6. Agent Skills overview — the format spec Phase 3 already teaches, here for
   side-by-side comparison —
   https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview

**Content to quiz on:**

*What a Project is, and the tie-back:*
- A Project is a self-contained workspace with its own chat history and its own
  knowledge base. Two things persist across every chat inside it: **project
  instructions** (that's the official term) and **project knowledge** (uploaded files).
  Free tier is capped at 5 projects; Pro/Max/Team/Enterprise are not.
- **The tie-back:** project instructions are the claude.ai analogue of CLAUDE.md — same
  job (persistent per-workspace instructions auto-applied to every session on that
  workspace), different delivery. What you give up moving from a file to a UI field:
  it's no longer version-controlled, diffable, or reviewable in a PR. Name that loss
  explicitly. "It's CLAUDE.md in a text box" is half the answer; the missing half is
  that a prompt change nobody can diff is a prompt change nobody can review.

*Project knowledge and managed RAG:*
- Uploaded files persist across every chat in the project. When project knowledge
  approaches the context limit, Claude **automatically** stops loading everything and
  switches to a project knowledge search tool that retrieves only the relevant parts —
  Anthropic states this expands capacity **up to 10x**. It isn't user-toggleable, and
  it reverts if the content later shrinks below the threshold.
- **This is Phase 4's RAG, managed.** Add it as a third flavor alongside
  structured/API-query retrieval and embedding/vector retrieval: **retrieval you don't
  own** — no chunking, embedding, or indexing decisions of yours, and no visibility
  into the ones being made for you.
- The design consequence, and the actual quiz-worthy point: once retrieval kicks in you
  lose the guarantee that every row of a large file is in context. That's fine for
  "find the relevant part," wrong for "reason across all of it" — totals, cross-row
  consistency, exhaustive checks. Phase 1's context-rot instinct, pointed at a managed
  system you can't inspect.
- **Don't quote a fixed project-size number.** Anthropic doesn't publish one. The
  ceiling depends on the model's context window (as of 2026-10-13 Anthropic's
  context-window help article lists 1M on the newest tiers, 500K on several 4.x models,
  200K on older ones), and past that RAG takes over. A stale "projects hold 200K tokens" claim is exactly the currency
  trap this curriculum keeps warning about.

*File uploads (verify before promising anyone this works):*
- Supported: PDF, DOCX, CSV, TXT, HTML, ODT, RTF, EPUB, JSON, plus images. **XLSX
  requires code execution enabled** — it is not a bare drag-and-drop type.
- Project files specifically: 30MB per file, no file-count cap (bounded by the context
  budget instead), text extraction only except multimodal PDFs.
- So "spreadsheet-shaped business data works in project knowledge" is true, with the
  caveat that CSV is the friction-free path and native Excel needs a setting turned on.

*Skills on claude.ai vs. Phase 3's Agent Skills — same format, different surface:*
- **Same** SKILL.md + YAML frontmatter (`name`, `description`) packaging Phase 3
  teaches. **Different** plumbing: on claude.ai a Skill is a zipped folder uploaded
  through Customize > Skills, requires code execution enabled, and is per-user by
  default — making one available org-wide is a separate admin provisioning path, not
  the same action. In Claude Code it's a folder on disk, no upload step, no
  code-execution prerequisite. Sync is one-way: Claude Code signed in with a claude.ai
  account downloads your claude.ai skills into `~/.claude/skills/synced/`; edits don't
  upload back, and Skills API uploads are a third, separate store. (Verified
  2026-10-13; Anthropic's platform overview page still says "do not sync" — the
  support and Claude Code docs are newer.)
- Say both halves when asked. Phase 5 makes the same kind of honesty call about
  "reflection" vs. "evaluator-optimizer" (one pattern, two publishers' names); this is
  the mirror image — one name and one format, two surfaces with different constraints.
  Claiming they're unrelated is as wrong as claiming they're interchangeable.

*MCP from claude.ai — the carry-over trap:*
- Custom connectors on claude.ai are **remote** MCP servers, reached from Anthropic's
  cloud, not from your machine, so they must be reachable from the public internet.
  **Local MCP servers run only through Claude Desktop** (desktop extensions), not from
  claude.ai or a Project.
- So the server you built for Phase 2's practical task cannot be dropped into a Project
  as-is. Being able to say exactly what would have to change — public HTTPS endpoint,
  hosting, OAuth or equivalent auth — is the interview-grade version of this answer.
- In Team/Enterprise an Owner enables a connector for the organization first; each user
  then authenticates to it individually, so Claude inherits that user's own permissions
  in the connected system.
- Google Drive and Gmail are among the connectors Anthropic offers directly. The
  directory (https://claude.com/connectors) lists connectors from Anthropic and from
  third-party developers, labelled Verified or Community. As of 2026-10-13 it carries
  vendor-specific database connectors (e.g. Neon, PlanetScale, Supabase) but no
  generic SQL, data-warehouse or ERP connector surfaced in a sampled search — check the
  live directory. Reaching a system with no listed connector means hosting your own
  remote MCP server. So "MCP links Claude to our live ERP data" is achievable and
  officially supported, but as a service you build, host, and secure, not a toggle.

*Governance (Phase 8 tie-in):*
- **Connectors are only available in private projects.**
- Project access is per-user "Can view" / "Can edit," or shared org-wide.
- Per-user connector authentication is a concrete instance of Phase 8's
  delegated-identity model: the agent acts as the invoking user, so blast radius is
  capped at that user's own permissions in the connected system rather than a shared
  service account's.

**Quiz targets:**
- Can you name what project instructions are the analogue of on the Claude Code side,
  AND the concrete thing you lose moving to the UI field, unprompted?
- Can you state why the MCP server from Phase 2's practical won't work inside a Project
  as built, and name specifically what would have to change?
- Can you name managed RAG as a third retrieval flavor alongside Phase 4's two, and say
  what design guarantee you give up when it activates?

**Worked example (added 2026-09-03):** "same protocol, different surface," run on MCP —
the trap most likely to catch someone who's finished Phase 2 and assumes it transfers.
Phase 2's practical is a local Python stdio server: Claude Code launches it as a
subprocess on your own machine, talks to it over stdin/stdout, no network exposure and
no auth — it's trusted because nothing outside your machine can reach it. Now put the
same capability inside a Project: claude.ai reaches connectors from Anthropic's cloud,
so that process has to become a publicly reachable HTTPS MCP endpoint with OAuth or
equivalent in front of it. The protocol is byte-for-byte the same and your tool and
resource definitions don't change at all — the transport, hosting, and auth story
change completely. The tell: MCP compatibility was never the question, reachability and
authentication were.

**Rubric (added 2026-09-03):**
- **Pass:** states the local-stdio-vs-remote-connector distinction unprompted and names
  what would have to change about their own Phase 2 server; names managed RAG as a
  third retrieval flavor and the guarantee it costs; identifies project instructions as
  the CLAUDE.md analogue with the version-control loss named.
- **Partial:** gets the project-instructions/CLAUDE.md analogy but needs a hint that a
  local MCP server won't work as-is, or describes claude.ai Skills as an unrelated
  feature rather than the same format on a different surface.
- **Fail:** assumes a local MCP server works from claude.ai unchanged, or states a fixed
  project-capacity token number as settled fact.

**Practical task (mandatory):** build one real Project — project instructions plus real
structured data uploaded as knowledge (CSV is the friction-free path) — then run one
query that requires reasoning across the whole file rather than looking up one part of
it. Write down three things: what you put in instructions vs. knowledge and why;
whether the answer showed signs of retrieval rather than whole-file reasoning, and how
you could tell; and one thing you'd have to BUILD rather than configure to make this
production-grade for a company. No code required — this is a boundary-finding exercise,
and the written answer is the artifact.

**Exit criteria:** given a described business workflow ("we want Claude to help plan X
against our existing spreadsheets and systems"), you can split the answer into what's
configuration (project instructions, knowledge upload, an existing connector) and what's
engineering you'd have to build and host (a remote MCP server for any system without
one, an eval harness per Phase 5, HITL gating per Phase 2b) — and you have one real
Project to point at.

---

## Phase 3 — Agent Skills

**Goal:** name a pattern you may already rely on without formal vocabulary for it. If
you use Claude Code, your own environment's skill listing is a live example — most
learners have been consuming Skills constantly without a term for what they are or how
they differ from a system prompt or an MCP tool.

**Resources:**
1. Anthropic engineering blog — "Equipping agents for the real world with Agent
   Skills" (Oct 2025; updated Dec 2025 to note Agent Skills is now an open standard) —
   https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills
2. Google Cloud blog — "Level Up Your Agents: Announcing Google's Official Skills
   Repository" (Apr 2026, vendor-authored) —
   https://cloud.google.com/blog/topics/developers-practitioners/level-up-your-agents-announcing-googles-official-skills-repository/
   — confirms this is a cross-vendor pattern, not Claude-only; good interview point.
3. The open standard itself — https://agentskills.io/specification. Required
   frontmatter: `name` (≤64 chars, lowercase/digits/hyphens, matches the folder name)
   and `description` (≤1024 chars); optional `license`, `compatibility`, `metadata`,
   `allowed-tools`. Claude Code adds extra fields; claude.ai uploads and the Skills API
   reject them.
All verified 2026-10-13.

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

**Worked example (added 2026-08-12):** Skill vs. system prompt vs. MCP tool, walked
through on one need — "format output as a Slack message." As a system prompt: the
formatting rule lives permanently in every session's instructions, even the runs that
never touch Slack — always resident, never free. As an MCP tool:
`format_for_slack(text)` — a single callable action, no judgment involved, just
input→output. As a Skill: a `slack-formatting/` folder with instructions on WHEN and
HOW to format for Slack (tone, structure, when to thread vs. post fresh) — loaded only
when a Slack-bound output is actually being produced, and it may itself call the MCP
tool above as one step. The Skill is the "how/when," layered on top of the tool's "do."

**Rubric (added 2026-08-12):**
- **Pass:** picks the correct home (system prompt / tool / Skill) for 2+ NEW
  hypothetical needs unprompted, justifying the token-economy reasoning each time.
- **Partial:** picks correctly with a hint toward "how often does every session need
  this."
- **Fail:** cannot distinguish Skill from system prompt or from MCP tool without the
  distinction being re-explained.

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

**Resources (no single course pinned; both vendor-authored, verified 2026-10-13):**
1. Anthropic — "Introducing Contextual Retrieval" (Sep 2024; benchmark figures are
   Anthropic's own) — https://www.anthropic.com/engineering/contextual-retrieval
2. OpenAI retrieval guide — https://developers.openai.com/api/docs/guides/retrieval
   (re-check any defaults or pricing before quoting them).

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

**Worked example (added 2026-08-12):** why exact-key API lookups already count as RAG,
and where embedding-retrieval would beat them. Querying a ticket system by exact key
(`PROJ-123`) is structured/API-query RAG: fetch relevant external data at query time,
ground generation in it, no training-time-only knowledge involved. It counts as RAG
despite zero vector math. Now: "find tickets that FEEL like this one" — no exact key,
no field match, just semantic similarity of the problem description — a structured
query can't do this at all (there's no field for "similar meaning"). That's the case
embedding-retrieval wins: fuzzy/semantic match where no structured field captures the
similarity.

**Rubric (added 2026-08-12):**
- **Pass:** explains unprompted why exact-key lookups are RAG, and names a
  semantic-similarity use case (not the exact one taught) where embeddings would beat
  structured query.
- **Partial:** agrees structured lookups are RAG only after being asked "does it fetch
  external data at query time," or needs the taught example repeated rather than
  generating a new one.
- **Fail:** believes RAG requires a vector database, or cannot name the embedding-RAG
  pipeline stages (chunk→embed→index→retrieve→[rerank]→inject).

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
https://www.deeplearning.ai/courses/agentic-ai/ (free audit = videos and forum; graded
assignments and certificate need a paid Pro membership — verified 2026-10-13)

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

**Worked example (added 2026-08-12):** "reviewer approved it" vs. "it was correct,"
the core distinction this phase exists to fix. A review stage today might: read the
output, say "looks well-supported," pass it — a SUBJECTIVE judgment call by another
LLM, no ground truth involved. A ground-truth eval set: 15 historical cases where the
CORRECT outcome is already known (verified after the fact) — run the pipeline on
those 15, compare its output to the KNOWN answer, count how many match. That produces
an actual number (e.g. 12/15 correct, 2 false positives, 1 false negative) — a claim
about correctness, not just "the reviewer was satisfied."

**Rubric (added 2026-08-12):**
- **Pass:** explains unprompted why "reviewer approved" ≠ "correct" using the
  ground-truth framing, and correctly bridges "reflection" (course term) to
  "evaluator-optimizer" (Phase 0 term) as the same pattern.
- **Partial:** agrees they're different only when asked "could the reviewer approve a
  wrong answer," or needs the reflection/evaluator-optimizer bridge pointed out.
- **Fail:** treats reviewer-approval as equivalent to correctness, or cannot define a
  ground-truth eval set.

**Practical task (mandatory — this is the capstone-feeding task):** retrofit your own
system's review/audit stage (or a scoped-down reproduction if you don't have one yet)
with a small ground-truth eval set — a set of cases with known-correct output — and
compute actual precision/recall or false-positive/false-negative rate instead of a
binary pass/fail per review. Don't let yourself skip straight to "the reviewer catches
most things" without a number.

**Prompt versioning and change safety (added 2026-09-09):**
Evals are only worth building if something triggers them. This is the trigger.

- **A prompt is code.** It lives in version control, it has a diff, and a change to it
  is a deploy. A prompt edited live in a console is an untracked production change.
- **The eval gate** — an eval suite runs against every prompt change and every model
  version change, offline, before the change ships. This is the answer to "how do you
  ship a prompt change safely," a standard senior interview question and the natural
  follow-up to any question about evaluation.
- **Model-version pinning** — pin the version, and treat a provider's model update as a
  change requiring the same gate. Model-version drift is on the Phase 10 technical-gap
  list; this is where the control for it lives.
- **Rollback** — what does reverting a bad prompt actually take, and how fast?

**Quiz target:** describe the pipeline from "I want to change this prompt" to "it is
live," naming what gates it and what happens when the eval regresses.

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

**Batch and scheduled agent workloads (added 2026-09-09):**
Most agent teaching assumes a request-response shape. A large share of real enterprise
agent work is the opposite: a job that fires on a schedule, processes a known volume,
and must be finished by a deadline. That shape has its own failure vocabulary.

- **Deadline arithmetic** — the batch window divided by the item count gives the
  per-item budget, which sets the parallelism width, which decides the topology. Do this
  arithmetic before choosing an architecture, not after.
- **Partial failure** — a nightly run that must finish by morning is entirely about what
  happens when item 738 fails. Does the run stop, skip, retry or quarantine?
- **Dead-letter queues** — where a poisoned item goes so it stops blocking the run, and
  who looks at that queue in the morning.
- **Restart idempotency** — if the run dies at item 600, does the restart re-process the
  first 600? In an agent workload that is not just wasted compute, it is a second token
  bill and possibly duplicate side effects.
- **Checkpointing** — the state that has to survive a crash. Note this is exactly what
  LangGraph's checkpointer provides and what a prompt-driven orchestrator has to build
  by hand.

**Quiz target:** given a nightly agent batch with a hard morning deadline, name the
per-item budget calculation, the partial-failure policy, and what a restart costs.

**Exit criteria:** a real run (or reproduction) produces a per-step trace they can
point to and say which specific agent/step is the cost or latency driver, not just the
total.

---

## Phase 7 — Model Routing & Cost-Tiering

**Goal:** deliberate model-tiering (route simple steps to a cheap/small model, complex
steps to a flagship model) is a direct cost lever. Many learners already do an informal version of this (e.g.
using a cheaper model for simple subagent tasks) — this phase names the technique and
makes the routing decision deliberate rather than incidental.

**Resource:** OpenAI — "A Practical Guide to Building Agents", model-selection section
(vendor-authored) —
https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf
— baseline on the most capable model, then swap in smaller models step by step wherever
evals hold. For live tier prices, read the provider's own models and pricing pages at
the time you quote them. No single dedicated course; resource-based like Phase 0.
Verified 2026-10-13.

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

**Worked example (added 2026-08-12):** the routing boundary is complexity AND error
cost, not complexity alone. Step A: classify a ticket's urgency (low/med/high) from
its title — low complexity, and if the cheap model misclassifies, worst case is a
triage delay, cheaply caught downstream. Good candidate for a small/cheap model. Step
B: decide whether a flagged finding is a genuine security risk worth escalating — also
could look "simple" (one judgment call), but wrong-call error cost is high (missed
real vulnerability, or false alarm burning an engineer's time). Stays on the flagship
model DESPITE looking similarly "simple" to step A on complexity alone — error cost is
what separates them.

**Rubric (added 2026-08-12):**
- **Pass:** picks a real step from their own system, justifies the tier call using
  BOTH complexity and error cost (not complexity alone) unprompted.
- **Partial:** names a step and tier but justifies on complexity alone until asked
  "what happens if the cheap model gets this wrong."
- **Fail:** can't name a candidate step, or argues for "cheaper everywhere" without
  qualifying by error cost.

**Practical task (mandatory):** apply deliberate model-tier routing to one real
multi-step task in your own system and document the before/after cost number — not a
theoretical estimate.

**Prompt caching (added 2026-09-09):**
Frequently the largest single cost lever available, and frequently misconfigured into
doing nothing at all.

- **What it actually is** — a cache of the processed prefix of a prompt, so repeated
  context is not re-processed on every call. Priced differently from ordinary input
  tokens: a write costs more, a read costs less.
- **Why prefix order decides whether it works** — cacheable content has to sit at the
  front and stay byte-identical. A timestamp or a per-request ID injected near the top
  of a system prompt invalidates everything after it. This is the most common way teams
  silently get no caching at all.
- **What to cache in an agent** — system prompt, tool definitions, long stable context
  (a policy document, a schema, a reference corpus). What not to cache: the per-request
  payload.
- **Measuring it** — cache-read vs cache-write vs uncached input tokens in the usage
  numbers, and the cost delta per run. That is a real before/after number of exactly the
  kind this phase's practical asks for.

**Rate limits, backoff and concurrency (added 2026-09-09):**
Cost tiering is covered above; throttling is a separate constraint. A fan-out agent
often hits limits before it hits budget.

- **Rate limits are per-dimension** — requests per minute and tokens per minute are
  separate ceilings (Anthropic's API splits tokens into input and output per minute),
  and an agent that batches aggressively can pass one while failing the other.
- **429 handling** — retry with exponential backoff *and jitter*, and honour a
  `retry-after` header when one is returned. Naive immediate retry turns a throttle into
  an outage. A spend-cap 429 has no `retry-after` and will not clear on retry —
  distinguish it from a rate-limit 429.
- **Concurrency caps as a design input** — the parallelism width chosen in the batch
  section is bounded by the rate limit, not only by the deadline. Reconcile the two; the
  lower one wins.
- **The interview form** — "what happens to your fan-out when the provider throttles
  you?" A candidate who has only ever tuned cost has no answer.

**Streaming and time-to-first-token (added 2026-09-09, awareness level):**
Total latency and time-to-first-token are different numbers, and for anything a person
watches, the second is the one they feel. Streaming does not make a response faster; it
makes the wait legible. Relevant to any user-facing tool, irrelevant to a nightly batch
— knowing which situation you are in is the actual skill.

**Exit criteria:** you show one real before/after cost number from an actual routing
change you made.

---

## Phase 8 — Security & Access Governance

**Goal:** Phase 10's habit-correction content covers tool-output prompt injection and
general guardrails; this phase adds the piece that's usually entirely absent from
personal-scale projects — access control and identity plumbing (what can an agent act
as, and what can it touch). This is what enterprise-tier senior postings gate on
directly, and most side/portfolio projects have never needed to formalize.

**Resource:** no single course pinned; reference the OWASP *Top 10 for LLM
Applications* **2026 edition** (Aug 2026) —
https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/ — note the
genai.owasp.org/llm-top-10/ landing page still shows the 2025 list, so check the year.
Excessive Agency moved up to #3, which is the entry this phase's least-privilege audit
answers. Also OWASP's *Top 10 for Agentic Applications for 2026* —
https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/ —
plus the MCP spec's own auth guidance (cited in Phase 2; OAuth-based, a direct tie-in).
Verified 2026-10-13.

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

**Worked example (added 2026-08-12):** why agent least-privilege differs from classic
app least-privilege. Classic: a human ops engineer has prod DB write access scoped off
unless they explicitly request it — the human doesn't accidentally ask for the wrong
thing by being tricked. Agent case: if an orchestrator had a `delete_record` tool
available "just in case," a crafted piece of fetched content (untrusted tool-output,
e.g. a scraped changelog entry) could contain text engineered to make the orchestrator
invoke `delete_record` on real data — the CALLER itself (the LLM) got manipulated into
requesting something a human operator never would have. Least privilege for agents has
to assume the requester can be tricked, not just that the requester is trusted but
might make a mistake.

**Rubric (added 2026-08-12):**
- **Pass:** explains unprompted why the agent-as-caller can be tricked (ties to
  Phase 10's injection-blindness drill) and completes a real tool-by-tool blast-radius
  audit, not a hypothetical one.
- **Partial:** states least-privilege correctly in the abstract but doesn't connect it
  to prompt-injection-triggered misuse without a hint.
- **Fail:** describes agent least-privilege identically to classic app least-privilege
  with no mention of the trickable-caller distinction.

**Practical task (mandatory):** audit your own system's (or one practical task's)
tool access list and write down, per tool, what it could be misused for if the agent
were tricked into misusing it — then note what scoping, if any, currently limits that.

**Data classification, PII and what leaves the building (added 2026-09-09):**
This phase covers tool-access blast radius and prompt injection. It also has to cover
what data reaches the model provider in the first place. For anyone building an internal
AI tool at an existing company, that is the first question the company asks.

- **Know what class of data you are handling** before the design, not after: public,
  internal, confidential, regulated (PII, PHI, cardholder data, and whatever sector
  rules apply to you).
- **What crosses the boundary** — which fields actually need to reach the model, and
  which are being sent only because they happened to be in the record. Minimisation is
  the cheapest control available and almost never applied.
- **Redaction and tokenisation before the call**, not after, with the reverse mapping
  held on your side. Note the failure mode: redaction that removes the signal the model
  needed produces confident nonsense.
- **Retention and training posture** — what the provider retains, for how long, whether
  it is used for training, and what your agreement actually says. Answering this
  precisely is a senior signal; being vague about it is disqualifying in a regulated
  industry.
- **Logs are a leak surface.** Traces, eval sets and debugging output all copy the
  prompt somewhere else. An eval set built from real records is a copy of production
  data with a friendlier name.
- **The human decision line** — for a regulated finding, the model flags and evidences,
  a human decides, and the audit record is the human's decision. Say this sentence out
  loud in any interview that touches compliance.

**Quiz target:** given a workflow over customer documents, state what data reaches the
provider, what is stripped first, what is retained where, and who makes the finding.

**Exit criteria:** you produce one real tool-access audit with documented
blast-radius reasoning per tool.

---

## Phase 9 — Comparative literacy (LangGraph / CrewAI / OpenAI Agents SDK, interview fluency only)

**Goal:** map your primary tool's subagent/skill model onto the vocabulary of the other
major frameworks, so "have you used LangGraph?" (or CrewAI, or OpenAI's SDK) has a real
answer. This is explicitly NOT a stack migration — keep your primary stack primary.

**Courses:**
All three verified 2026-10-13.
1. **LangGraph Academy — "Introduction to LangGraph - Python"** (free, ~6 hr) —
   https://academy.langchain.com/courses/intro-to-langgraph
   Covers: state schemas and reducers, short- and long-term memory, human-in-the-loop
   (breakpoints, time travel), sub-graphs, parallelization, map-reduce. Supervisor and
   handoff patterns are not in this course — see LangChain's multi-agent guide:
   https://docs.langchain.com/oss/python/langchain/multi-agent
2. **DeepLearning.AI — "Design, Develop, and Deploy Multi-Agent Systems with CrewAI"**
   (taught by João Moura, CrewAI founder and CEO; ~13 hr) —
   https://www.deeplearning.ai/courses/design-develop-and-deploy-multi-agent-systems-with-crewai/
   Covers: memory, tools (including MCP server integration), guardrails, execution
   hooks, Crews and Flows, traces and LLM-as-judge, CI/CD and production deployment.
   (The shorter "Multi AI Agent Systems with CrewAI" is the older intro and does not
   cover Flows or MCP.)
3. **OpenAI Agents SDK — official documentation** (first released Mar 2025; still
   pre-1.0) — https://openai.github.io/openai-agents-python/ and OpenAI's agents
   overview https://developers.openai.com/api/docs/guides/agents. No structured
   "course," but the canonical reference for OpenAI's agent primitives (agents,
   handoffs, agents-as-tools, guardrails, sessions, tracing; the Responses API is the
   default model API).

**Content to quiz on:**
- LangGraph's **supervisor pattern** — a main agent coordinates specialists (LangChain's
  current docs call this "subagents"; the standalone `langgraph-supervisor` library is
  archived and the docs recommend building the pattern directly with tool calls). When
  you build the workflow yourself with `StateGraph`, LangGraph makes the state graph
  and transitions explicit/inspectable as a first-class object. Compare: does your own
  system's orchestration logic live in
  explicit graph state, or implicitly in an orchestrator's prompt/control flow? This
  distinction — explicit state graph vs prompt-driven control flow — is the single
  most interview-useful fact from this phase.
- CrewAI's "Flows" vs "Crews" distinction — Crews are autonomous multi-agent
  collaboration (agents decide), Flows are structured, event-driven, more deterministic
  control (developer decides the sequence). Map this onto your own system: is it
  closer to a Crew or a Flow? (CrewAI's docs recommend both together: a Flow wrapping
  Crews.)
- OpenAI Agents SDK's orchestration model — a small set of primitives (agents,
  handoffs, agents-as-tools, guardrails) plus sessions and tracing; handoffs are
  LLM-driven by default (exposed to the model as tools), code-driven orchestration when
  you write it, and no separate explicit graph abstraction like LangGraph's.
- Workload identity federation — an OpenAI API platform feature, not Agents-SDK-specific:
  a workload swaps its own identity-provider token for a short-lived OpenAI access token
  instead of storing an API key. The answer to "what identity does this agent run as" —
  ties directly to Phase 8's access-governance content.
  https://developers.openai.com/api/docs/guides/workload-identity-federation

**Quiz targets:** can you, unprompted, explain what your primary tool's subagent
delegation model is analogous to and different from in LangGraph, CrewAI, AND OpenAI's
Agents SDK? This is the actual interview-readiness test — not "did you finish the
course" but "can you translate."

**Worked example (added 2026-08-12):** a generic orchestrator-workers ticket-triage
system translated across all three frameworks on the SAME scenario. LangGraph: each
lookup would be a node in an explicit `StateGraph`, edges defined upfront, state
passed as a typed object between nodes — the control flow is a first-class inspectable
artifact you could diagram without reading code. CrewAI: closer to a Flow (developer-
sequenced) with the evaluator step behaving like an autonomous Crew member inside it —
deterministic skeleton, one autonomous judgment embedded. OpenAI Agents SDK: no
separate graph object at all — the orchestration would just BE the tool-calling
control flow plus SDK sessions and handoffs, closest to how most
Claude-Code-built systems are actually structured today (logic lives in the
orchestrator's prompt, not a declared graph).

**Rubric (added 2026-08-12):**
- **Pass:** translates your own system's architecture into all 3 frameworks'
  vocabulary correctly and unprompted, in under 2 minutes each, AND states the
  MCP-vs-A2A distinction in one sentence.
- **Partial:** gets 2 of 3 frameworks right without prompting, or needs the "is the
  control flow a first-class object or prompt-driven" framing repeated.
- **Fail:** cannot map your system onto any framework's vocabulary without being
  taught the mapping fresh, or conflates MCP with A2A.

**A2A (Agent-to-Agent Protocol) — no dedicated course, awareness-level:**
Google-originated open standard, now at v1.0 and under the Linux Foundation's Agentic
AI Foundation (which also hosts MCP) — https://a2a-protocol.org, verified 2026-10-13.
Agent-to-agent communication, distinct from MCP, which is agent-to-tool; the A2A site
itself calls them complementary, not competitors. MCP: an agent discovers and calls a
tool/resource. A2A: one agent discovers and negotiates with another autonomous agent
(capability discovery, task delegation across agent boundaries, not just within one
orchestrator's workers). CrewAI (native, via the `crewai[a2a]` extra) and Microsoft
Agent Framework (successor to AutoGen and Semantic Kernel) both support it. Flag it as
"the protocol for cross-vendor/cross-org agent handoff" the way MCP is "the protocol
for tool access." Quiz target: can you state the MCP vs A2A
distinction in one sentence (tool access vs agent-to-agent handoff) without conflating
them?

**Fine-tuning (added 2026-09-09, awareness level only — no course, one paragraph):**
"Why didn't you fine-tune?" is a common interview question and needs a short, confident
answer rather than a phase. The honest one for almost every agent system: fine-tuning
changes behaviour and style, it does not add knowledge reliably, it freezes you to a
model version you then have to re-do work to leave, and it costs a data-collection
effort that prompting, retrieval and tool design usually make unnecessary. Reach for it
when you need a consistent output format or tone at very high volume and prompting has
measurably failed — and note that you cannot know prompting has failed without the eval
set from Phase 5. The wrong answer is "we didn't need it"; the right one names the
tradeoff and the measurement that would change your mind.

**Exit criteria:** you can describe your own system's architecture correctly using
LangGraph vocabulary, CrewAI vocabulary, AND OpenAI Agents SDK vocabulary in under 2
minutes each, without notes.

---

## Extra — Computer Use (no course, awareness-level only)

**Not a numbered phase — no quiz gate, no mandatory practical, no exit criteria.**
Bonus-content only, covered if time allows or you ask for it.

Computer use = agent capability to directly control a browser or desktop UI (click,
type, screenshot-read) rather than acting only through defined tool/API calls. Anthropic
ships this as a client-executed tool in the Messages API — current version
`computer_toolset_20260801`, GA with no beta header; the older `computer_20251124` /
`computer_20250124` versions need a beta header and error on Claude 5.5+. It is not part
of the Claude Agent SDK's built-in toolset. Verified 2026-10-13:
https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool
Relevant to browser-automation and legacy-UI-without-API use cases (e.g. QA
automation, workflows where no API exists).

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
