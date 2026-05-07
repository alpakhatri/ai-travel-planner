# api-gateway (Phase 1)

Spring Boot API gateway that:
- exposes `/v1/trips/plan`
- forwards to the Python AI engine (`ai-engine-python`) `/v1/plan`
- adds request IDs, timeouts, and (later) auth/rate limits

