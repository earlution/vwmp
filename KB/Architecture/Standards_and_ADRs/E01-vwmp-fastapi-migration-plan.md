# VWMP FastAPI Migration Plan

**Status:** Plan - Implementation Guide
**Epic:** Epic 01 - Visual Workflow Management Platform (VWMP)
**Created:** 2025-11-21
**Version:** v0.21.0.41+9
**Related ADR:** [FastAPI Technology Decision](E01-vwmp-fastapi-decision.md)

---

## Executive Summary

This document provides a detailed migration plan for moving VWMP from Django REST Framework to FastAPI, ensuring minimal compute overhead and maintaining all existing functionality.

**Migration Goal:** Replace Django API layer with FastAPI while maintaining 100% feature compatibility and reducing compute overhead by 5x.

**Estimated Effort:** 2-3 weeks
**Risk Level:** Low (core engine unchanged, API layer only)

---

## Migration Strategy

### Overview

**Principle:** Core engine is framework-agnostic, so migration is API layer only.

**Approach:**
1. Create FastAPI backend alongside Django (parallel development)
2. Port endpoints one by one
3. Test each endpoint independently
4. Update frontend API client incrementally
5. Switch over when complete
6. Remove Django API layer

---

## Current Architecture

### Django API Layer (Current)

```
src/vwmp_django/vwmp/
  views.py              # Django REST API endpoints
  urls.py               # URL routing
  apps.py               # Django app config
```

**Endpoints:**
- `GET/POST /api/vwmp/workflows/` - List/Create workflows
- `GET/PUT/DELETE /api/vwmp/workflows/<id>/` - Workflow CRUD
- `POST /api/vwmp/workflows/<id>/execute/` - Execute workflow
- `GET /api/vwmp/step-handlers/` - List step handlers
- `GET /api/vwmp/step-handlers/<path>/schema/` - Handler schema
- `GET /api/vwmp/workflow-types/` - List workflow types
- WebSocket: Django Channels (not yet implemented)

**Templates:**
- `templates/vwmp/dashboard.html` - Dashboard UI
- `templates/vwmp/designer.html` - Visual designer UI

---

## Target Architecture

### FastAPI API Layer (Target)

```
vwmp/
  core/                 # Framework-agnostic (unchanged)
  plugins/              # Framework-agnostic (unchanged)
  api/                  # NEW: FastAPI API layer
    __init__.py
    main.py             # FastAPI app
    routes/
      workflows.py      # Workflow CRUD routes
      execution.py      # Execution routes
      handlers.py       # Handler routes
      websocket.py      # WebSocket routes
    models.py           # Pydantic request/response models
    dependencies.py     # Auth, workspace dependencies
    config.py           # API configuration
  cli.py                # CLI entry point (headless mode)
  server.py             # Server entry point
```

**Endpoints (Same API, FastAPI implementation):**
- `GET/POST /api/vwmp/workflows/` - List/Create workflows
- `GET/PUT/DELETE /api/vwmp/workflows/{id}/` - Workflow CRUD
- `POST /api/vwmp/workflows/{id}/execute/` - Execute workflow
- `GET /api/vwmp/step-handlers/` - List step handlers
- `GET /api/vwmp/step-handlers/{path}/schema/` - Handler schema
- `GET /api/vwmp/workflow-types/` - List workflow types
- WebSocket: FastAPI built-in WebSocket

**Frontend:**
- Standalone SPA (Vite build) - serve from FastAPI static files
- Or continue serving via Django templates (during transition)

---

## Migration Phases

### Phase 1: FastAPI Foundation (Week 1)

**Goal:** Set up FastAPI application structure and port basic endpoints.

**Tasks:**
1. **Create FastAPI Structure**
   - Create `vwmp/api/` directory
   - Create `main.py` with FastAPI app
   - Set up routing structure

2. **Port Basic Endpoints**
   - `GET /api/vwmp/workflows/` - List workflows
   - `GET /api/vwmp/workflows/{id}/` - Get workflow
   - Test with existing frontend

3. **Create Pydantic Models**
   - Workflow request/response models
   - Parameter models
   - Error models

4. **Test Basic Functionality**
   - Verify workflows list works
   - Verify workflow retrieval works
   - Compare with Django implementation

**Deliverables:**
- FastAPI app structure
- Basic workflow endpoints
- Pydantic models
- Tests passing

---

### Phase 2: Full CRUD & Execution (Week 2)

**Goal:** Port remaining CRUD endpoints and execution endpoint.

**Tasks:**
1. **Port Workflow CRUD**
   - `POST /api/vwmp/workflows/` - Create workflow
   - `PUT /api/vwmp/workflows/{id}/` - Update workflow
   - `DELETE /api/vwmp/workflows/{id}/` - Delete workflow

