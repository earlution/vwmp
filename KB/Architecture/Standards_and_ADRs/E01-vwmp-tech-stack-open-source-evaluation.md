# VWMP Technology Stack Evaluation for Standalone Open-Source Project

**Status:** Draft - Evaluation
**Epic:** Epic 01 - Visual Workflow Management Platform (VWMP)
**Created:** 2025-11-21
**Version:** v0.21.0.41+9

---

## Executive Summary

This document evaluates technology stack options for VWMP as a standalone open-source project, considering developer experience, maintenance burden, community adoption, deployment simplicity, and framework independence. **Django is NOT a requirement** - we're open to better alternatives for standalone deployment.

**PRIMARY OBJECTIVE: Lightweightness** ⚠️
- **Minimize compute overhead** when developers use VWMP in their projects
- Avoid unnecessary dependencies and runtime overhead
- Keep resource usage minimal (CPU, memory, disk)
- Fast startup times and low latency

**Key Insight:** For a standalone open-source project, lightweight, modern, and developer-friendly technologies will maximize community adoption while minimizing compute overhead for end users.

---

## Current Technology Stack

### Current Implementation

**Backend API:**
- Django REST Framework (Django 5.2+)
- Django Channels (WebSocket support)
- Django authentication

**Frontend:**
- React 18 (via CDN)
- React Flow 11 (via CDN)
- Babel Standalone (JSX in browser)

**Core Engine:**
- Python 3.13+ (framework-agnostic) ✅
- No framework dependencies ✅

**Storage:**
- File-based (JSON/YAML) for workflow definitions
- No database required

**Real-time:**
- Django Channels (WebSocket)

---

## Evaluation Criteria

### ⚠️ **PRIMARY: Lightweightness & Compute Overhead**
- **Runtime overhead:** CPU, memory, disk usage when running
- **Startup time:** Time to start/initialize
- **Dependency size:** Total size of dependencies
- **Resource usage:** Memory footprint, CPU cycles
- **Idle overhead:** Resources used when idle
- **Integration overhead:** Impact on host application

### 1. Developer Experience
- **Setup complexity:** How easy is it to get started?
- **Development speed:** How fast can developers build features?
- **Documentation:** Quality and completeness of docs
- **Community support:** Availability of help and examples

### 2. Maintenance Burden
- **Code complexity:** Simplicity of codebase
- **Dependency management:** Number and size of dependencies
- **Upgrade path:** Ease of keeping dependencies updated
- **Security:** Frequency of security updates and patches

### 3. Community Adoption Potential
- **Technology popularity:** Current market trends
- **Learning curve:** Accessibility for new contributors
- **Ecosystem:** Availability of libraries and tools
- **Documentation quality:** Ease of onboarding

### 4. Deployment Simplicity
- **Standalone deployment:** Single executable or simple setup
- **Docker support:** Containerization ease
- **Cloud deployment:** AWS, GCP, Azure support
- **Self-hosted:** Simple local deployment

### 5. Framework Independence
- **Core isolation:** Core engine independent of API layer
- **API flexibility:** Can swap API framework without changing core
- **Plugin system:** Framework-agnostic plugin loading

---

## Backend API Framework Evaluation

### Option 1: FastAPI ⭐ **RECOMMENDED**

**Technology Stack:**
- FastAPI (modern, fast Python web framework)
- WebSocket support (built-in)
- Pydantic (type validation)
- Uvicorn (ASGI server)

**Pros:**
- ✅ **Modern and Popular:** Very popular in Python community (2020-2024)
- ✅ **Fast Performance:** Built on Starlette (async), very fast
- ✅ **Automatic API Docs:** OpenAPI/Swagger docs auto-generated
- ✅ **Type Safety:** Pydantic models for request/response validation
- ✅ **WebSocket Built-in:** Native WebSocket support (no separate library)
- ✅ **Async/Await:** Full async support for concurrent workflows
- ✅ **Minimal Boilerplate:** Less code than Django REST Framework
- ✅ **Standalone Ready:** Can run as single file or package
- ✅ **Developer Experience:** Excellent docs, great IDE support
- ✅ **Lightweight Dependencies:** ⚠️ **FastAPI + Uvicorn = ~5MB**, vs Django ~50MB
- ✅ **Low Runtime Overhead:** Minimal memory footprint (~20-50MB), fast startup (<1s)
- ✅ **Efficient Resource Usage:** Event-driven async, no blocking operations

