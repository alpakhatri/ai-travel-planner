# Phase 3 Implementation Roadmap (Agentic AI)

## Phase 3 objective
Evolve from “single planner” into a **multi-agent planning system** with reliable **orchestration** and **memory**, while keeping the platform debuggable through stored run artifacts and evaluations.

## Responsibilities (matches `README.md`)
1. multiple specialist agents collaborate (Flight/Hotel/Local/Budget/Itinerary/Support)
2. orchestration via LangGraph (fan-out/fan-in, tool routing + guardrails, re-planning loops)
3. memory
   - run memory (always-on): persist artifacts (inputs, extracted constraints, retrieval sets, tool calls, outputs)
   - user memory (opt-in): long-term preferences + short-term context across turns
4. quality gates: feasibility checks, budget consistency, and “open questions” when constraints are missing

## Key deliverables
- **Agent interfaces**: consistent input/output contracts per agent (no “free-form text as state”).
- **Trip State**: single shared state object passed across the graph (versioned schema).
- **Graph execution**:
  - fan-out: Flight/Hotel/Local can run in parallel
  - fan-in: Budget merges costs + constraints; Itinerary assembles final plan
  - re-plan: Support agent triggers partial recomputation when user changes constraints
- **Memory model**:
  - run artifacts persisted per node (inputs/outputs/tool calls/timing)
  - user profile store (opt-in) + retrieval into Trip State per run
- **Evaluation harness**:
  - golden prompts for regression
  - schema validation + feasibility checks as automated gates

## Suggested milestones
### Milestone 1 — Introduce specialist agents (1–2 weeks)
- Add Flight/Hotel/Local/Support agents (may still use stub tools)
- Define per-agent contracts and state transitions

Acceptance criteria:
- Graph produces a schema-valid `TripPlan` with contributions from multiple agents.

### Milestone 2 — Orchestration upgrades (1–2 weeks)
- Add fan-out/fan-in structure
- Add tool routing policy (which agent can call which tool and when)
- Add re-planning loop for “change requests”

Acceptance criteria:
- A “change” request updates only the affected parts of the plan (not full recompute), and artifacts show the minimal recomputation path.

### Milestone 3 — Memory (1–2 weeks)
- Always-on run memory (already started in Phase 1) expanded to per-node artifacts + timings
- Opt-in user memory:
  - store preferences (pace, budget style, dietary needs)
  - inject into Trip State at run start

Acceptance criteria:
- Given the same user profile, the planner adapts outputs predictably (e.g., vegetarian preference influences restaurant picks).

### Milestone 4 — Quality gates (ongoing)
- Add feasibility checks (overlaps, travel time sanity, duplicates)
- Budget consistency validation
- Open-question generation when constraints missing

Acceptance criteria:
- Invalid plans are rejected with actionable errors (and the system asks clarifying questions instead).

