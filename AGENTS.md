# AGENTS.md

> **Single source of truth for AI coding assistants.**
> This file is read by all harnesses (Claude Code, Cursor, Aider, Gemini, etc.).
> English version. Русская версия: [AGENTS.ru.md](AGENTS.ru.md).

---

### What is this project?

**NoOversight (Quorum)** — a production-ready multi-agent AI platform. Several LLM agents
(Claude, GPT, Gemini, Grok) collaborate on complex tasks via a unified OpenRouter API.
An intelligent orchestrator delegates subtasks to specialized agents and synthesizes their
answers. Architecture is streaming-first, event-driven, with real-time feedback over
WebSocket and SSE.

### Read the `docs/` folder FIRST — do not scan the whole repo

Before exploring source code, read the analysis documents in [`docs/`](docs/). They are the
single source of truth and were written specifically to avoid blind full-repo walks:

| File | What it contains |
|---|---|
| [`docs/01_project_structure.md`](docs/01_project_structure.md) | Full directory tree, every key file described, dependencies, tech stack |
| [`docs/02_architecture.md`](docs/02_architecture.md) | Layered architecture, design patterns, data flow, DB schema, security |
| [`docs/03_execution_flow.md`](docs/03_execution_flow.md) | App lifecycle, business processes, routing, WebSocket protocol, error handling, logging |
| [`docs/04_code_quality.md`](docs/04_code_quality.md) | SOLID/DRY/KISS audit, tech debt, dead code, code smells, bottlenecks, security gaps |
| [`docs/05_optimization_roadmap.md`](docs/05_optimization_roadmap.md) | Prioritized improvements, performance fixes, refactor plan, phases, estimates |

**Workflow for any task:**
1. Find the relevant section in `docs/` (structure/architecture/flow/quality/roadmap).
2. Only then open the specific source files mentioned there.
3. Do not recursively list or read the entire `backend/src/` or `frontend/src/` — the tree
   is already documented in `docs/01`.

### Tech stack (brief)

- **Backend:** Python 3.13+, FastAPI, async/await, LangChain (`langchain-openai` → OpenRouter),
  SQLAlchemy 2.0 (async), Alembic, PostgreSQL 13+ with pgvector, structlog, WebSocket + SSE.
- **Frontend:** React 18, TypeScript 5.5, Vite 5, Zustand 4 (slices + normalized state),
  Tailwind CSS 3, Framer Motion, react-markdown.
- **External:** OpenRouter (LLM access), OpenAI (embeddings), DuckDuckGo/Tavily/SerpAPI (web search).

### Project layout (top level)

```
Quorum/
├── backend/    # FastAPI backend (Python, async) — see docs/01 for full tree
├── frontend/   # React 18 + TypeScript SPA — see docs/01 for full tree
├── docs/       # Analysis documents (read these first)
├── scripts/    # Shell scripts to start/stop services
├── Makefile    # Root: install, start, stop, dev, test, lint, build
└── pyproject.toml
```

### Conventions and coding style

- **Backend layers:** `api/` (routes) → `core/` (business logic, orchestrator) →
  `agents/` (LLM clients) → `infrastructure/` (DB, WS, logging, tracking) → `tools/`.
  Respect the layers; do not let routes call repositories for cross-cutting logic — use services.
- **Async everywhere** on the backend. Use `async def`, `AsyncSession`, `asyncio.gather`.
  Never block the event loop.
- **Pydantic** for all request/response models and settings (`pydantic-settings`).
- **Repositories** take an `AsyncSession` passed from outside; they do not manage session lifecycle.
- **Singletons** are module-level globals (`db_manager`, `connection_manager`, `settings`,
  `get_token_manager()`, `get_settings_service()`, `get_tool_registry()`).
- **Frontend state:** Zustand with slices + normalized state (`byId`/`allIds`). All WS events
  flow through a single `handleStreamEvent` in `streamSlice.ts` (event sourcing).
- **Naming:** backend `snake_case`, frontend `camelCase` (Pydantic `alias_generator = to_camel`).
- **Logging:** structured (`structlog` backend, custom `Logger` frontend) with correlation IDs.
  Sensitive data (API keys, tokens) is redacted automatically — never log raw secrets.

### Running and testing

```bash
make install     # install backend + frontend deps
make dev         # start both services (backend :8000, frontend :5173)
make test        # backend pytest
make lint        # backend + frontend linters
make build       # frontend production build
```

Backend tests: `cd backend && pytest`. Frontend has no test script yet (see `docs/05`).

### Known gotchas (details in `docs/04`)

- `ARCHITECTURE.md` and `README.md` are partly out of date (mention LiteLLM / endpoints that
  do not exist). Trust `docs/` over them.
- `health.py` references non-existent settings fields — health check is misleading.
- `_execute_sub_agents` (parallel) is dead code; the live path is sequential
  `_run_agent_conversation`.
- Tool calling is declared (schemas passed to ChatOpenAI) but **not executed** —
  `BaseAgent.execute_tool()` is never called from the stream loop.
- `AgentType` enum values do not match `MODEL_MAP` model IDs.
- Token usage is in-memory only — lost on restart.
- No auth, no rate limiting — do not deploy publicly without `docs/05` Phase 3.

### Rules for AI assistants

1. **Read `docs/` first**, open source files only as needed.
2. Make **minimal** changes; follow existing layering and naming.
3. Keep code async on the backend; never block.
4. Do not edit `docs/` unless explicitly asked — it is an analysis snapshot (August 2026).
5. Run `make test` / `make lint` after backend changes when feasible.
6. Do not commit or push without explicit user confirmation.
