# agentic-coach

A [Claude Code](https://code.claude.com/docs) **skill** that turns
Claude into a strict, Socratic coach for agentic-AI-engineering fundamentals — the
skills that separate "I've prompted an LLM" from "I can architect, evaluate, secure,
and ship a production agent system." It runs structured drills, one curriculum phase
at a time, reviews the practical work you submit, and keeps a persistent progress log
across sessions.

This isn't a general "ask Claude about agents" prompt. It's an 11-phase, opinionated
curriculum covering pattern vocabulary, prompt/context engineering, MCP, Agent Skills,
RAG, eval design, observability, model routing, security, comparative framework
literacy (LangGraph/CrewAI/OpenAI Agents SDK), and habit correction — each phase
backed by real course links or published resources, last verified 2026-10-13.

## What this is

Three files that work together:

- [`SKILL.md`](SKILL.md) — the coach's operating instructions, in Claude Code's Skill
  format (YAML frontmatter + Markdown).
- [`curriculum.md`](curriculum.md) — the full course content: what to learn per phase,
  what gets quizzed, and the mandatory practical task for each phase. This is the
  coach's answer key; it also works as a standalone self-study reference if you never
  install the skill.
- `progress.md` — a running log the coach appends to after every session: what you
  covered, quiz results, practical-task progress, and recurring gaps. **This file is
  gitignored** — it's yours, stays local, and is never part of what you'd fork or
  share. [`progress.example.md`](progress.example.md) is the blank template it starts
  from.

## How to use it

1. **Install the skill.** Copy `SKILL.md`, `curriculum.md`, and `progress.example.md`
   into a folder named `agentic-coach` under your Claude Code skills directory
   (typically `~/.claude/skills/agentic-coach/`).
2. **Start a session.** In Claude Code, say `/agentic-coach`, "drill me on agents," or
   "agentic engineering session." On first run, the coach copies
   `progress.example.md` to `progress.md` and starts you at Phase 0.
3. **Work through phases in order.** Each phase has real course/resource links plus a
   quiz bank. The coach confirms you've engaged with the material (self-report — it
   can't verify external course completion), then quizzes you Socratically, escalating
   hints only when you're genuinely stuck. It won't hand you the answer on a first
   miss.
4. **Complete the practical task.** Most phases require a real, mandatory artifact —
   an MCP server you built, an eval report with actual numbers, a tracing
   implementation, a tool-access audit — not just passing a quiz. The coach reviews
   and pushes back; it does not write the work for you.
5. **Session ends with a log entry.** The coach appends to `progress.md`: phase
   worked, quiz results, practical-task progress, and any recurring habit gaps (see
   Phase 10). This is what makes the next session pick up where you left off.

## What to expect

- **Strict, not a tutorial.** Built to behave like a demanding technical reviewer, not
  a patient explainer. Doesn't lecture — asks, and pushes you to articulate the "why."
- **No answers on demand.** Tiered hints exist specifically to make you work for each
  level before the concept is explained plainly.
- **Practical tasks are mandatory, not optional.** A phase isn't complete on quiz
  performance alone — most require a real artifact.
- **State lives in plain Markdown you own.** No database, no hidden config. Read,
  edit, or delete your own progress log at any time.
- **Phase 10 (habit correction) has no fixed answer key.** It's a methodology the
  coach applies by reviewing *your* actual session history or self-report — three
  categories of habit to watch for, drilled live during the other phases' practical
  work, not lectured as abstract advice.

## Is this the right fit for you?

Good fit if you:
- Already have some hands-on experience building something with an LLM — even a rough
  side project — and want to formalize scattered knowledge into interview-ready
  vocabulary and production practice.
- Want drilling and real practical output, not another article to passively read.
- Are disciplined enough to keep `progress.md` updated (or fine with the coach nudging
  you to).

Probably not a fit if you:
- Have never built anything with an LLM before — this assumes you bring a real project
  or are willing to build one alongside the curriculum.
- Want a lenient, encouraging tone — this is deliberately terse and demanding.
- Don't use Claude Code (this is a Claude Code Skill, not a standalone app).

## Adapting it

Everything is plain Markdown — fork it and edit `curriculum.md` or `SKILL.md` directly
to:
- Change the tone (less strict, more explanatory).
- Add, remove, or reorder phases for a different target role or stack.
- Swap the primary-tool bias (this ships biased toward Claude Code / Claude Agent SDK
  — change the framing in `SKILL.md`'s persona paragraph if yours differs).
- Change hint tiers, session-end format, or practical-task requirements.

Course/resource URLs were last verified 2026-10-13. This field moves fast —
if a link is dead, search for the resource's current location rather than assuming the
content itself is gone.

## License

MIT — use, fork, and modify freely.