2. **Port Execution Endpoint**
   - `POST /api/vwmp/workflows/{id}/execute/` - Execute workflow
   - Test workflow execution
   - Verify result format matches Django

3. **Port Handler Endpoints**
   - `GET /api/vwmp/step-handlers/` - List handlers
   - `GET /api/vwmp/step-handlers/{path}/schema/` - Handler schema

4. **Port Workflow Types Endpoint**
   - `GET /api/vwmp/workflow-types/` - List workflow types

5. **Update Frontend API Client**
   - Update fetch calls to work with FastAPI
   - Test all endpoints
   - Fix any incompatibilities

**Deliverables:**
- All CRUD endpoints working
- Execution endpoint working
- Handler endpoints working
- Frontend working with FastAPI

---

### Phase 3: WebSocket & Frontend (Week 3)

**Goal:** Port WebSocket functionality and finalize frontend integration.

**Tasks:**
1. **Port WebSocket Support**
   - Implement FastAPI WebSocket endpoint
   - Port real-time execution updates
   - Test WebSocket connection

2. **Frontend WebSocket Client**
   - Update frontend WebSocket client
   - Test real-time updates
   - Verify execution monitoring

3. **Frontend Deployment**
   - Option A: Serve SPA from FastAPI static files
   - Option B: Keep Django templates during transition
   - Test full integration

4. **Performance Testing**
   - Measure memory usage
   - Measure startup time
   - Verify lightweightness targets met

5. **End-to-End Testing**
   - Test all workflows
   - Test visual designer
   - Test execution monitoring
   - Test headless mode

**Deliverables:**
- WebSocket working
- Frontend fully integrated
- Performance targets met
- All tests passing

---

### Phase 4: Cleanup & Documentation (Week 3-4)

**Goal:** Remove Django API layer and update documentation.

**Tasks:**
1. **Remove Django API Code**
   - Remove `src/vwmp_django/vwmp/views.py` API endpoints
   - Keep Django templates if serving from Django
   - Update Django URLs (remove API routes)

2. **Update Documentation**
   - Update architecture docs
   - Update API documentation
   - Update deployment guides
   - Update developer guides

3. **Create Migration Guide**
   - Document migration process
   - Document breaking changes (if any)
   - Provide examples

4. **Final Testing**
   - Full regression testing
   - Performance validation
   - Documentation review

**Deliverables:**
- Django API layer removed
- Documentation updated
- Migration guide created
- Ready for production

---

## Implementation Details

### FastAPI Application Structure

```python
# vwmp/api/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from vwmp.api.routes import workflows, execution, handlers, websocket

app = FastAPI(
    title="VWMP API",
    description="Visual Workflow Management Platform API",
    version="0.21.0.0",
)

# CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Configure for production
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Routes
app.include_router(workflows.router, prefix="/api/vwmp", tags=["workflows"])
app.include_router(execution.router, prefix="/api/vwmp", tags=["execution"])
app.include_router(handlers.router, prefix="/api/vwmp", tags=["handlers"])
app.include_router(websocket.router, prefix="/ws/vwmp", tags=["websocket"])
```

### Workflow Endpoint Example

```python
# vwmp/api/routes/workflows.py
from fastapi import APIRouter, HTTPException, Depends
from vwmp.core import WorkflowStorage, WorkflowValidator
from vwmp.api.models import WorkflowCreate, WorkflowResponse
from vwmp.api.dependencies import get_workspace_root

router = APIRouter()

@router.get("/workflows/", response_model=list[WorkflowResponse])
async def list_workflows(workspace_root: Path = Depends(get_workspace_root)):
    """List all workflows."""
    storage = WorkflowStorage(storage_root=workspace_root / "workflows")
    workflows = storage.list_workflows()
    return [WorkflowResponse.from_metadata(meta) for meta in workflows]

@router.post("/workflows/", response_model=WorkflowResponse, status_code=201)
async def create_workflow(
    workflow_data: WorkflowCreate,
    workspace_root: Path = Depends(get_workspace_root),
):
    """Create a new workflow."""
    # Validate workflow
    validator = WorkflowValidator()
    validation_result = validator.validate(workflow_data.to_workflow_definition())
    if not validation_result.valid:
        raise HTTPException(status_code=400, detail=validation_result.errors)

    # Save workflow
    storage = WorkflowStorage(storage_root=workspace_root / "workflows")
    workflow_id = storage.save(workflow_data.to_workflow_definition())
    meta = storage.get_metadata(workflow_id)
    return WorkflowResponse.from_metadata(meta)
```

### WebSocket Example

