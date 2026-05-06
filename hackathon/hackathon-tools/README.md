# Hackathon Tools

**Production-grade Claude Code operating system for high-velocity hackathon execution.**

A curated collection of agent configurations, skills, protocols, and project scaffolding that transforms Claude Code into a disciplined engineering co-pilot. Built from patterns refined across multiple competitive hackathons.

---

## Overview

This repository contains the full Claude Code infrastructure I use to ship production-quality software under extreme time pressure. It is not a loose collection of prompts -- it is an integrated operating system with state management, verification gates, failure recovery, and adversarial review.

The system treats every hackathon like a production deployment: structured specs, tiered verification, atomic commits, and post-mortem logging. The result is code that survives demo day and could ship to production the next morning.

---

## System Architecture

```
hackathon-tools/
├── claude-config/
│   ├── CLAUDE.md                  # Global agent operating manual
│   └── project-template.md       # Project-level CLAUDE.md template
├── skills/
│   ├── brainstorming/             # Idea-to-spec collaborative design
│   ├── writing-plans/             # Spec-to-implementation planning
│   ├── frontend-design/           # UI/UX design system skills
│   ├── debugging/                 # Structured debugging protocols
│   └── code-review/               # Adversarial review gate
├── scaffolding/
│   ├── state.json                 # Task state schema template
│   ├── memory.md                  # Memory file template
│   └── spec-template.md           # Project spec format
├── protocols/
│   ├── session-start.md           # 5-step session initialization
│   ├── task-execution.md          # 4-tier verification pipeline
│   ├── task-completion.md         # State advancement + commit
│   ├── context-pressure.md        # Context window management
│   └── adversarial-review.md      # Pre-PR review gate
└── examples/
    └── mifir-rejection-engine/    # Full worked example
```

---

## Core Protocols

### Session Start Protocol

Every session begins with 5 mandatory steps before any code is written:

| Step | Action | Purpose |
|------|--------|---------|
| 1 | Internalize Memory | Load `.claude/memory.md` -- decisions, patterns, gotchas |
| 2 | Load State | Read `tasks/state.json` -- current task/step/stage |
| 3 | Process Post-Mortems | Handle unresolved failures from prior sessions |
| 4 | Locate Current Stage | Find the active stage in the project spec |
| 5 | Check Dependencies | Verify all prerequisite stages are complete |

No exceptions. No shortcuts. The agent does not touch code until all 5 steps pass.

### Task Execution Pipeline

Every stage passes through a 4-tier verification pipeline:

```
IMPLEMENT --> TIER 1 (Build) --> TIER 2 (Simplify) --> TIER 3 (Unit) --> TIER 4 (Integration)
```

- **Tier 1 -- Build**: Deterministic build and lint commands. Must exit 0.
- **Tier 2 -- Simplify**: Code simplification agent on all changed files. Hard pre-commit gate.
- **Tier 3 -- Unit**: Unit tests targeting changed code paths.
- **Tier 4 -- Integration**: End-to-end tests with dependency spin-up and teardown.

Tiers are sequential. A tier must pass before the next runs. On failure: fix, re-run the failing tier and all previous tiers.

**3-Strike Rule**: Same tier, same root cause, three failures -- write a `verification_failure` post-mortem, commit, stop. The next session reads the post-mortem and approaches the problem fresh.

### Adversarial Review Gate

Before any PR or major task completion, a fresh subagent is dispatched with deliberately adversarial framing to break out of agreeable defaults. Every concrete issue is triaged before the PR opens. Self-review and polite review do not satisfy this gate.

### Context Pressure Rule

After each stage completes, the agent estimates remaining context window. Below 15%: write a `context_exhaustion` post-mortem with a resumption hint, commit, and stop. Never start a new stage in a depleted context window.

---

## State Management

### Task State (`state.json`)

Hierarchical state tracking: **Project > Task > Step > Stage**

