# VWMP Technology Stack Decision Rationales Summary

**Status:** Summary - Reference
**Epic:** Epic 01 - Visual Workflow Management Platform (VWMP)
**Created:** 2025-11-21
**Version:** v0.21.0.41+9

---

## Purpose

This document provides a quick reference summary of all technology stack decision rationales for VWMP, ensuring that the reasoning behind each choice is clearly documented and easily accessible.

**PRIMARY OBJECTIVE:** Lightweightness - we do not want to apply unnecessary compute overhead when other developers use VWMP in their own projects.

---

## Technology Stack Overview

### Backend API: FastAPI ⚠️ **SELECTED**

**Decision:** FastAPI (replacing Django REST Framework)

**Rationale (PRIMARY - Lightweightness):**
- **5x less memory** (~20-50MB vs ~100-200MB)
- **5x faster startup** (<1s vs ~2-5s)
- **10x smaller dependencies** (~5MB vs ~50MB)
- **Minimal integration impact** on host applications

**Additional Rationale:**
- Built-in WebSocket support (no Redis required)
- Automatic API documentation (OpenAPI/Swagger)
- Modern, popular framework (growing Python community)
- Async native (efficient resource usage)
- Headless support (API layer is optional)

**Rejected:** Django REST Framework
- **Reason:** High compute overhead (~100-200MB memory, ~2-5s startup)
- **Reason:** Requires Redis for WebSocket (additional dependency)
- **Reason:** Overkill for VWMP needs
- **Reason:** High integration impact on host applications

**Reference:** [FastAPI Decision ADR](E01-vwmp-fastapi-decision.md)

---

### Frontend: React + React Flow ⚠️ **SELECTED**

**Decision:** React 18 + React Flow 11

**Rationale:**
1. **React Flow is Purpose-Built**
   - Specifically designed for node-based visual editors
   - Includes drag-and-drop, zoom, pan, mini-map, connections out of the box
   - **This is exactly what we need** for the visual workflow designer

2. **Maturity and Proven Track Record**
   - Used in production by many companies
   - Large, active community
   - Comprehensive documentation
   - Regular updates and bug fixes
   - **Lower risk** compared to alternatives

3. **Rapid Development**
   - CDN approach enables immediate start (no build setup)
   - React Flow provides most features we need out of the box
   - **Significantly faster** than building from scratch (weeks vs months)

4. **Standalone Deployment Support**
   - Can be deployed as standalone SPA
   - Works with any REST API backend (not tied to Django)
   - Easy to migrate from CDN to npm/Vite for production
   - **Aligns with spin-off objective**

5. **Future Flexibility**
   - Can migrate to npm/Vite for production optimization
   - Can add TypeScript later
   - Can integrate other React libraries as needed

**Rejected:** Vue.js + Vue Flow
- **Reason:** Vue Flow is newer and less mature
- **Reason:** Smaller ecosystem and fewer examples
- **Reason:** Higher risk for critical visual editor feature

**Rejected:** Vanilla JavaScript + Custom Canvas
- **Reason:** Much higher development effort (200+ hours vs 50-100 hours)
- **Reason:** Need to implement drag-and-drop, zoom, pan, connections from scratch
- **Reason:** More maintenance burden

**Reference:** [Frontend Technology Decision ADR](E01-vwmp-frontend-technology-decision.md)

---

### Core Engine: Python 3.13+ (Framework-Agnostic) ⚠️ **SELECTED**

**Decision:** Python 3.13+ with framework-agnostic design

**Rationale:**
1. **Framework-Agnostic Design**
   - No dependencies on Django, FastAPI, or any web framework
   - Can be used standalone (headless mode)
   - **Critical for spin-off** - easy to extract independently

2. **Plugin System (importlib)**
   - Uses Python standard library (no dependencies)
   - Dynamic plugin loading
   - **Lightweight** - no heavy plugin frameworks

3. **Validation (JSON Schema)**
   - Standard validation (jsonschema library)
   - Lightweight validation library
   - **No heavy validation frameworks**

4. **Language Choice**
   - Python 3.13+ provides latest features (type hints, async/await)
   - Popular for workflow/orchestration tools
   - **Good fit** for VWMP requirements

**Reference:** [Platform Architecture](E01-vwmp-platform-architecture.md)

---

### Storage: File-Based (JSON/YAML) ⚠️ **SELECTED**

**Decision:** File-based storage for workflow definitions

**Rationale:**
1. **Simplicity**
   - No database setup required - works immediately
   - No migrations or schema management
   - **Fastest setup** for developers

2. **Version Control**
   - Workflow definitions can be tracked in Git
   - Easy to see changes and history
   - **Developer-friendly** workflow management

