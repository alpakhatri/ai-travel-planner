# AI Travel Planner Platform

Build an AI-powered travel planning platform where:
- Users describe trips conversationally
- Multiple AI agents collaborate
- RAG retrieves travel knowledge (Phase 2)
- MCP tools integrate external services (Phase 4)
- Planner dynamically generates itineraries

## Problem statement
Travel planning is high-effort: it requires reconciling constraints (budget, pace, interests, logistics) and rapidly re-planning as assumptions change. This project builds a platform that turns a conversational request into a **structured**, **debuggable**, and eventually **tool-verified** itinerary.

## Final user experience (target)
Example:
> Plan a 5-day Japan trip under $3000 focused on food and culture.

System (target state):
- finds flights and hotels
- builds a day-by-day itinerary
- optimizes budget allocations
- recommends restaurants and local experiences
- adapts dynamically to changes

## Architecture
- **Phase 1 (Foundation)**: Java API Gateway → Python AI Engine (FastAPI + LangGraph) → Postgres
- **Later**: RAG + MCP tool integrations + event-driven refreshes

Start here:
- Phase 1 architecture: `docs/architecture/phase-1.md`
- Target-state architecture: `docs/architecture/target-state.md`
- Phase 1 roadmap: `docs/roadmap/phase-1.md`

## Core system ideas 
- **Contracts-first**: the system of record is **structured JSON** (`TripRequest`, `TripPlan`), not prose.
- **Run artifacts**: persist inputs/outputs/tool calls per run for debugging and evaluation.
- **Clear boundaries**: Java owns gateway concerns (auth, quotas, timeouts); Python owns AI orchestration.
- **Scale-ready patterns** (later): retries, idempotency, caching, event-driven recomputation, observability.

## Agent layer (target design)
Agents collaborate via an orchestrator (LangGraph recommended):
- **Flight Agent**: flight search + alternatives
- **Hotel Agent**: location vs budget optimization
- **Budget Agent**: allocations and constraint enforcement
- **Itinerary Agent**: day-by-day sequencing + feasibility checks
- **Local Experience Agent**: food, culture, hidden gems
- **Support/Replan Agent**: change handling and incremental re-planning

Phase 1 uses a reduced set (Intent/Budget/Itinerary) to build the platform foundation.

## RAG pipeline (Phase 2)
Use RAG for travel guides, visa rules, seasonal tips, local recommendations:
1. ingest docs/web data
2. chunk + embed
3. store in vector DB
4. retrieve per question
5. inject context + citations into agent reasoning


## Agentic AI (Phase 3)
Add true multi-agent planning with orchestration + memory:
1. multiple specialist agents collaborate (Flight/Hotel/Local/Budget/Itinerary/Support)
2. orchestration via LangGraph (fan-out/fan-in, tool routing + guardrails, re-planning loops)
3. memory
   - run memory (always-on): persist artifacts (inputs, extracted constraints, retrieval sets, tool calls, outputs)
   - user memory (opt-in): long-term preferences + short-term context across turns
4. quality gates: feasibility checks, budget consistency, and “open questions” when constraints are missing

## MCP design (Phase 4)
MCP tools provide structured tool invocation for external services:
- Maps / routing
- Flights / hotels search
- Weather
- Currency FX

## Scalability considerations (Phase 5)
- async workflows and background recomputation
- caching (Redis), rate limiting, retries with backoff
- idempotency keys for plan/booking operations
- event-driven updates (Kafka) for “booking confirmed → refresh plan”
- observability: run traces, tool-call logs, cost/latency metrics

## Tradeoffs (early choices)
- **Start with stubs/mocks** rather than real booking flows to keep iteration fast.
- **Prefer deterministic orchestration** (simple graph) before adding parallel agents.
- **Persist everything** (artifacts) even if it costs more storage—debuggability wins early.

## Repo structure
```text
ai-travel-planner/
├── backend-java/
│   └── api-gateway/
├── ai-engine-python/
│   ├── agents/
│   ├── orchestrator/
│   ├── memory/
│   ├── rag/   (Phase 2+)
│   └── mcp/   (Phase 4+)
├── frontend/
├── docs/
│   ├── architecture/
│   └── roadmap/
└── infra/
```

## Roadmap
- **Phase 1**: end-to-end plan generation with persistence (see `docs/roadmap/phase-1.md`)
- **Phase 2**: RAG knowledge base + citations
- **Phase 3 (Agentic AI)**: multiple agents + orchestration + memory
  - **Multiple agents**: add specialist agents (Flight/Hotel/Local Experience/Support) collaborating via a shared Trip State
  - **Orchestration**: move from a linear workflow to a graph with fan-out/fan-in, tool-routing, and re-planning loops (LangGraph)
  - **Memory**
    - **Run memory (always-on)**: persist artifacts per run (inputs, extracted constraints, retrieval sets, tool calls, outputs) for debugging/evals
    - **User memory (opt-in)**: preferences and long-term profile (pace, budget style, dietary needs) + short-term trip context across turns
- **Phase 4**: MCP tool registry + external API integrations
- **Phase 5**: production patterns (Redis, Kafka, retries, observability)
