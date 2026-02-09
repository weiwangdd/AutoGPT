# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

AutoGPT is a dual-license monorepo for building, deploying, and managing AI agents. It contains:

- **`autogpt_platform/`** — The main AI agent platform (Polyform Shield License)
  - **`backend/`** — Python FastAPI server (execution engine, REST/WebSocket APIs)
  - **`frontend/`** — Next.js 15 React application (agent builder UI)
  - **`autogpt_libs/`** — Shared Python utilities (auth, Redis, logging, feature flags)
  - **`db/`** — Database services (Supabase, PostgreSQL)
  - **`graph_templates/`** — Pre-built agent workflow templates
  - **`installer/`** — Installation scripts
- **`classic/`** — Original AutoGPT tools (MIT License)
  - **`forge/`** — Agent development toolkit
  - **`original_autogpt/`** — Original standalone agent
  - **`benchmark/`** — agbenchmark testing framework
  - **`frontend/`** — Flutter cross-platform UI
- **`docs/`** — Documentation site content

## Component-Specific Guidance

Each major component has its own CLAUDE.md with detailed commands and patterns:

- **Platform root**: See `autogpt_platform/CLAUDE.md`
- **Backend**: See `autogpt_platform/backend/CLAUDE.md` for commands, architecture, block development
- **Frontend**: See `autogpt_platform/frontend/CLAUDE.md` for commands, component patterns, data fetching
- **Frontend testing**: See `autogpt_platform/frontend/src/tests/CLAUDE.md` for test strategy
- **Docs**: See `docs/CLAUDE.md` for documentation conventions

## Essential Commands

### Backend (from `autogpt_platform/backend/`)

```bash
poetry install                    # Install dependencies
poetry run app                    # Run all backend services
poetry run rest                   # Run REST server only
poetry run ws                     # Run WebSocket server only
poetry run executor               # Run execution engine only
poetry run test                   # Run all tests
poetry run pytest path/to/test.py # Run specific test
poetry run format                 # Format code (Black + isort)
poetry run lint                   # Lint code (Ruff)
poetry run prisma migrate dev     # Run database migrations
```

### Frontend (from `autogpt_platform/frontend/`)

```bash
pnpm i                # Install dependencies
pnpm dev              # Start dev server
pnpm build            # Build for production
pnpm test             # Run Playwright E2E tests
pnpm storybook        # Run Storybook
pnpm generate:api     # Regenerate API client from OpenAPI spec
pnpm format           # Format code (Prettier)
pnpm types            # Type-check (TypeScript)
```

### Platform (from `autogpt_platform/`)

```bash
make start-core       # Start infrastructure (Supabase, Redis, RabbitMQ)
make stop-core        # Stop infrastructure
make migrate          # Run database migrations + generate Prisma client
make format           # Format both backend and frontend
make run-backend      # Run backend server
make run-frontend     # Run frontend dev server
make reset-db         # Reset database (destructive)
make init-env         # Copy .env.default files to .env
```

### Docker

```bash
# From autogpt_platform/
docker compose up -d              # Start all services
docker compose up -d deps         # Start only infrastructure (DB, Redis, RabbitMQ, ClamAV)
```

## Tech Stack

| Layer | Backend | Frontend |
|-------|---------|----------|
| **Language** | Python 3.10+ | TypeScript (Node 22.x) |
| **Framework** | FastAPI + Uvicorn | Next.js 15 App Router |
| **Package Manager** | Poetry | pnpm 10.20.0 |
| **Database** | PostgreSQL + Prisma ORM (pgvector) | — |
| **Queue** | RabbitMQ (aio-pika) | — |
| **Cache** | Redis | — |
| **Auth** | Supabase JWT | Supabase JWT |
| **API Client** | — | Orval (auto-generated React Query hooks) |
| **State** | — | React Query (server), Zustand (UI) |
| **UI** | — | Tailwind CSS + shadcn/ui (Radix) |
| **Icons** | — | Phosphor Icons only |
| **Testing** | pytest (snapshot, asyncio, mock) | Playwright (E2E), Vitest (unit/integration), Storybook |
| **LLM Providers** | Anthropic, OpenAI, Groq, Ollama, LiteLLM | — |
| **Monitoring** | Prometheus, Sentry, Langfuse, PostHog | Sentry, PostHog |
| **Feature Flags** | LaunchDarkly | LaunchDarkly |

## Architecture

### Key Concepts

1. **Agent Graphs** — Workflow definitions stored as JSON, composed of connected blocks
2. **Blocks** — Reusable components in `backend/backend/blocks/` (45+ blocks: LLM, HTTP, email, Reddit, YouTube, media, code execution, etc.)
3. **Executor** — Async execution engine that processes agent workflows with cluster-wide coordination
4. **Integrations** — OAuth and API credential management per user
5. **Store** — Marketplace for sharing and discovering agent templates
6. **CoPilot** — Chat-based agent interaction with workspace file support

### Backend Services (microservices via Docker)

