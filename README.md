# ai-travel-planner
Build an AI-powered travel planning platform where:
- Users describe trips conversationally
- Multiple AI agents collaborate
- RAG retrieves travel knowledge
- MCP tools integrate external services
- Planner dynamically generates itineraries



                    ┌──────────────────────┐
                    │      Frontend        │
                    │  (Simple UI/Postman) │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │     API Gateway      │
                    │   Spring Boot (Java) │
                    └──────────┬───────────┘
                               │ REST API
                               ▼
                    ┌──────────────────────┐
                    │ AI Orchestrator API  │
                    │    FastAPI (Python)  │
                    └──────────┬───────────┘
                               │
                ┌──────────────┼──────────────┐
                ▼              ▼              ▼
        ┌────────────┐ ┌────────────┐ ┌────────────┐
        │ Intent     │ │ Itinerary  │ │ Budget     │
        │ Agent      │ │ Agent      │ │ Agent      │
        └────────────┘ └────────────┘ └────────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Mock Travel Dataset  │
                    │ JSON / Static Data   │
                    └──────────────────────┘
