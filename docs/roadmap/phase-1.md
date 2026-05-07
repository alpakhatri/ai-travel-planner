# Phase 1 Implementation Roadmap (Foundation)

## Phase 1 objective
Deliver an end-to-end system where a user can submit a natural-language trip request and receive a **structured** itinerary (`TripPlan`) produced by a **multi-step agent workflow**, with all inputs/outputs persisted for debugging and iteration.

**Constraints**
- No real travel booking yet (use mock datasets and stub tools).
- Keep contracts stable; optimize for correctness, traceability, and debuggability.

## Exit criteria (Definition of Done)
- A user can call one endpoint and receive a validated `TripPlan` JSON response.
- The system stores **run artifacts**: raw input, extracted `TripRequest`, final `TripPlan`, model config, tool calls (even if stubbed).
- The Java gateway and Python AI engine are independently runnable and have health checks.
- Basic error handling exists (invalid request, model failure, timeout).

## Milestone 0 — Repo scaffolding (0.5 day)
Create folders and placeholder READMEs:
- `backend-java/` (Spring Boot gateway)
- `ai-engine-python/` (FastAPI + LangGraph)
- `docs/` (architecture + roadmap)
- `infra/` (docker-compose for local)
- `frontend/` (placeholder)

## Milestone 1 — Data contracts first (0.5–1 day)
Define JSON schemas (or Pydantic models) for:
- `TripRequest`
- `TripPlan`
- `RunArtifacts`

Acceptance criteria:
- `TripPlan` can be rendered deterministically by a frontend without parsing prose.
- Validation errors are user-readable (e.g., missing dates/budget).

Suggested minimal shapes:
- `TripRequest`: `origin`, `destination`, `start_date?`, `end_date?`, `days`, `budget`, `currency`, `party_size`, `preferences[]`, `constraints[]`
- `TripPlan`: `summary`, `assumptions[]`, `open_questions[]`, `budget_breakdown`, `days[]` (each day has `blocks[]`), `alternatives[]`

## Milestone 2 — Python AI engine (FastAPI) skeleton (1–2 days)
Endpoints (internal, behind gateway):
- `GET /healthz`
- `POST /v1/plan`
  - input: `{ message: string, user_id?: string, trip_id?: string }`
  - output: `{ run_id: string, trip_request: TripRequest, trip_plan: TripPlan }`

Implementation tasks:
- Pydantic models for contracts
- Run persistence in Postgres:
  - `runs` table: `run_id`, timestamps, status, model config
  - `run_artifacts` table (or JSONB columns): input, trip_request, trip_plan, tool_calls
- Simple “LLM adapter” interface (OpenAI/Ollama later) so you can swap providers without rewriting agents

Acceptance criteria:
- `POST /v1/plan` works locally with a mocked LLM response (even before real models).
- Every request creates a run record.

## Milestone 3 — Agent orchestration (LangGraph) for Phase 1 (2–4 days)
Phase 1 agents (keep it minimal):
- **Intent Agent**: extract constraints → `TripRequest`
- **Budget Agent**: compute budget bands → `BudgetPlan` (can be embedded inside `TripPlan`)
- **Itinerary Agent**: generate `TripPlan` (structured) + rationale text

Graph design:
- Deterministic sequence: `Intent -> Budget -> Itinerary`
- Single “Trip State” object passed between nodes

Acceptance criteria:
- Same input yields schema-valid output; failures are captured in `RunArtifacts`.
- `TripPlan.days[].blocks[]` never overlaps within a day (basic feasibility check).

## Milestone 4 — Java API Gateway (Spring Boot) (1–2 days)
Public endpoints (frontend-facing):
- `GET /healthz`
- `POST /v1/trips/plan`
  - forwards to Python `POST /v1/plan`
  - adds request IDs, auth context (stub), and timeout budgets

Platform patterns to demonstrate (even in Phase 1):
- Request IDs propagated from gateway → AI engine
- Timeouts (gateway <-> AI engine)
- Idempotency key support (optional in Phase 1; required later)

Acceptance criteria:
- Calling the Java endpoint returns the same response as calling Python directly.

## Milestone 5 — Local infra (Docker Compose) (1 day)
`infra/docker-compose.yml` runs:
- Postgres
- Java gateway
- Python AI engine

Acceptance criteria:
- `docker compose up` brings up everything; health checks pass.

## Milestone 6 — Minimal frontend (optional Phase 1) (1–2 days)
Ship either:
- a tiny web UI (chat input + itinerary renderer), or
- keep “client” as Postman/curl, but include a JSON renderer script.

Acceptance criteria:
- It’s easy for a reviewer to try one request and see an itinerary.

## What Phase 1 intentionally does NOT include
- RAG ingestion + vector DB (Phase 2)
- MCP tool registry + real API integrations (Phase 4)
- Kafka/event-driven refreshes (Phase 5)
- Advanced memory/personalization (Phase 3+)

## Suggested “demo script” for your README
1. Start platform locally.
2. Call `POST /v1/trips/plan` with the Japan request.
3. Show returned `TripPlan` JSON + rendered view.
4. Show `runs` table row and stored artifacts for that run.