- `rest_server` — REST API (port 8006)
- `websocket_server` — Real-time agent status updates
- `executor` — Async agent execution engine
- `scheduler_server` — Scheduled/recurring task execution
- `notification_server` — User notification delivery
- `database_manager` — Database connection management

### API Feature Modules

Routes are organized in `backend/backend/api/features/`:
`admin`, `builder`, `chat`, `executions`, `integrations`, `library`, `otto`, `postmark`, `store`, `workspace`

### Database Schema (Prisma)

Key models in `backend/schema.prisma`:
- `User` — Auth and profile with notification preferences
- `AgentGraph` — Workflow definitions with versioning
- `AgentGraphExecution` — Execution history and results
- `AgentNode` — Individual nodes in workflows
- `StoreListing` — Marketplace agent listings
- `ChatSession` — Agent conversation history
- `LibraryAgent` — Published agents in library
- `IntegrationWebhook` — Webhook configurations

## Environment Configuration

### Files

- **Backend**: `autogpt_platform/backend/.env.default` → `.env`
- **Frontend**: `autogpt_platform/frontend/.env.default` → `.env`
- **Platform**: `autogpt_platform/.env.default` → `.env`

### Loading Order (Docker)

1. `.env.default` — Base defaults (tracked in git)
2. `.env` — User overrides (gitignored)
3. Docker Compose `environment:` sections
4. Shell environment variables (highest precedence)

## Code Quality & Pre-commit Hooks

Pre-commit hooks enforce:
- **Python**: Ruff (lint + autofix), Black (format), isort (imports), Pyright (type-check)
- **Frontend**: Prettier (format), TypeScript (`pnpm types`)
- **Security**: `detect-secrets` for high-entropy strings (pre-push)
- **General**: Large file check (>500KB), merge conflict detection, Prisma client generation

## Development Conventions

### Commit Messages (Conventional Commits)

Format: `type(scope): description`

**Types**: `feat`, `fix`, `refactor`, `ci`, `docs`, `dx`

**Scopes**: `platform`, `frontend`, `backend`, `infra`, `blocks`

**Subscopes**: `backend/executor`, `backend/db`, `frontend/builder`, `infra/prod`

### Pull Requests

- Target the `dev` branch
- Follow conventional commit format for PR title
- Fill out `.github/PULL_REQUEST_TEMPLATE.md`
- Run pre-commit hooks before submitting
- Use squash merges

### Backend Conventions

- All commands require `poetry run ...` prefix
- Test files are colocated with source: `module.py` → `module_test.py`
- New blocks go in `backend/backend/blocks/`, inherit from `Block`, define `BlockSchema` for I/O
- Use `store_media_file()` with appropriate `return_format` for file handling in blocks
- API routes live in `backend/backend/api/features/`
- Snapshot testing for API responses (review diffs before committing)

### Frontend Conventions

- Client-first approach (server components only for SEO/TTFB)
- Generated API hooks via Orval — do NOT use deprecated `BackendAPI` or `src/lib/autogpt-server-api/*`
- Separate render logic (`.tsx`) from business logic (`use*.ts` hooks)
- Component structure: `ComponentName/ComponentName.tsx` + `useComponentName.ts` + `helpers.ts`
- Design system components from `src/components/` (atoms, molecules, organisms) — never use `__legacy__`
- Phosphor Icons only — no other icon libraries
- Function declarations for components/handlers (not arrow functions)
- No `useCallback`/`useMemo` unless optimizing a measured performance issue
- Fully capitalize acronyms: `graphID`, `useBackendAPI`
- Props type: `type Props = { ... }` (not exported unless needed externally)

### Reviewing/Revising Pull Requests

```bash
# Get PR reviews
gh api /repos/Significant-Gravitas/AutoGPT/pulls/{pr_number}/reviews

# Get review comments
gh api /repos/Significant-Gravitas/AutoGPT/pulls/{pr_number}/reviews/{review_id}/comments

# Get PR comments
gh api /repos/Significant-Gravitas/AutoGPT/issues/{pr_number}/comments
```

## Testing

### Backend

- **Framework**: pytest with pytest-asyncio, pytest-mock, pytest-snapshot
- **Run**: `poetry run test` or `poetry run pytest path/to/test.py::test_name`
- **Block tests**: `poetry run pytest backend/blocks/test/test_block.py -xvs`
- **Snapshots**: `poetry run pytest path/to/test.py --snapshot-update` (review diffs!)
- **Auth fixtures**: `mock_jwt_user`, `mock_jwt_admin` from `conftest.py`

### Frontend

- **E2E**: Playwright — `pnpm test` (centralized in `src/tests/*.spec.ts`)
- **Integration**: Vitest + React Testing Library — `__tests__/` folders next to components
- **Unit**: Vitest — colocated `Component.test.tsx` files
- **Visual**: Storybook + Chromatic — `Component.stories.tsx` files
- **API mocking**: MSW with auto-generated handlers from Orval

## Licensing

- `autogpt_platform/` — Polyform Shield License (CLA required for contributions)
- Everything else — MIT License
