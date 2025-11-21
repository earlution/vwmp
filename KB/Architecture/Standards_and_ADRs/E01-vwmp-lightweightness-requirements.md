# VWMP Lightweightness Requirements

**Status:** Requirements - Critical
**Epic:** Epic 01 - Visual Workflow Management Platform (VWMP)
**Created:** 2025-11-21
**Version:** v0.21.0.41+9

---

## Executive Summary

**PRIMARY OBJECTIVE:** Minimize compute overhead when developers use VWMP in their own projects.

This document defines lightweightness requirements for VWMP, ensuring it has minimal impact on host applications and resource-constrained environments.

---

## Lightweightness Principles

### 1. Minimal Memory Footprint
- **Target:** <50MB idle memory, <100MB active memory
- **Current Django:** ~100-200MB idle, ~200-400MB active ❌
- **Target FastAPI:** ~20-50MB idle, ~50-100MB active ✅

### 2. Fast Startup Time
- **Target:** <1 second initialization
- **Current Django:** ~2-5 seconds ❌
- **Target FastAPI:** <1 second ✅

### 3. Small Dependency Size
- **Target:** <10MB total dependencies
- **Current Django:** ~50MB (Django + DRF + Channels + Redis) ❌
- **Target FastAPI:** ~5MB (FastAPI + Uvicorn) ✅

### 4. Low CPU Overhead
- **Target:** Minimal CPU usage when idle, efficient under load
- **Implementation:** Async/await for all I/O operations
- **Avoid:** Blocking synchronous operations

### 5. Minimal Integration Impact
- **Target:** Low overhead when embedded in other projects
- **Requirement:** Headless mode (no API server required)
- **Requirement:** Optional dependencies (API layer can be excluded)

---

## Resource Usage Targets

### Memory Footprint

| Component | Target | Current (Django) | Target (FastAPI) |
|-----------|--------|------------------|------------------|
| **Core Engine** | <20MB | ~30MB | ~15MB ✅ |
| **API Server (Idle)** | <30MB | ~100MB | ~25MB ✅ |
| **API Server (Active)** | <50MB | ~200MB | ~40MB ✅ |
| **WebSocket** | <10MB | ~30MB (Redis) | ~5MB (built-in) ✅ |
| **Total (Idle)** | **<50MB** | **~160MB** | **~45MB** ✅ |
| **Total (Active)** | **<100MB** | **~260MB** | **~60MB** ✅ |

### Startup Time

| Operation | Target | Current (Django) | Target (FastAPI) |
|-----------|--------|------------------|------------------|
| **Core Import** | <0.1s | ~0.2s | ~0.1s ✅ |
| **API Server Start** | <0.5s | ~2-5s | ~0.3s ✅ |
| **First Request** | <1.0s | ~3-6s | ~0.5s ✅ |
| **Total Initialization** | **<1.0s** | **~5-8s** | **<0.6s** ✅ |

### Dependency Size

| Package | Target | Current (Django) | Target (FastAPI) |
|---------|--------|------------------|------------------|
| **Core Engine** | <2MB | ~3MB | ~2MB ✅ |
| **API Framework** | <3MB | ~40MB (Django+DRF) | ~2MB (FastAPI) ✅ |
| **WebSocket** | <1MB | ~5MB (Channels+Redis) | Built-in ✅ |
| **Total** | **<10MB** | **~50MB** | **~5MB** ✅ |

---

## Implementation Requirements

### 1. Use FastAPI (Not Django)

**Requirement:** Backend API must use FastAPI, not Django.

**Rationale:**
- **10x smaller dependencies** (~5MB vs ~50MB)
- **4x less memory** (~20-50MB vs ~100-200MB)
- **5x faster startup** (<1s vs 2-5s)
- **Lower CPU overhead** (async vs blocking)

**Validation:**
- [ ] FastAPI used for API endpoints
- [ ] No Django dependencies in core or plugins
- [ ] Memory footprint <50MB idle
- [ ] Startup time <1 second

---

### 2. Headless Mode Support

**Requirement:** VWMP must support headless mode (no API server required).

**Rationale:**
- Allows embedding VWMP without API overhead
- Core engine can be used directly in scripts/CLI
- Minimizes resource usage when API not needed

**Implementation:**
```python
# Headless mode - no API server
from vwmp.core import WorkflowEngine

engine = WorkflowEngine()
result = engine.execute_workflow(workflow, parameters)

# With API server (optional)
from vwmp.api import create_app

app = create_app()  # Only if web UI needed
```

**Validation:**
- [ ] Core engine can be used without API server
- [ ] API server is optional dependency
- [ ] Headless mode documented

---

### 3. Lazy Loading

**Requirement:** Load plugins and components only when needed.