**Cons:**
- ⚠️ **Newer Framework:** Less mature ecosystem than Django (but growing fast)
- ⚠️ **Less "Batteries Included":** Need to add auth, ORM separately (but not needed for VWMP)

**Example Structure:**
```python
# main.py
from fastapi import FastAPI, WebSocket
from vwmp.core import WorkflowEngine

app = FastAPI()

@app.post("/api/workflows/{workflow_id}/execute")
async def execute_workflow(workflow_id: str, parameters: dict):
    engine = WorkflowEngine()
    result = engine.execute_workflow(workflow_id, parameters)
    return result

@app.websocket("/ws/workflows/{workflow_id}")
async def workflow_status(websocket: WebSocket, workflow_id: str):
    await websocket.accept()
    # Stream execution updates
```

**Deployment:**
```bash
# Single command
uvicorn main:app --host 0.0.0.0 --port 8000

# With Docker
FROM python:3.13-slim
COPY . /app
WORKDIR /app
RUN pip install fastapi uvicorn vwmp
CMD ["uvicorn", "main:app", "--host", "0.0.0.0"]
```

**Verdict:** ⭐ **BEST CHOICE** - Modern, fast, popular, perfect for standalone OS project

---

### Option 2: Flask + Flask-SocketIO

**Technology Stack:**
- Flask (minimal Python web framework)
- Flask-SocketIO (WebSocket support)
- Flask-RESTful or Flask-RESTX (API support)

**Pros:**
- ✅ **Minimal and Simple:** Very lightweight, easy to understand
- ✅ **Mature Ecosystem:** Lots of extensions and community support
- ✅ **Flexible:** Can build exactly what you need
- ✅ **Well Documented:** Extensive documentation and examples

**Cons:**
- ❌ **More Boilerplate:** Need to manually set up API routes
- ❌ **Manual Validation:** Need to add request validation manually
- ❌ **No Auto Docs:** Need to generate API docs separately
- ⚠️ **WebSocket Setup:** Flask-SocketIO adds dependency complexity

**Verdict:** Good option, but FastAPI is better for modern projects

---

### Option 3: Django REST Framework (Current)

**Technology Stack:**
- Django REST Framework
- Django Channels (WebSocket)
- Django authentication

**Pros:**
- ✅ **Batteries Included:** Lots of built-in features
- ✅ **Mature Ecosystem:** Very stable and well-tested
- ✅ **Admin Interface:** Built-in admin UI (though not needed for VWMP)
- ✅ **ORM Available:** Database support if needed later

**Cons:**
- ❌ **Heavy:** Large framework with many dependencies
- ❌ **High Compute Overhead:** ⚠️ **Django + DRF + Channels = ~100-200MB memory**, slow startup (~2-5s)
- ❌ **Complex Setup:** More complex than needed for VWMP
- ❌ **Overkill:** VWMP doesn't need most Django features
- ❌ **Maintenance Burden:** Large dependency tree to maintain
- ❌ **Less Modern:** Not as modern/popular as FastAPI for new projects
- ❌ **Tight Coupling:** Django-specific patterns throughout
- ❌ **High Integration Overhead:** Heavy impact on host application performance

**Verdict:** ❌ **NOT RECOMMENDED** for standalone OS project - too heavy, unnecessary complexity

---

### Option 4: Starlette (FastAPI's Base)

**Technology Stack:**
- Starlette (async web framework)
- WebSocket support (built-in)

**Pros:**
- ✅ **Minimal:** Very lightweight
- ✅ **Async Native:** Full async support
- ✅ **Fast:** Very performant

**Cons:**
- ❌ **Too Low-Level:** Need to build API framework yourself
- ❌ **More Work:** FastAPI gives you more out of the box

**Verdict:** Too low-level, FastAPI is better

---

### Option 5: Quart (Async Flask)

**Technology Stack:**
- Quart (async Flask)
- Quart-CORS (CORS support)

**Pros:**
- ✅ **Async Flask:** Familiar Flask API with async support
- ✅ **WebSocket Support:** Built-in WebSocket support

**Cons:**
- ⚠️ **Less Popular:** Smaller community than FastAPI
- ⚠️ **Flask Compatibility:** Some Flask extensions don't work

**Verdict:** Good alternative, but FastAPI has more momentum

---

## Frontend Evaluation

### Option 1: React + React Flow (Current) ✅ **RECOMMENDED**

**Status:** ✅ Keep current choice

**Technology Stack:**
- React 18
- React Flow 11
- Modern build tools (Vite recommended for standalone)

**Pros:**
- ✅ **Mature Library:** React Flow is purpose-built for node editors
- ✅ **Large Ecosystem:** Huge React ecosystem
- ✅ **TypeScript Support:** Can migrate to TypeScript
- ✅ **Standalone SPA:** Can be deployed as standalone SPA
- ✅ **Proven:** Already working in current implementation

**Cons:**
- ⚠️ **Build Pipeline:** Need build tools for production (Vite recommended)

**Recommendation:** Keep React + React Flow, but migrate to Vite for standalone deployment

**Standalone Deployment:**
```bash
# Build standalone SPA
npm install
npm run build
# Deploy dist/ folder to CDN or static hosting
```

**Verdict:** ✅ **KEEP** - Proven, mature, perfect for visual editor

---

### Option 2: Vue 3 + Vue Flow

**Pros:**
- ✅ **Simpler Syntax:** Easier learning curve
- ✅ **Good Performance:** Vue 3 is very performant

**Cons:**
- ❌ **Vue Flow Less Mature:** Not as mature as React Flow
- ❌ **Smaller Ecosystem:** Less community support

**Verdict:** React Flow is better choice

---

## Real-Time Communication Evaluation

### Option 1: WebSocket (Current) ✅ **RECOMMENDED**

**Pros:**
- ✅ **Bidirectional:** Full bidirectional communication
- ✅ **Efficient:** Low overhead for frequent updates
- ✅ **Standard:** Well-supported standard

**Cons:**
- ⚠️ **Connection Management:** Need to handle reconnections

**Implementation:**
- FastAPI: Built-in WebSocket support
- Flask: Flask-SocketIO
- Django: Django Channels (heavy)

**Verdict:** ✅ **KEEP** - Best choice for real-time updates

---

### Option 2: Server-Sent Events (SSE)

**Pros:**
- ✅ **Simpler:** One-way streaming (server → client)
- ✅ **Automatic Reconnect:** Browser handles reconnection

**Cons:**
- ❌ **One-Way Only:** Can't send commands from client during execution
- ❌ **Less Efficient:** HTTP-based (more overhead)

**Verdict:** ❌ Not suitable - need bidirectional communication

---

### Option 3: HTTP Polling

**Pros:**
- ✅ **Simple:** Just HTTP GET requests

**Cons:**
- ❌ **Inefficient:** Constant polling overhead
- ❌ **Delayed Updates:** Not real-time

**Verdict:** ❌ Not suitable - too inefficient

---

## Database/Storage Evaluation

### Option 1: File-Based (Current) ✅ **RECOMMENDED**

**Pros:**
- ✅ **Simple:** No database setup required
- ✅ **Version Control:** Workflows can be versioned in Git
- ✅ **Portable:** Easy to backup and migrate
- ✅ **Fast Setup:** No database configuration

**Cons:**
- ⚠️ **Concurrency:** Need file locking for concurrent writes
- ⚠️ **Scalability:** May need database for multi-user scenarios

**Recommendation:** Keep file-based for initial release, add database option later

**Verdict:** ✅ **KEEP** - Perfect for standalone, single-user deployments

---

### Option 2: SQLite (Future Option)

**Pros:**
- ✅ **No Server:** Single file database
- ✅ **SQL Support:** Full SQL queries
- ✅ **Concurrency:** Better than file-based

**Cons:**
- ⚠️ **Setup Complexity:** Need database initialization
- ⚠️ **Less Portable:** Need database migrations

**Recommendation:** Add as optional storage backend for multi-user deployments

**Verdict:** Good future option for multi-user scenarios

---

## Recommended Tech Stack for Standalone OS Project

### ⭐ **RECOMMENDED STACK**

#### Backend API
- **Framework:** FastAPI ⭐
- **Server:** Uvicorn (ASGI)
- **WebSocket:** FastAPI built-in WebSocket
- **Validation:** Pydantic (built-in)
- **API Docs:** Auto-generated OpenAPI/Swagger

#### Core Engine
- **Language:** Python 3.13+ ✅ (keep current - framework-agnostic)
- **Plugin System:** importlib (keep current)
- **Validation:** JSON Schema (keep current)

#### Frontend
- **Framework:** React 18 ✅ (keep current)
- **Visual Editor:** React Flow 11 ✅ (keep current)
- **Build Tool:** Vite (migrate from CDN for standalone)
- **State Management:** Zustand or React Context (lightweight)

#### Storage
- **Workflows:** File-based (JSON/YAML) ✅ (keep current)
- **Future:** SQLite option for multi-user

#### Real-Time
- **Protocol:** WebSocket ✅ (keep current)
- **Implementation:** FastAPI built-in WebSocket

#### Authentication (Future)
- **Option 1:** FastAPI Security (OAuth2, JWT)
- **Option 2:** External auth (Keycloak, Auth0)
- **Option 3:** Simple API keys (for CLI usage)

---

## Migration Strategy

### Phase 1: Backend Migration (Django → FastAPI)

**Steps:**
1. Create FastAPI application structure
2. Port Django REST API endpoints to FastAPI
3. Port WebSocket functionality (Django Channels → FastAPI WebSocket)
4. Update frontend API client to use FastAPI endpoints
5. Test with existing React frontend
6. Deploy FastAPI backend
7. Remove Django dependency

**Estimated Effort:** 2-3 weeks

**Files to Create:**
```
vwmp/
  api/
    __init__.py
    main.py              # FastAPI app
    routes/
      workflows.py       # Workflow CRUD
      execution.py       # Execution endpoints
      websocket.py       # WebSocket handlers
    models.py            # Pydantic models
    dependencies.py      # Auth, workspace dependencies
  core/                  # Keep as-is (framework-agnostic)
  plugins/               # Keep as-is (framework-agnostic)
```

---

### Phase 2: Frontend Migration (CDN → Vite)

**Steps:**
1. Create Vite project structure
2. Migrate React components from CDN to npm modules
3. Set up build pipeline
4. Create standalone SPA bundle
5. Update API client for FastAPI endpoints
6. Test standalone deployment
7. Remove Django templates (replace with SPA)

**Estimated Effort:** 1-2 weeks

**Structure:**
```
vwmp-frontend/
  package.json
  vite.config.js
  src/
    main.jsx
    App.jsx
    components/
      Designer/
      Dashboard/
    api/
      client.js
```

---

### Phase 3: Standalone Package

**Steps:**
1. Create `vwmp` Python package structure
2. Move core engine to `vwmp/core/`
3. Move generic plugins to `vwmp/plugins/`
4. Create FastAPI API layer as `vwmp/api/`
5. Create standalone CLI entry point
6. Add setup.py/pyproject.toml
7. Create Docker image
8. Publish to PyPI

**Estimated Effort:** 1-2 weeks

**Package Structure:**
```
vwmp/
  __init__.py
  core/              # Framework-agnostic core
  plugins/           # Generic plugins (git, release)
  api/               # FastAPI API layer
  cli.py             # CLI entry point
  server.py          # Server entry point
  pyproject.toml
```

---

## Comparison: Django vs FastAPI for OS Project

| Criteria | Django REST Framework | FastAPI |
|----------|---------------------|---------|
| **Setup Complexity** | Complex (settings, apps, middleware) | Simple (single file possible) |
| **Code Size** | Large (many files) | Small (minimal boilerplate) |
| **Dependencies** | Heavy (Django + DRF + Channels) | Lightweight (FastAPI + Uvicorn) |
| **⚠️ Dependency Size** | **~50MB** (Django + DRF + Channels + Redis) | **~5MB** (FastAPI + Uvicorn) |
| **⚠️ Memory Footprint** | **~100-200MB** (idle) | **~20-50MB** (idle) |
| **⚠️ Startup Time** | **~2-5 seconds** | **<1 second** |
| **⚠️ CPU Overhead** | High (synchronous, blocking) | Low (async, event-driven) |
| **Performance** | Good | Excellent (async native) |
| **API Docs** | Manual (drf-spectacular) | Automatic (built-in OpenAPI) |
| **WebSocket** | Django Channels (complex) | Built-in (simple) |
| **Type Validation** | Manual (serializers) | Automatic (Pydantic) |
| **Modern** | Mature but older patterns | Modern, async-first |
| **Community** | Large but Django-focused | Growing, modern Python |
| **Maintenance** | Large dependency tree | Minimal dependencies |
| **Standalone** | Complex (Django project structure) | Simple (single executable) |
| **⚠️ Integration Overhead** | **HIGH** (heavy impact on host app) | **LOW** (minimal impact) |

