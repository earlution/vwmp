# Epic 01 – VWMP Container Architecture (C4 Level 2)

**Status:** Draft
**Epic:** Epic 01 - Visual Workflow Management Platform (VWMP)
**Last Updated:** 2025-11-20
**Related Artifacts:**
- `KB/Architecture/context/overview/epic01-vwmp-context.md`
- `KB/Architecture/component/overview/epic01-vwmp-components.md`
- `KB/Architecture/Standards_and_ADRs/E01-vwmp-platform-architecture.md`
- `KB/Architecture/Diagrams/Container/epic01-vwmp-container.puml`

---

## 1. Context & Goals

The VWMP container architecture decomposes the platform into major services/modules that can be deployed, scaled, and maintained independently. This view focuses on the high-level containers (services) and their responsibilities, technology choices, and interactions.

**Architecture Principles:**
- **Separation of Concerns:** Clear boundaries between UI, API, execution engine, and plugins
- **Plugin-Based:** Extensible architecture with plugin system
- **Framework-Agnostic Core:** Core engine independent of framework-specific code
- **Open-Source Ready:** Architecture designed for future standalone deployment

---

## 2. Core Containers

![C4 Container Diagram](../../Diagrams/Container/epic01-vwmp-container.puml)

| Container | Description | Technology | Key Responsibilities |
| --- | --- | --- | --- |
| **Visual Workflow Designer** | Frontend UI for workflow creation and execution | React/Vue.js + React Flow/Vue Flow | Drag-and-drop workflow builder, step configuration UI, parameter input forms, execution monitoring dashboard |
| **Workflow Management API** | REST API and WebSocket server for workflow operations | FastAPI + FastAPI WebSocket | Workflow CRUD operations, execution control (start/stop/pause), status monitoring endpoints, configuration management, real-time updates via WebSocket |
| **Workflow Execution Engine** | Core workflow orchestration and step execution | Python 3.13+ (framework-agnostic) | Workflow parsing and validation, step execution orchestration, dependency resolution, error handling and recovery, plugin system management |
| **Workflow Storage** | Persistent storage for workflow definitions and execution history | JSON/YAML files + PostgreSQL (optional) | Workflow definition persistence, version history, execution history, configuration storage |
| **Release Workflow Plugin** | Plugin implementing release workflow use case | Python plugin (depends on execution engine) | Release workflow type definition, step handlers (version bump, changelog, git ops), Project-specific integrations (KB, Kanban) |
| **Git Integration Plugin** | Plugin for git operations | Python plugin (depends on execution engine) | Git command execution (stage, commit, tag, push), git status checking, branch management |
| **Project Integration Plugin** | Plugin for Project-specific integrations | Python plugin (depends on execution engine) | KB updater integration, Kanban updater integration, validator script execution |
| **Validator Service** | External validation scripts | Python scripts (existing) | Branch context validation, changelog format validation, custom validation rules |

---

## 3. Container Interactions (High Level)

### Workflow Creation Flow
1. **User** creates workflow in Visual Workflow Designer
2. **Visual Workflow Designer** sends workflow definition to Workflow Management API
3. **Workflow Management API** validates definition via Execution Engine
4. **Workflow Management API** stores definition in Workflow Storage
5. **Workflow Storage** persists workflow definition (YAML/JSON file)

### Workflow Execution Flow (Release Workflow)
1. **User** selects workflow and provides parameters via Visual Workflow Designer
2. **Visual Workflow Designer** sends execution request to Workflow Management API
3. **Workflow Management API** validates parameters and triggers Execution Engine
4. **Execution Engine** loads workflow definition from Workflow Storage
5. **Execution Engine** resolves execution order and executes steps:
   - Calls Release Workflow Plugin for release-specific steps
   - Calls Git Integration Plugin for git operations
   - Calls Project Integration Plugin for KB/Kanban updates
   - Calls Validator Service for validation
6. **Execution Engine** sends status updates to Workflow Management API
7. **Workflow Management API** streams status to Visual Workflow Designer via WebSocket
8. **Execution Engine** stores execution history in Workflow Storage
9. **Visual Workflow Designer** displays real-time execution progress

### Error Handling Flow
1. **Execution Engine** detects step failure
2. **Execution Engine** applies retry logic (if configured)
3. **Execution Engine** reports error to Workflow Management API
4. **Workflow Management API** streams error to Visual Workflow Designer
5. **User** reviews error and can retry or cancel workflow
6. **Execution Engine** handles partial success (optional steps can fail)

---

## 4. Technology Stack Decisions

### Frontend (Visual Workflow Designer)
- **Framework:** React (recommended) or Vue.js
- **Drag-and-Drop:** React Flow (React) or Vue Flow (Vue.js)
- **UI Components:** Material-UI (React) or Vuetify (Vue.js)
- **State Management:** Redux/Zustand (React) or Pinia (Vue.js)
- **WebSocket:** Socket.io client or native WebSocket API

### Backend (Workflow Management API)
- **Framework:** FastAPI
- **WebSocket:** FastAPI WebSocket
- **Authentication:** Authentication (can integrate with project auth)
- **API Documentation:** drf-spectacular (OpenAPI)