```json
{
  "schema_version": "2.0",
  "current_task": "T1",
  "current_step": "T1.1",
  "current_stage": "T1.1.1",
  "tasks": {
    "T1": {
      "status": "in_progress",
      "steps": {
        "T1.1": {
          "status": "in_progress",
          "stages": {
            "T1.1.1": { "status": "complete", "completed_at": "2026-04-30T18:00:00Z" },
            "T1.1.2": { "status": "pending" }
          }
        }
      }
    }
  },
  "postmortems": []
}
```

### Memory File (`.claude/memory.md`)

Accumulated project knowledge across sessions:

```markdown
## Decisions
- [2026-04-30] Use Bun for dev server, Node for SDK compatibility

## Patterns
- [2026-04-30] All CSV loaders follow loadCSV<T>(filename, mapFn) pattern

## Gotchas
- [2026-04-30] Bun HTTP2 incompatible with @cursor/sdk -- use Node with --import tsx

## Open Questions
- [2026-04-30] Cursor SDK Agent.prompt() returns empty result field
```

---

## Skills

### Brainstorming

Collaborative design skill that turns an idea into a validated spec through structured dialogue:

1. Explore project context
2. Ask clarifying questions (one at a time, prefer multiple choice)
3. Propose 2-3 approaches with trade-offs
4. Present design section by section with approval gates
5. Write spec to `docs/superpowers/specs/`
6. Self-review for placeholders, contradictions, ambiguity
7. User review gate
8. Transition to implementation planning

### Writing Plans

Converts approved specs into staged implementation plans with deterministic verification blocks at every stage. Each stage owns 4 verify blocks (build, simplify, unit, integration) with exact shell commands.

### Adversarial Code Review

Fresh-context subagent dispatched with deliberately confrontational framing. Reviews against the original spec, not the code's self-presentation. Catches scope drift, missing edge cases, and spec violations that self-review misses.

---

## Project Spec Format

```markdown
# Project Title -- Spec

## T1 -- Task Title
**Description:** What and why.

### T1.1 -- Step Title

#### T1.1.1 -- Stage Title
**Description:** Files changed and scope.
**Requires:** T1.0.2

**Verify:**
tier1_build: <build commands>
tier2_simplify: <simplification pass>
tier3_unit: <unit test commands>
tier4_integration: <e2e test commands>
```

Every stage must own complete verify blocks. Verify commands are deterministic shell. No subjective judgment.

---

## Usage

### New Hackathon Setup

1. Copy `claude-config/CLAUDE.md` to `~/.claude/CLAUDE.md`
2. Copy `claude-config/project-template.md` to your project's `CLAUDE.md`
3. Fill in the Project-Specific Constraints table
4. Use the brainstorming skill to generate a spec
5. Use the writing-plans skill to generate an implementation plan
6. Execute -- the protocols handle the rest

### Mid-Hackathon Recovery

If a session runs out of context:
1. The agent has already written a `context_exhaustion` post-mortem
2. Start a new session -- the Session Start Protocol reads the post-mortem
3. The `resumption_hint` tells the new session exactly where to pick up

If a verification tier keeps failing:
1. After 3 strikes, the agent writes a `verification_failure` post-mortem
2. The next session reads `output_tail`, diagnoses the root cause fresh
3. No wasted context retrying the same broken approach

---

## Design Principles

| Principle | Implementation |
|-----------|---------------|
| **No code before alignment** | Brainstorming skill enforces design-first |
| **Deterministic verification** | Every stage has shell-based verify blocks |
| **Fail forward** | Post-mortems capture failures for next session |
| **Context is finite** | 15% threshold triggers graceful handoff |
| **Adversarial quality** | Review gate uses deliberately hostile framing |
| **State survives sessions** | `state.json` + `memory.md` persist everything |
| **Atomic progress** | One commit per completed stage, never mid-work |

---

## Proven Track Record

This system has been battle-tested across multiple competitive hackathons, consistently producing production-grade applications under 24-hour time constraints. The structured approach eliminates the common hackathon failure modes: scope creep, lost context, untested code, and demo-day crashes.

---

*Built for speed. Engineered for quality.*