```python
# vwmp/api/routes/websocket.py
from fastapi import APIRouter, WebSocket, WebSocketDisconnect
from vwmp.core import WorkflowEngine
from vwmp.core.monitoring import ExecutionObserver

router = APIRouter()

@router.websocket("/workflows/{workflow_id}/execute")
async def workflow_execution_updates(websocket: WebSocket, workflow_id: str):
    """WebSocket endpoint for real-time workflow execution updates."""
    await websocket.accept()

    try:
        # Create observer to send updates via WebSocket
        class WebSocketObserver(ExecutionObserver):
            async def on_step_completed(self, step_id: str, result: StepResult):
                await websocket.send_json({
                    "event": "step_completed",
                    "step_id": step_id,
                    "status": result.status.value,
                })

        # Execute workflow with observer
        engine = WorkflowEngine()
        observer = WebSocketObserver()
        engine.execution_subject.attach_observer(observer)

        result = await engine.execute_workflow_async(workflow_id, {})

        await websocket.send_json({
            "event": "workflow_completed",
            "status": result.status.value,
        })
    except WebSocketDisconnect:
        pass
```

---

## Testing Strategy

### Unit Tests

**Test FastAPI endpoints independently:**
- Test each route handler
- Test Pydantic models
- Test error handling
- Test validation

### Integration Tests

**Test with core engine:**
- Test workflow CRUD operations
- Test workflow execution
- Test WebSocket updates
- Test with real workflows

### Performance Tests

**Verify lightweightness targets:**
- Memory usage profiling
- Startup time measurement
- Request latency testing
- Concurrent request handling

### End-to-End Tests

**Test full system:**
- Test with React frontend
- Test visual designer
- Test execution monitoring
- Test headless mode

---

## Rollback Plan

### If Issues Arise

1. **Keep Django API Running** - Parallel deployment
2. **Switch Frontend Back** - Update API client URL
3. **Investigate Issues** - Fix FastAPI implementation
4. **Re-test** - Verify fixes
5. **Switch Back** - When stable

### Migration State Management

- Use feature flags to switch between Django/FastAPI
- Monitor performance metrics
- Have rollback procedure documented
- Test rollback procedure

---

## Success Criteria

### Functionality

- ✅ All endpoints working
- ✅ WebSocket working
- ✅ Frontend fully functional
- ✅ Headless mode working
- ✅ CLI working

### Performance

- ✅ Memory (Idle): <50MB
- ✅ Memory (Active): <100MB
- ✅ Startup Time: <1 second
- ✅ Dependencies: <10MB
- ✅ API Latency: <100ms

### Quality

- ✅ All tests passing
- ✅ Documentation updated
- ✅ Migration guide complete
- ✅ No regressions

---

## Timeline

| Phase | Duration | Start | End |
|-------|----------|-------|-----|
| **Phase 1: Foundation** | 1 week | Week 1 | Week 1 |
| **Phase 2: CRUD & Execution** | 1 week | Week 2 | Week 2 |
| **Phase 3: WebSocket & Frontend** | 1 week | Week 3 | Week 3 |
| **Phase 4: Cleanup** | 1 week | Week 4 | Week 4 |
| **Total** | **3-4 weeks** | - | - |

---

## Dependencies

### External

- FastAPI (latest stable)
- Uvicorn (latest stable)
- Pydantic (latest stable)

### Internal

- Core engine (unchanged)
- Plugins (unchanged)
- Frontend (needs API client update)

---

## Risks & Mitigations

### Risk 1: WebSocket Compatibility

**Risk:** Frontend WebSocket client may not work with FastAPI WebSocket.

**Mitigation:**
- Test WebSocket early (Phase 3)
- Use standard WebSocket API (compatible)
- Have fallback (polling) if needed

### Risk 2: Performance Not Met

**Risk:** FastAPI may not meet lightweightness targets.

**Mitigation:**
- Profile early and often
- Measure at each phase
- Optimize as needed
- Have Django as fallback

### Risk 3: Breaking Changes

**Risk:** API changes may break frontend.

**Mitigation:**
- Maintain API compatibility
- Test frontend early
- Version API if needed

---

## Related Documents

- **[FastAPI Technology Decision](E01-vwmp-fastapi-decision.md)** - ADR for FastAPI choice
- **[Tech Stack Evaluation](E01-vwmp-tech-stack-open-source-evaluation.md)** - Full evaluation
- **[Lightweightness Requirements](E01-vwmp-lightweightness-requirements.md)** - Performance targets

---

## Status

**Plan Status:** Ready for implementation
**Next Step:** Phase 1 - FastAPI Foundation

---

**Last Updated:** 2025-11-21
**Plan Version:** 1.0.0