### Core Engine (Workflow Execution Engine)
- **Language:** Python 3.13+
- **Framework:** Framework-agnostic (no framework dependencies)
- **Plugin System:** Python importlib for dynamic loading
- **Schema Validation:** JSON Schema (jsonschema library)
- **Async:** Python asyncio for parallel step execution

### Storage (Workflow Storage)
- **Workflow Definitions:** JSON/YAML files (recommended for version control)
- **Execution History:** PostgreSQL (optional, can start with files)
- **Configuration:** JSON/YAML files

### Plugins
- **Language:** Python
- **Interface:** Abstract base classes with clear contracts
- **Discovery:** Entry point registration or file-based discovery
- **Isolation:** Can run in separate processes for security-critical plugins

---

## 5. Deployment & Scaling Considerations

### Deployment Topology

**Option 1: Integrated with parent project (Initial)**
- Visual Workflow Designer: Served as static files or separate service
- Workflow Management API: FastAPI service within parent project
- Execution Engine: Python module within FastAPI service
- Plugins: Python modules loaded dynamically
- Storage: File system for definitions, PostgreSQL for execution history

**Option 2: Standalone Service (Future Open-Source)**
- Visual Workflow Designer: Separate frontend service (React/Vue SPA)
- Workflow Management API: Standalone FastAPI service
- Execution Engine: Python package installable independently
- Plugins: Installable Python packages
- Storage: Configurable (files or database)

### Scaling Strategies

**Frontend (Visual Workflow Designer):**
- Static file hosting (CDN for production)
- Horizontal scaling via load balancer
- WebSocket connections managed by FastAPI WebSocket

**Backend (Workflow Management API):**
- Horizontal scaling via FastAPI/ASGI (Gunicorn/uWSGI)
- FastAPI WebSocket for WebSocket scaling (Redis as channel layer)
- Database connection pooling for PostgreSQL

**Core Engine (Execution Engine):**
- Stateless design enables horizontal scaling
- Step execution can be distributed via message queue (Celery, optional)
- Parallel step execution within single workflow

**Storage:**
- File system: NFS/shared storage for multi-instance deployment
- Database: PostgreSQL with read replicas for execution history queries

### Resource Requirements

**Development:**
- Frontend: Node.js dev server
- Backend: FastAPI dev server
- Database: Local PostgreSQL (optional)
- Storage: Local file system

**Production:**
- Frontend: Static file hosting (100MB+)
- Backend: 512MB RAM, 1 CPU core minimum
- Database: PostgreSQL (existing project DB or separate)
- Storage: File system (100MB+ for workflow definitions)

---

## 6. Extension Points

### Adding New Workflow Types
1. Create plugin directory structure in `plugins/<workflow_type>/`
2. Implement `WorkflowTypePlugin` interface
3. Register workflow type in plugin registry
4. Define workflow templates (optional)
5. Add workflow type to Visual Designer palette

### Adding New Step Handlers
1. Create step handler class implementing `StepHandler` interface
2. Implement `execute()`, `validate_config()`, `get_config_schema()`
3. Register handler in plugin registry
4. Add handler to step palette in Visual Designer

### Adding New Integrations
1. Create integration adapter implementing integration interface
2. Register adapter in plugin registry
3. Create step handlers that use adapter
4. Configure adapter in workflow definition

### Custom Storage Backends
1. Implement storage interface (file-based or database)
2. Register storage backend in configuration
3. Execution engine uses registered backend

---

## 7. Security Considerations

### Input Validation
- All workflow definitions validated against JSON Schema
- All user parameters validated against parameter schemas
- File paths validated to prevent directory traversal

### File System Access
- Restricted to workspace directory (configurable)
- No execution of arbitrary commands
- Git operations validated before execution

### Authentication & Authorization
- Integrate with Authentication
- Role-based access control (RBAC) for workflow operations
- Audit logging for workflow executions

### Plugin Security
- Plugin isolation (optional process isolation for untrusted plugins)
- Plugin signature validation (future)
- Sandboxed execution for security-critical plugins

---

## 8. Open Questions

1. **Deployment Model:** Integrated with parent project initially, or standalone from start?
2. **Storage Strategy:** File-based only, or database support from the beginning?
3. **Plugin Isolation:** Process isolation for plugins, or trust model?
4. **Multi-tenancy:** Support multiple projects/workspaces, or single project initially?
5. **Workflow Sharing:** Share workflows across projects, or project-specific only?

---

## 9. Related Documents

- **Context View:** `KB/Architecture/context/overview/epic01-vwmp-context.md`
- **Component View:** `KB/Architecture/component/overview/epic01-vwmp-components.md`
- **Runtime View:** `KB/Architecture/runtime/reference/epic01-vwmp-sequences.md`
- **Integration Architecture:** `KB/Architecture/Integration_Architecture/vwmp/vwmp-integration-architecture.md`
- **Main Architecture:** `KB/Architecture/Standards_and_ADRs/E01-vwmp-platform-architecture.md`

---

**Next Steps:** Proceed to Component View for detailed component breakdown
