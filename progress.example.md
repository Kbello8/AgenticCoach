# Agentic Engineering Coach — Progress Tracker (template)

Copy this file to `progress.md`. The coach reads and appends to `progress.md`, not
this file. Leave it mostly empty at the start — the coach fills it in after each
session.

## Current state
- **Active phase:** 0 — Pattern vocabulary
- **Phases:** 0 (vocab) → 1 (prompt & context engineering) → 2 (Claude Code/Agent SDK +
  MCP) → 3 (Agent Skills) → 4 (RAG & retrieval/vector DB) → 5 (eval design, priority
  gap) → 6 (observability, no course) → 7 (model routing & cost-tiering) → 8 (security
  & access governance) → 9 (LangGraph/CrewAI/OpenAI Agents SDK literacy, incl. A2A
  protocol) → 10 (habit correction, runs alongside 2-8, not standalone). Extra
  (non-gated, bonus, sits between Phase 9 and 10): Computer Use.
- **Status per phase:** `[ ]` not started, `[~]` in progress, `[x]` exit criteria met
- **Session order (differs from phase numbers — numbers stay fixed, this is purely
  scheduling):** consider running 5 and 6 before 7-9 — both are the phases most likely
  to produce a capstone-ready artifact (quantified eval set, per-agent tracing), so
  it's worth doing them while momentum's fresh rather than after three
  literacy/awareness-heavy phases that don't produce capstone artifacts. Full
  suggested order: 0 → 1 → 2a → 2b → 3 → 2c → 4 → 5 → 6 → 7 → 8 → 9 → 10(ongoing) →
  Extra(if time).
- **Phase 2 split:** Phase 2 is heavy — run as three sessions, same phase number, not
  marked `[x]` until all three are done:
  - **2a** — courses (Claude Code 101, Claude Code in Action, Intro to MCP) + build the
    MCP server practical + AGENTS.md content.
  - **2b** — planning methodology (PRD/decomposition/tracer-bullet) + test-loop
    discipline (CI/pre-commit/red-green-refactor) + autonomous/HITL loop design.
  - **2c** — Claude Projects, the claude.ai surface (added 2026-09-03). Scheduled after
    Phase 3 because it depends on both 2a's MCP content and Phase 3's Skills content.

| Phase | Est. sessions | Notes |
|---|---|---|
| 0 — Pattern vocabulary | [ ] 1 | No course — resource read + quiz only |
| 1 — Prompt & context engineering | [ ] 1-2 | Practical: rewrite a real prompt applying 3+ techniques |
| 2a — Claude Code/SDK + MCP (courses + build) | [ ] 2 | Practical task: build one MCP server |
| 2b — Planning/test-loop/HITL | [ ] 1-2 | No separate practical, informs later practicals |
| 3 — Agent Skills | [ ] 1-2 | Practical: author one real Skill |
| 2c — Claude Projects (claude.ai surface) | [ ] 1 | Added 2026-09-03. Run after Phase 3. Practical: build one real Project, document where it breaks |
| 4 — RAG & retrieval/vector DB | [ ] 2 | Practical: one embedding-based retrieval example |
| 5 — Eval design | [ ] 2-3 | PRIORITY for most learners. Practical task: ground-truth eval set for your review stage. Consider running before 7-9 |
| 6 — Observability | [ ] 2 | Practical task: per-agent-step tracing. Consider running before 7-9 |
| 7 — Model routing & cost-tiering | [ ] 1 | Practical: apply tiered routing to one real task, document before/after cost |
| 8 — Security & access governance | [ ] 1-2 | Practical: tool-access blast-radius audit |
| 9 — Comparative literacy | [ ] 2 | LangGraph + CrewAI + OpenAI Agents SDK vocabulary mapping + A2A protocol |
| Extra — Computer Use | [ ] 0-1 | No course, awareness-level, no exit criteria, cover only if time allows |
| 10 — Habit correction | [ ] ongoing | Runs alongside 2-8 — track recurrence, not a separate session |

**Total estimate:** ~19-24 sessions.

## Recall Queue (spaced repetition)

Every concept taught (not just missed ones) gets logged here on a box schedule, so
retention gets tested over time instead of assumed once a phase closes `[x]`.

**Box schedule:**
- **Box 1** (new, or dropped back after a miss) → re-quiz next session.
- **Box 2** (1 clean recall) → re-quiz ~2 phases later.
- **Box 3** (2 clean recalls) → re-quiz ~4 phases later.
- **Box 4** (3 clean recalls) → retired, no more forced review.
- Any miss at ANY box → drop back to Box 1, don't just decrement.

**Session behavior:** warm-up pulls the 1-2 oldest-due items from this table before
new-phase content begins (see SKILL.md drill protocol).

| Concept | Phase taught | Box | Last tested | Next due | Notes |
|---|---|---|---|---|---|
| (populate as concepts are taught — one row per concept, not per phase) | | | | | |

## Recurring gaps (running list)
-

## Session log

### YYYY-MM-DD
- Phase worked:
- Quiz results:
- Practical-task progress:
- Recurring gaps / Phase 10 habit recurrence:
- Next session focus:
