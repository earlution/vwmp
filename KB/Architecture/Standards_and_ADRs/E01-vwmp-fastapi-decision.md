# VWMP FastAPI Technology Decision (ADR)

**Status:** Approved - Decision
**Epic:** Epic 01 - Visual Workflow Management Platform (VWMP)
**Created:** 2025-11-21
**Version:** v0.21.0.41+9
**Type:** Architecture Decision Record (ADR)

---

## Decision

**Selected:** FastAPI as the backend API framework for VWMP, replacing Django REST Framework.

**Rationale:** Lightweightness is a primary concern - we do not want to apply unnecessary compute overhead when other developers use VWMP in their own projects.

---

## Context

VWMP needs a backend API framework to provide REST endpoints and WebSocket support for the visual workflow designer UI. The original implementation used Django REST Framework, but for standalone open-source deployment, we need a lightweight solution that minimizes compute overhead.

**Primary Requirement:** Minimize compute overhead when developers embed VWMP in their projects.

---

## Decision Drivers

### ⚠️ **1. Lightweightness (PRIMARY REQUIREMENT)**

**Rationale:** We do not want to apply unnecessary compute overhead when other developers use VWMP in their own projects. Lightweightness is a primary concern beyond project objectives - minimizing resource usage is critical for adoption and integration.

**Django Stack:**
- Memory (Idle): ~100-200MB ❌
- Memory (Active): ~200-400MB ❌
- Startup Time: ~2-5 seconds ❌
- Dependencies: ~50MB ❌
- **High integration impact on host applications** ❌
- **Impact:** Significant overhead when embedded in other projects, making VWMP less attractive for adoption

**FastAPI Stack:**
- Memory (Idle): ~20-50MB ✅ (**5x less**)
- Memory (Active): ~50-100MB ✅ (**4x less**)
- Startup Time: <1 second ✅ (**5x faster**)
- Dependencies: ~5MB ✅ (**10x smaller**)
- **Low integration impact on host applications** ✅
- **Impact:** Minimal overhead when embedded, making VWMP attractive for adoption

### 2. Framework Independence

**Rationale:** Core engine must remain framework-agnostic to support standalone deployment and headless usage. API layer should be optional to minimize overhead when API server is not needed.

**Requirements:**
- Core engine must remain framework-agnostic
- API layer should be optional (headless mode)
- Easy to swap API framework if needed
- **FastAPI supports this** - lightweight API layer that can be excluded

### 3. Developer Experience

**Rationale:** Modern, popular technology with good developer experience will maximize community adoption and contribution potential for standalone open-source project.

**Requirements:**
- Modern, popular technology (attractive to contributors)
- Automatic API documentation (reduces maintenance burden)
- Good IDE support (improves development velocity)
- Active community (ensures long-term viability)
- **FastAPI meets all requirements** - modern, auto-docs, excellent IDE support

### 4. Maintenance Burden

**Rationale:** Minimal dependencies reduce security risk, update burden, and complexity. Simple codebase reduces maintenance effort and makes contribution easier.

**Requirements:**
- Minimal dependencies (reduce security risk)
- Easy to update (fewer dependencies to update)
- Low security risk (smaller attack surface)
- Simple codebase (easier to maintain and contribute)
- **FastAPI has minimal dependencies** - FastAPI + Uvicorn + Pydantic (~5MB total)

---

## Considered Options

### Option 1: FastAPI ⭐ **SELECTED**

**Technology Stack:**
- FastAPI (modern, fast Python web framework)
- Uvicorn (ASGI server)
- Built-in WebSocket support
- Pydantic (type validation)

**Pros:**
- ✅ **Minimal Compute Overhead** - 5x less memory, 5x faster startup
- ✅ **Small Dependencies** - ~5MB vs ~50MB
- ✅ **Built-in WebSocket** - No additional dependencies
- ✅ **Automatic API Docs** - OpenAPI/Swagger auto-generated
- ✅ **Async Native** - Full async/await support
- ✅ **Modern & Popular** - Growing Python community
- ✅ **Excellent Performance** - Built on Starlette (async)
- ✅ **Type Safety** - Pydantic models for validation
- ✅ **Headless Support** - API layer is optional

**Cons:**
- ⚠️ Newer framework (less mature than Django, but stable)
- ⚠️ Smaller ecosystem (but sufficient for VWMP needs)

**Verdict:** ✅ **BEST CHOICE** - Meets all requirements, especially lightweightness

---

### Option 2: Django REST Framework (Current) ❌

**Technology Stack:**
- Django REST Framework
- Django Channels (WebSocket)
- Redis (for Channels)

**Pros:**
- ✅ Mature, stable ecosystem
- ✅ Extensive documentation
- ✅ Large community