3. **Portability**
   - Easy to backup, restore, and migrate
   - Can copy workflows between projects
   - **Perfect for standalone deployment**

4. **Lightweight**
   - File-based storage has minimal overhead
   - No database process running
   - **Minimal resource usage**

5. **Sufficient**
   - Most workflows are small enough for file storage
   - JSON/YAML is human-readable
   - **Works for majority of use cases**

**Future Option:** SQLite can be added for multi-user scenarios if needed.

**Reference:** [Platform Architecture](E01-vwmp-platform-architecture.md)

---

### Real-Time: WebSocket ⚠️ **SELECTED**

**Decision:** WebSocket for real-time execution updates

**Rationale:**
1. **Bidirectional Communication**
   - Full bidirectional communication (server ↔ client)
   - Allows real-time updates during execution
   - **Efficient** for frequent status updates

2. **FastAPI Built-in**
   - FastAPI has built-in WebSocket support
   - No additional dependencies (unlike Django Channels which needs Redis)
   - **Lightweight** - no Redis required

3. **Efficient**
   - Low overhead for frequent updates
   - Better than polling (no constant HTTP requests)
   - **Minimal resource usage**

**Rejected:** Server-Sent Events (SSE)
- **Reason:** One-way only (server → client)
- **Reason:** Can't send commands from client during execution

**Rejected:** HTTP Polling
- **Reason:** Inefficient (constant polling overhead)
- **Reason:** Delayed updates (not real-time)

**Reference:** [Platform Architecture](E01-vwmp-platform-architecture.md)

---

### Plugin Architecture: Python Plugin System ⚠️ **SELECTED**

**Decision:** Python plugin architecture using importlib

**Rationale:**
1. **Separation of Concerns**
   - Isolates Project-specific-specific code from generic core engine
   - **Critical for spin-off** - generic plugins can be open-sourced

2. **Spin-Off Ready**
   - Generic plugins (Git, Release, Filesystem) can be open-sourced
   - Project-specific plugins (KB, Kanban, Validators) remain proprietary
   - **Clear separation** enables standalone project

3. **Extensibility**
   - Community can add custom workflow types and step handlers
   - Easy to create new plugins
   - **Fosters ecosystem growth**

4. **Maintainability**
   - Clear plugin boundaries make codebase easier to understand
   - Plugin isolation simplifies maintenance
   - **Reduces complexity**

5. **Lightweight**
   - Uses Python standard library (importlib)
   - No heavy plugin frameworks
   - **Minimal overhead**

**Reference:** [Plugin Architecture](E01-vwmp-plugin-architecture.md)

---

## Decision Summary Table

| Technology | Decision | Primary Rationale | Performance Impact |
|-----------|----------|-------------------|-------------------|
| **Backend API** | FastAPI | ⚠️ **Lightweightness** (5x less memory, 5x faster startup) | ✅ Minimal (~20-50MB) |
| **Frontend** | React + React Flow | Purpose-built for visual editors, rapid development | ✅ Efficient |
| **Core Engine** | Python 3.13+ (Framework-Agnostic) | Framework independence, spin-off ready | ✅ Minimal (~15MB) |
| **Storage** | File-based (JSON/YAML) | Simplicity, version control, lightweight | ✅ Minimal overhead |
| **Real-Time** | WebSocket (FastAPI built-in) | Bidirectional, efficient, no Redis required | ✅ Low overhead |
| **Plugins** | Python importlib | Lightweight, separation of concerns | ✅ Standard library |

---

## Key Rationale Themes

### 1. ⚠️ **Lightweightness (PRIMARY)**
- Minimize compute overhead when developers embed VWMP
- Fast startup times
- Low memory footprint
- Small dependencies

**Technologies Selected:**
- FastAPI (not Django)
- File-based storage (not database)
- FastAPI built-in WebSocket (not Redis)
- Python standard library plugins (not heavy frameworks)

### 2. **Framework Independence**
- Core engine independent of web frameworks
- API layer is optional (headless mode)
- Frontend works with any REST API
- Easy to swap components

**Technologies Selected:**
- Framework-agnostic core engine
- FastAPI (can be replaced)
- React SPA (standalone deployment)

### 3. **Spin-Off Ready**
- Generic components can be extracted
- Project-specific-specific code isolated
- Clear separation of concerns
- Easy to package independently

**Technologies Selected:**
- Plugin architecture with clear boundaries
- Framework-agnostic core
- Standalone SPA frontend
- File-based storage (portable)

### 4. **Developer Experience**
- Modern, popular technologies
- Good documentation
- Active communities
- Easy to learn and use

**Technologies Selected:**
- FastAPI (modern, popular)
- React + React Flow (mature, proven)
- Python 3.13+ (widely used)

### 5. **Rapid Development**
- Purpose-built libraries (React Flow)
- CDN approach (no build pipeline initially)
- Automatic documentation (FastAPI)
- Minimal boilerplate

**Technologies Selected:**
- React Flow (purpose-built)
- FastAPI (automatic docs)
- CDN approach (rapid prototyping)

---

## Rejected Alternatives Summary

| Technology | Rejected Alternative | Reason |
|-----------|---------------------|---------|
| **Backend** | Django REST Framework | ❌ High compute overhead (~100-200MB, ~2-5s startup) |
| **Backend** | Flask + Flask-SocketIO | ⚠️ Manual API setup, no auto docs, less modern |
| **Frontend** | Vue.js + Vue Flow | ❌ Less mature, smaller ecosystem |
| **Frontend** | Vanilla JavaScript | ❌ 200+ hours vs 50-100 hours, much higher effort |
| **Real-Time** | Server-Sent Events | ❌ One-way only, can't send commands |
| **Real-Time** | HTTP Polling | ❌ Inefficient, delayed updates |
| **WebSocket** | Django Channels + Redis | ❌ Additional dependency, higher overhead |

---

## Complete Rationale Documentation

### Detailed ADRs

1. **[FastAPI Decision ADR](E01-vwmp-fastapi-decision.md)** ⚠️ **CRITICAL**
   - Complete rationale for FastAPI over Django
   - Performance comparison
   - Consequences and implementation strategy

2. **[Frontend Technology Decision ADR](E01-vwmp-frontend-technology-decision.md)**
   - Complete rationale for React + React Flow
   - Alternatives evaluated
   - Implementation strategy

### Architecture Documents

3. **[Platform Architecture](E01-vwmp-platform-architecture.md)**
   - Overall architecture rationale
   - Technology stack decisions with rationales
   - Component design rationales

4. **[Plugin Architecture](E01-vwmp-plugin-architecture.md)**
   - Plugin system rationale
   - Design decisions with reasoning
   - Open-source packaging strategy

### Requirements

5. **[Lightweightness Requirements](E01-vwmp-lightweightness-requirements.md)** ⚠️ **CRITICAL**
   - Primary objective: Minimize compute overhead
   - Resource usage targets
   - Implementation requirements

6. **[Tech Stack Evaluation](E01-vwmp-tech-stack-open-source-evaluation.md)**
   - Comprehensive technology evaluation
   - Framework comparison with detailed rationale
   - Recommended stack analysis

---

## Rationale Verification Checklist

- ✅ **Backend API:** FastAPI rationale documented (lightweightness - PRIMARY)
- ✅ **Frontend:** React + React Flow rationale documented (purpose-built, maturity)
- ✅ **Core Engine:** Framework-agnostic rationale documented (independence, spin-off)
- ✅ **Storage:** File-based rationale documented (simplicity, version control)
- ✅ **Real-Time:** WebSocket rationale documented (bidirectional, built-in)
- ✅ **Plugins:** Plugin architecture rationale documented (separation, spin-off)
- ✅ **All alternatives:** Rejected options documented with reasons
- ✅ **Performance:** Resource usage targets documented
- ✅ **Lightweightness:** PRIMARY concern clearly documented

---

## Quick Reference

**Primary Rationale:** ⚠️ **Lightweightness** - Minimize compute overhead when developers use VWMP in their projects.

**Key Decisions:**
1. FastAPI (not Django) - 5x less memory, 5x faster startup
2. React + React Flow (not Vue or Vanilla) - Purpose-built, proven
3. File-based storage (not database) - Simplicity, version control
4. Framework-agnostic core - Independence, spin-off ready
5. Python standard library plugins - Lightweight, no dependencies

**All Rationales Documented:** ✅ Yes

---

## Related Documents

- **[FastAPI Decision ADR](E01-vwmp-fastapi-decision.md)** - Detailed FastAPI rationale
- **[Frontend Technology Decision ADR](E01-vwmp-frontend-technology-decision.md)** - Detailed React rationale
- **[Platform Architecture](E01-vwmp-platform-architecture.md)** - Overall architecture rationales
- **[Plugin Architecture](E01-vwmp-plugin-architecture.md)** - Plugin system rationales
- **[Lightweightness Requirements](E01-vwmp-lightweightness-requirements.md)** - Performance targets
- **[Tech Stack Evaluation](E01-vwmp-tech-stack-open-source-evaluation.md)** - Full evaluation

---

**Last Updated:** 2025-11-21
**Summary Version:** 1.0.0