**Verdict:** FastAPI wins on all criteria for standalone OS project

---

## Deployment Comparison

### Django Deployment
```bash
# Complex setup
django-admin startproject vwmp
# Configure settings.py, urls.py, wsgi.py, asgi.py
# Install Django, DRF, Channels, Redis
# Set up database (even if not using)
# Run migrations
# Configure static files
python manage.py collectstatic
python manage.py runserver
```

### FastAPI Deployment
```bash
# Simple setup
pip install fastapi uvicorn vwmp
# Single command
uvicorn vwmp.api.main:app --host 0.0.0.0 --port 8000

# Or as package
vwmp-server --host 0.0.0.0 --port 8000
```

**Verdict:** FastAPI is dramatically simpler for standalone deployment

---

## Recommended Action Plan

### Phase 1: Evaluate FastAPI (1 week)
1. Build prototype FastAPI backend
2. Port one endpoint (workflow list)
3. Test with existing React frontend
4. Evaluate developer experience

### Phase 2: Backend Migration (2-3 weeks)
1. Create FastAPI application structure
2. Port all API endpoints
3. Port WebSocket functionality
4. Update frontend API client
5. Test end-to-end
6. Deploy FastAPI backend

### Phase 3: Frontend Migration (1-2 weeks)
1. Set up Vite project
2. Migrate React components
3. Create standalone SPA
4. Test deployment

### Phase 4: Package Creation (1-2 weeks)
1. Create Python package structure
2. Add CLI entry point
3. Create Docker image
4. Publish to PyPI

**Total Estimated Effort:** 5-8 weeks

---

## Conclusion

### Recommended Tech Stack for Standalone OS Project

**Backend:** FastAPI ⭐
- Modern, fast, popular
- Automatic API docs
- Built-in WebSocket
- Minimal dependencies
- Perfect for standalone deployment

**Frontend:** React + React Flow ✅
- Proven, mature
- Excellent visual editor library
- Migrate to Vite for standalone deployment

**Core:** Python 3.13+ ✅
- Keep current - framework-agnostic

**Storage:** File-based ✅
- Keep current - simple, portable

**Real-Time:** WebSocket ✅
- Keep current - FastAPI has built-in support

### Why FastAPI Over Django? (Lightweightness Priority)

1. **⚠️ 10x Smaller Dependencies:** ~5MB vs ~50MB
2. **⚠️ 4x Less Memory:** ~20-50MB vs ~100-200MB
3. **⚠️ 5x Faster Startup:** <1s vs 2-5s
4. **⚠️ Lower CPU Overhead:** Async event-driven vs blocking synchronous
5. **⚠️ Minimal Integration Impact:** Low overhead when embedded in other projects
6. **Simpler:** Less code, easier to understand
7. **Faster:** Better performance, async native
8. **Modern:** More attractive to new contributors
9. **Standalone:** Perfect for single-executable deployment

### Migration Risk: **LOW**

- Core engine is framework-agnostic ✅
- Frontend can work with any REST API ✅
- Migration is mostly API layer refactoring
- No changes needed to core engine or plugins

---

## Related Documents

- **[VWMP Platform Architecture](E01-vwmp-platform-architecture.md)** - Current architecture
- **[VWMP Spin-Off Readiness Analysis](E01-vwmp-spin-off-readiness-analysis.md)** - Separation analysis
- **[VWMP Lightweightness Requirements](E01-vwmp-lightweightness-requirements.md)** - ⚠️ **Critical lightweightness requirements**
- **[FastAPI Documentation](https://fastapi.tiangolo.com/)** - FastAPI official docs

---

## Summary: Lightweightness is Primary Concern

**Key Finding:** FastAPI is **essential** for lightweightness.

**Django Stack:**
- ❌ ~100-200MB memory overhead
- ❌ ~2-5s startup time
- ❌ ~50MB dependencies
- ❌ High CPU overhead
- ❌ **Significant impact on host applications**

**FastAPI Stack:**
- ✅ ~20-50MB memory overhead ⚠️ **5x less**
- ✅ <1s startup time ⚠️ **5x faster**
- ✅ ~5MB dependencies ⚠️ **10x smaller**
- ✅ Low CPU overhead
- ✅ **Minimal impact on host applications** ⚠️ **CRITICAL**

**Recommendation:** Migrate to FastAPI to meet lightweightness requirements.

---

**Last Updated:** 2025-11-21
**Evaluation Version:** 1.0.0