**Cons:**
- ❌ **High Compute Overhead** - ~100-200MB memory, ~2-5s startup
- ❌ **Large Dependencies** - ~50MB
- ❌ **Complex Setup** - Requires Django project structure
- ❌ **High Integration Impact** - Significant overhead on host applications
- ❌ **Overkill** - VWMP doesn't need most Django features
- ❌ **Redis Required** - Additional dependency for WebSocket

**Verdict:** ❌ **REJECTED** - Does not meet lightweightness requirements

---

### Option 3: Flask + Flask-SocketIO

**Pros:**
- ✅ Minimal framework
- ✅ Well documented

**Cons:**
- ⚠️ Manual API setup required
- ⚠️ No automatic API docs
- ⚠️ Flask-SocketIO adds complexity
- ⚠️ Less modern than FastAPI

**Verdict:** ⚠️ Not selected - FastAPI is better choice

---

## Consequences

### Positive

1. **Minimal Compute Overhead** ✅
   - ~20-50MB memory vs ~100-200MB (5x less)
   - <1s startup vs ~2-5s (5x faster)
   - Minimal impact on host applications

2. **Smaller Dependencies** ✅
   - ~5MB vs ~50MB (10x smaller)
   - Faster installation
   - Less disk space

3. **Better Performance** ✅
   - Async native
   - Event-driven
   - Efficient resource usage

4. **Simpler Deployment** ✅
   - Single command: `uvicorn vwmp.api.main:app`
   - No complex configuration
   - No database required (file-based storage)

5. **Headless Mode** ✅
   - API layer is optional
   - Core engine can be used without API server
   - Perfect for CLI and script embedding

6. **Automatic Documentation** ✅
   - OpenAPI/Swagger auto-generated
   - No manual documentation required

### Negative

1. **Migration Effort** ⚠️
   - Need to port Django REST API to FastAPI
   - Update frontend API client
   - Test thoroughly
   - **Estimated: 2-3 weeks**

2. **Learning Curve** ⚠️
   - Team needs to learn FastAPI (but it's simple)
   - Different patterns than Django

3. **Smaller Ecosystem** ⚠️
   - Less mature than Django
   - Fewer third-party packages (but sufficient)

### Neutral

1. **Code Changes** - API layer only (core engine unchanged)
2. **Frontend Changes** - Minimal (REST API compatible)
3. **Plugin Changes** - None (plugins are framework-agnostic)

---

## Implementation Strategy

### Phase 1: FastAPI Backend (2-3 weeks)

**Tasks:**
1. Create FastAPI application structure
2. Port Django REST API endpoints to FastAPI
3. Port WebSocket functionality (Django Channels → FastAPI WebSocket)
4. Update frontend API client
5. Test with existing React frontend
6. Deploy FastAPI backend

**Structure:**
```
vwmp/
  core/              # Framework-agnostic (unchanged)
  plugins/           # Framework-agnostic (unchanged)
  api/               # NEW: FastAPI API layer
    __init__.py
    main.py          # FastAPI app
    routes/
      workflows.py   # Workflow CRUD
      execution.py   # Execution endpoints
      websocket.py   # WebSocket handlers
    models.py        # Pydantic models
    dependencies.py  # Auth, workspace dependencies
```

### Phase 2: Headless Mode (1 week)

**Tasks:**
1. Ensure core engine works without API
2. Add CLI entry point
3. Document headless usage
4. Test headless mode

**Usage:**
```python
# Headless mode - no API server
from vwmp.core import WorkflowEngine

engine = WorkflowEngine()
result = engine.execute_workflow(workflow, parameters)
```

### Phase 3: Frontend Update (1 week)

**Tasks:**
1. Update API client for FastAPI endpoints
2. Test WebSocket connection
3. Verify all features work
4. Update documentation

---

## Validation Criteria

### Lightweightness Targets

- ✅ Memory (Idle): <50MB
- ✅ Memory (Active): <100MB
- ✅ Startup Time: <1 second
- ✅ Dependencies: <10MB
- ✅ Integration Impact: LOW

### Functionality

- ✅ All API endpoints working
- ✅ WebSocket real-time updates working
- ✅ Frontend integration working
- ✅ Headless mode working
- ✅ CLI usage working

### Performance

- ✅ API response time <100ms
- ✅ WebSocket latency <50ms
- ✅ Concurrent workflow execution working
- ✅ Memory usage within targets

---

## References

- **[Tech Stack Evaluation](E01-vwmp-tech-stack-open-source-evaluation.md)** - Full technology evaluation
- **[Lightweightness Requirements](E01-vwmp-lightweightness-requirements.md)** - Lightweightness targets
- **[FastAPI Documentation](https://fastapi.tiangolo.com/)** - FastAPI official docs
- **[Uvicorn Documentation](https://www.uvicorn.org/)** - Uvicorn ASGI server docs

---

## Status

**Decision Date:** 2025-11-21
**Status:** ✅ **APPROVED**
**Implementation:** Pending (2-3 weeks estimated)

---

**Last Updated:** 2025-11-21
**ADR Version:** 1.0.0
