# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**DeerFlow** is a multi-agent research framework built on LangGraph that orchestrates AI agents to conduct deep research, generate reports, and create podcasts and presentations.

- **Backend**: Python 3.12+, FastAPI, LangGraph, LangChain
- **Frontend**: Next.js 15+, React 19+, TypeScript
- **Package Management**: uv (Python), pnpm (Node.js)

## Common Development Commands

### Initial Setup
```bash
# Copy configuration files
cp .env.example .env
cp conf.yaml.example conf.yaml

# Install Python dependencies
uv sync

# Install dev dependencies (required for testing)
make install-dev

# Install frontend dependencies
cd web && pnpm install
```

### Running the Application
```bash
# Console UI only
uv run main.py

# Web UI (backend + frontend)
./bootstrap.sh -d           # macOS/Linux
bootstrap.bat -d            # Windows

# Backend only (with reload)
uv run server.py --reload

# Frontend only
cd web && pnpm dev          # Runs on localhost:3000

# LangGraph Studio (for visual workflow debugging)
make langgraph-dev
```

### Testing
```bash
# Run all Python tests
make test

# Run specific test file
pytest tests/unit/config/test_configuration.py

# Run with coverage
make coverage

# Frontend tests
cd web && pnpm test:run
```

### Code Quality
```bash
# Python linting and formatting (uses Ruff)
make lint
make format

# Frontend linting and type checking
cd web && pnpm lint
cd web && pnpm typecheck
cd web && pnpm format:write

# Full frontend CI check
make lint-frontend
```

## High-Level Architecture

### Multi-Agent System
Built on LangGraph with state-based workflows:
- **Coordinator**: Entry point managing workflow lifecycle
- **Planner**: Decomposes research objectives into structured plans
- **Research Team**: Specialized agents (Researcher, Coder) executing plans
- **Reporter**: Aggregates findings and generates final reports
- **Human-in-the-loop**: Interactive plan modification and approval

### State Management
- Uses LangGraph StateGraph for agent communication
- **MemorySaver** for conversation persistence
- **Checkpointing** supports MongoDB or PostgreSQL storage (configured via `LANGGRAPH_CHECKPOINT_DB_URL`)

### Content Generation Pipeline
1. **Planning**: Planner creates research plan
2. **Research**: Researcher gathers information
3. **Processing**: Coder analyzes data/code
4. **Reporting**: Reporter synthesizes findings
5. **Post-processing**: Optional podcast/PPT generation

### Directory Structure
```
src/
├── agents/          # Agent definitions and behaviors
├── graph/           # LangGraph workflow definitions and builder
├── llms/            # LLM provider integrations (OpenAI, DeepSeek, Google, etc.)
├── prompts/         # Agent prompt templates
├── tools/           # External tools (search, TTS, Python REPL)
├── crawler/         # Web crawling and content extraction
├── server/          # FastAPI web server
├── config/          # Configuration management
└── rag/             # RAG integration for private knowledgebases

web/                 # Next.js frontend
├── src/app/         # Next.js pages and API routes
├── src/components/  # UI components
└── src/core/        # Frontend utilities and API client
```

## Configuration

### Environment Variables (.env)
Key variables to configure:
- `TAVILY_API_KEY` or `INFOQUEST_API_KEY`: Web search (required)
- `SEARCH_API`: Choose from `tavily`, `infoquest`, `duckduckgo`, `brave_search`, `arxiv`
- `LANGGRAPH_CHECKPOINT_DB_URL`: MongoDB/PostgreSQL for persistence (optional)
- `RAGFLOW_API_URL` or `QDRANT_LOCATION`: RAG integration (optional)
- `ENABLE_MCP_SERVER_CONFIGURATION`: Enable MCP (default: false, requires secured environment)
- `ENABLE_PYTHON_REPL`: Enable code execution (default: false, security risk)

### Application Configuration (conf.yaml)
- **BASIC_MODEL**: Primary LLM configuration (OpenAI, DeepSeek, Google, Ollama)
- **REASONING_MODEL**: Optional reasoning model for planning
- **SEARCH_ENGINE**: Override search engine settings
- **TOOL_INTERRUPTS**: Configure tool execution interrupts for sensitive operations
- **ENABLE_WEB_SEARCH**: Set to false to disable web search (use RAG only)

## Development Patterns

### Adding New Features
1. **New Agent**: Add agent in `src/agents/` + update graph in `src/graph/builder.py`
2. **New Tool**: Add tool in `src/tools/` + register in agent prompts in `src/prompts/`
3. **New Workflow**: Create graph builder following the pattern in `src/graph/builder.py`
4. **Frontend Component**: Add to `web/src/components/` + update API in `web/src/core/api/`

### Testing Patterns
- Unit tests mirror the `src/` structure in `tests/unit/`
- Integration tests in `tests/integration/`
- Minimum coverage threshold: 25%
- Use pytest fixtures for test setup
- Database tests use `pytest-postgresql` and `mongomock`

### LangGraph Patterns
- Agents communicate via LangGraph state (defined in `src/graph/state.py`)
- Each agent has specific tool permissions configured in prompts
- Use persistent checkpoints for conversation history
- Follow the node → edge → state pattern
- Tools are async and use proper error handling

## External Integrations

### Search Engines
- **Tavily** (default): AI-optimized search API
- **InfoQuest** (recommended): BytePlus intelligent search
- **DuckDuckGo**: No API key required
- **Brave Search**: Privacy-focused

### RAG Providers
- **RAGFlow**: Open source RAG engine
- **Qdrant**: Vector database
- **Milvus**: Vector database
- **VikingDB**: BytePlus vector database

### MCP (Model Context Protocol)
- Configurable via settings when `ENABLE_MCP_SERVER_CONFIGURATION=true`
- Requires securing the environment before enabling
- Allows integration with external MCP servers

## Important Notes

- Python 3.12+ required
- Line length: 88 characters (Ruff default)
- The project uses `uv` for Python dependency management
- Frontend uses pnpm, not npm
- Always restart server after changing `conf.yaml`
- Security features (MCP, Python REPL) are disabled by default