**Rationale:**
- Reduces initial memory footprint
- Faster startup time
- Minimizes unused imports

**Implementation:**
- Plugins loaded on-demand
- Step handlers imported only when workflow uses them
- API routes registered lazily

**Validation:**
- [ ] Plugins not loaded at import time
- [ ] Handlers imported only when needed
- [ ] Initial memory footprint minimal

---

### 4. Async Everywhere

**Requirement:** Use async/await for all I/O operations.

**Rationale:**
- Non-blocking operations
- Efficient resource usage
- Better performance under load

**Implementation:**
- All file I/O is async
- All network operations are async
- Step execution uses async where possible

**Validation:**
- [ ] No blocking I/O in core engine
- [ ] Async/await used for file operations
- [ ] Async/await used for network operations

---

### 5. Minimal Dependencies

**Requirement:** Only include essential dependencies, avoid heavy libraries.

**Core Dependencies (Required):**
- `pyyaml` - YAML parsing (~500KB)
- `jsonschema` - JSON Schema validation (~1MB)
- `pydantic` - Type validation (~2MB, if using FastAPI)

**API Dependencies (Optional):**
- `fastapi` - API framework (~2MB)
- `uvicorn` - ASGI server (~1MB)

**Avoid:**
- ❌ Django (~20MB)
- ❌ Django REST Framework (~5MB)
- ❌ Django Channels (~10MB)
- ❌ Redis client (~5MB, if not needed)
- ❌ Heavy ORM libraries (not needed - file-based storage)

**Validation:**
- [ ] Dependencies list reviewed
- [ ] No unnecessary dependencies
- [ ] Total dependency size <10MB

---

### 6. Optional API Layer

**Requirement:** API layer must be optional, core engine must work standalone.

**Rationale:**
- Some users may not need web UI
- CLI usage doesn't require API server
- Reduces overhead when API not needed

**Structure:**
```
vwmp/
  core/              # Framework-agnostic, no API dependencies
  plugins/           # Generic plugins, no API dependencies
  api/               # Optional FastAPI API layer
  cli.py             # CLI entry point (no API)
```

**Validation:**
- [ ] Core can be imported without API
- [ ] CLI works without API server
- [ ] API is separate optional package

---

## Testing Requirements

### Memory Profiling

**Requirement:** Measure and document memory usage.

**Tests:**
1. **Idle Memory:** Import VWMP, measure memory footprint
2. **Active Memory:** Execute workflow, measure peak memory
3. **API Memory:** Start API server, measure memory usage

**Targets:**
- Idle: <50MB
- Active: <100MB
- API Server: <30MB

---

### Startup Time Profiling

**Requirement:** Measure and document startup times.

**Tests:**
1. **Import Time:** Time to import vwmp.core
2. **Engine Init:** Time to create WorkflowEngine
3. **API Start:** Time to start API server
4. **First Request:** Time to handle first API request

**Targets:**
- Import: <0.1s
- Engine Init: <0.2s
- API Start: <0.5s
- First Request: <1.0s

---

### Dependency Size Measurement

**Requirement:** Measure and document dependency sizes.

**Tests:**
1. **Core Dependencies:** Size of core-only dependencies
2. **API Dependencies:** Size of API layer dependencies
3. **Total Size:** Combined dependency size

**Targets:**
- Core: <5MB
- API: <5MB
- Total: <10MB

---

## Validation Checklist

### Architecture
- [ ] FastAPI used (not Django)
- [ ] Headless mode supported
- [ ] API layer is optional
- [ ] Core engine is framework-agnostic

### Implementation
- [ ] Lazy loading implemented
- [ ] Async/await used for I/O
- [ ] Minimal dependencies
- [ ] No blocking operations

### Testing
- [ ] Memory profiling completed
- [ ] Startup time profiling completed
- [ ] Dependency size measured
- [ ] Targets met

### Documentation
- [ ] Memory usage documented
- [ ] Startup time documented
- [ ] Dependency size documented
- [ ] Headless mode documented

---

## Success Criteria

**VWMP is lightweight enough when:**

1. ✅ **Memory (Idle):** <50MB
2. ✅ **Memory (Active):** <100MB
3. ✅ **Startup Time:** <1 second
4. ✅ **Dependencies:** <10MB
5. ✅ **Integration Impact:** Minimal overhead on host application
6. ✅ **Headless Mode:** Works without API server

---

## Related Documents

- **[Tech Stack Evaluation](E01-vwmp-tech-stack-open-source-evaluation.md)** - Technology choices
- **[Spin-Off Readiness Analysis](E01-vwmp-spin-off-readiness-analysis.md)** - Architecture analysis

---

**Last Updated:** 2025-11-21
**Requirements Version:** 1.0.0
