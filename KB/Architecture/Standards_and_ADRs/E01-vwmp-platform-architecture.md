# Epic 01: Visual Workflow Management Platform - Architecture Design

**Status:** Draft - In Progress
**Epic:** Epic 01 - Visual Workflow Management Platform (VWMP)
**Task:** E01:S01:T001 - Design general-purpose workflow management platform architecture
**Created:** 2025-11-20
**Version:** v0.21.0.1+

---

## Executive Summary

This document defines the architecture for the Visual Workflow Management Platform (VWMP), a general-purpose workflow management system designed for visual, intuitive workflow definition, execution, and monitoring. The platform is architected for extensibility and independence, enabling future open-source spin-off potential.

**Key Design Principles:**
- **Visual and Intuitive:** NOT CLI-based, accessible to all team members
- **Plugin-Based:** Extensible architecture with clear separation of concerns
- **Configuration-Driven:** Workflows defined via YAML/JSON, not hardcoded
- **Framework-Agnostic Core:** Core engine independent of project-specific integrations
- **⚠️ Lightweight:** Minimize compute overhead when developers use VWMP in their projects (PRIMARY)
- **Open-Source Ready:** Architecture designed for future standalone project

---

## Architecture Overview

### High-Level Architecture

**Rationale for Layered Architecture:**
- **Separation of Concerns:** Each layer has clear responsibilities
- **Framework Independence:** Frontend and Core are framework-agnostic
- **Spin-Off Ready:** Each layer can be extracted independently
- **Extensibility:** Plugin system allows adding new workflow types
- **Lightweightness:** Optional API layer (headless mode available)

```
┌─────────────────────────────────────────────────────────────────┐
│                    Visual Workflow Designer                      │
│                  (Frontend UI - React + React Flow)              │
│  - Drag-and-drop workflow builder                                │
│  - Step configuration panels                                     │
│  - Parameter input forms                                         │
│  - Dependency visualization                                      │
│  - Workflow preview                                              │
│  Rationale: React Flow is purpose-built for visual editors      │
└────────────────────┬────────────────────────────────────────────┘
                     │ REST API / WebSocket
┌────────────────────▼────────────────────────────────────────────┐
│                    Workflow Management API                       │
│                    (FastAPI ⚠️ RECOMMENDED)                      │
│  - Workflow CRUD operations                                      │
│  - Execution control (start, stop, pause)                       │
│  - Status and monitoring endpoints                              │
│  - Configuration management                                      │
│  - WebSocket real-time updates (built-in)                       │
│  Rationale: Lightweight (5x less memory, 5x faster startup)     │
│  Note: Optional - headless mode available (no API server)       │
└────────────────────┬────────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────────┐
│                    Workflow Execution Engine                     │
│                  (Core Platform - Framework-Agnostic)            │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Workflow Parser & Validator                              │   │
│  │ - Parse YAML/JSON workflow definitions                   │   │
│  │ - Validate schema and dependencies                       │   │
│  └──────────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Step Execution Orchestrator                              │   │
│  │ - Dependency resolution                                  │   │
│  │ - Step execution order                                   │   │
│  │ - Error handling and recovery                            │   │
│  └──────────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Plugin System                                            │   │
│  │ - Step handler registration                              │   │
│  │ - Workflow type plugins                                  │   │
│  │ - Integration adapters                                   │   │
│  └──────────────────────────────────────────────────────────┘   │
│  Rationale: Framework-agnostic for flexibility and spin-off     │
└────────────────────┬────────────────────────────────────────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
┌───────▼────────┐      ┌─────────▼──────────┐
│ Generic        │      │ Confidentia        │
│ Plugins        │      │ Plugins            │
│ (Open-Source)  │      │ (Proprietary)      │
│                │      │                    │
│ - Release      │      │ - KB Updater       │
│ - Git          │      │ - Kanban Updater   │
│ - Filesystem   │      │ - Validators       │
│ - Future Types │      │                    │
│                │      │                    │
│ Rationale:     │      │ Rationale:         │
│ Generic        │      │ Project-specific   │
│ functionality  │      │ code isolated      │
│ reusable       │      │                    │
└────────────────┘      └────────────────────┘
```

### Core Components

#### 1. Visual Workflow Designer (Frontend)
- **Technology:** React 18 + React Flow 11 (see [Frontend Technology Decision ADR](E01-vwmp-frontend-technology-decision.md))
- **Responsibilities:**
  - Visual workflow creation and editing
  - Step configuration UI
  - Parameter input and validation
  - Dependency visualization
  - Workflow preview
  - Execution monitoring UI
- **Rationale:**
  - **React Flow is purpose-built** for node-based visual editors
  - **Mature and proven** - used in production by many companies
  - **Rapid development** - CDN approach enables immediate start
  - **Standalone ready** - can be deployed as standalone SPA
  - **Future flexibility** - can migrate to npm/Vite for optimization

#### 2. Workflow Management API (Backend)
- **Technology:** FastAPI (⚠️ **RECOMMENDED** - see [FastAPI Decision ADR](E01-vwmp-fastapi-decision.md))
- **Responsibilities:**
  - Workflow definition CRUD operations
  - Workflow execution control
  - Status and monitoring endpoints
  - Configuration management
  - WebSocket for real-time updates (built-in)
- **Note:** Django REST Framework was used initially but FastAPI is recommended for lightweightness (5x less memory, 5x faster startup)

#### 3. Workflow Execution Engine (Core)
- **Technology:** Python 3.13+ (framework-agnostic)
- **Responsibilities:**
  - Workflow parsing and validation
  - Step execution orchestration
  - Dependency resolution
  - Error handling and recovery
  - Plugin system management
- **Rationale:**
  - **Framework-agnostic:** No dependencies on Django, FastAPI, or any web framework
  - **Standalone ready:** Can be used independently (headless mode)
  - **Spin-off ready:** Easy to extract for standalone open-source project
  - **Plugin support:** Dynamic plugin loading via importlib (standard library)

#### 4. Plugin System
- **Technology:** Python plugin architecture (importlib)
- **Responsibilities:**
  - Workflow type registration (Release, CI/CD, etc.)
  - Step handler registration
  - Integration adapter management
  - Configuration schema validation
- **Rationale:**
  - **Separation of Concerns:** Isolates Confidentia-specific code from generic core
  - **Spin-Off Ready:** Generic plugins can be open-sourced independently
  - **Extensibility:** Community can add custom workflow types and handlers
  - **Maintainability:** Clear boundaries make codebase easier to understand

#### 5. Workflow Storage
- **Technology:** File-based (JSON/YAML) - primary, optional database for multi-user
- **Responsibilities:**
  - Workflow definition persistence
  - Version history
  - Execution history (optional)
  - Configuration storage
- **Rationale:**
  - **Simplicity:** No database setup required - works immediately
  - **Version Control:** Workflow definitions can be tracked in Git
  - **Portability:** Easy to backup, restore, and migrate
  - **Lightweight:** File-based storage has minimal overhead
  - **Standalone Ready:** Perfect for single-user deployments

---

## Workflow Definition Schema

### YAML Schema (Example)

```yaml
# workflow-definition.yaml
workflow:
  name: "Release Workflow"
  version: "1.0.0"
  type: "release"  # Plugin type
  description: "Automated release workflow for version bump, changelog, commit, tag, push"

  # Workflow-level configuration
  config:
    version_file: "src/confidentia/version.py"
    changelog_dir: "KB/Changelog_and_Release_Notes/Changelog_Archive"
    main_changelog: "CHANGELOG.md"

  # Steps definition
  steps:
    - id: "step-1"
      name: "Bump Version"
      type: "version_bump"
      handler: "release.version_bump"
      config:
        increment_type: "patch"  # patch, minor, major, feature, feature_set
        version_file: "${config.version_file}"
      required: true
      dependencies: []

    - id: "step-2"
      name: "Create Detailed Changelog"
      type: "changelog_create"
      handler: "release.changelog_create"
      config:
        changelog_dir: "${config.changelog_dir}"
        format: "full_timestamp"  # YYYY-MM-DD HH:MM:SS UTC
        epic_detection: "auto"  # from branch
        story_detection: "auto"  # from branch
      required: true
      dependencies: ["step-1"]

    - id: "step-3"
      name: "Update Main Changelog"
      type: "changelog_update"
      handler: "release.changelog_update"
      config:
        main_changelog: "${config.main_changelog}"
        date_format: "DD-MM-YY"
        link_template: "[CHANGELOG_v{version}.md]({path})"
      required: true
      dependencies: ["step-2"]

    - id: "step-4"
      name: "Update README"
      type: "readme_update"
      handler: "release.readme_update"
      config:
        readme_file: "README.md"
        update_badge: true
        update_latest_release: true
      required: false
      enabled: true  # Can be toggled
      dependencies: ["step-1"]

    - id: "step-5"
      name: "Auto-update Kanban Docs"
      type: "kanban_update"
      handler: "confidentia.kanban_update"
      config:
        epic_doc_pattern: "KB/PM_and_Portfolio/epics/overview/Epic {epic}/Epic-{epic}.md"
        kanban_board: "KB/PM_and_Portfolio/epics/overview/_index.md"
      required: false
      automatic: true  # Always runs
      dependencies: ["step-1"]

    - id: "step-6"
      name: "Stage Files"
      type: "git_stage"
      handler: "git.stage_all"
      config:
        paths: ["*"]  # All files
      required: true
      dependencies: ["step-1", "step-2", "step-3", "step-4", "step-5"]

    - id: "step-7"
      name: "Run Validators"
      type: "validation"
      handler: "confidentia.run_validators"
      config:
        validators:
          - "scripts/validation/validate_branch_context.py"
          - "scripts/validation/validate_changelog_format.py"
        strict_mode: true
      required: true
      dependencies: ["step-6"]

    - id: "step-8"
      name: "Commit Changes"
      type: "git_commit"
      handler: "git.commit"
      config:
        message_template: "{version} - {summary}"
      required: true
      dependencies: ["step-7"]

    - id: "step-9"
      name: "Create Git Tag"
      type: "git_tag"
      handler: "git.create_tag"
      config:
        tag_template: "v{version}"
        message_template: "Release {tag}: {summary}"
        annotated: true
      required: false
      enabled: true  # Can be toggled
      dependencies: ["step-8"]

    - id: "step-10"
      name: "Push to Remote"
      type: "git_push"
      handler: "git.push"
      config:
        push_tags: true
        remote: "origin"
      required: false
      enabled: true  # Can be toggled
      dependencies: ["step-8", "step-9"]

  # Workflow parameters (user inputs)
  parameters:
    - name: "summary"
      type: "string"
      required: true
      label: "Release Summary"
      description: "Brief description of the release"
      validation:
        min_length: 10
        max_length: 200

    - name: "change_type"
      type: "enum"
      required: true
      default: "tooling"
      label: "Release Type"
      description: "Type of release"
      options:
        - { value: "feature", label: "Feature 🚀" }
        - { value: "fix", label: "Fix 🐞" }
        - { value: "infrastructure", label: "Infrastructure 🔧" }
        - { value: "security", label: "Security 🔒" }
        - { value: "cleanup", label: "Cleanup 🧹" }
        - { value: "refactor", label: "Refactor 🔄" }
        - { value: "documentation", label: "Documentation 📚" }
        - { value: "tooling", label: "Tooling 🧰" }
        - { value: "maintenance", label: "Maintenance 🚧" }

    - name: "detailed_changes"
      type: "textarea"
      required: false
      label: "Detailed Changes"
      description: "Optional detailed list of changes"
      validation:
        max_length: 2000

    - name: "skip_readme"
      type: "boolean"
      required: false
      default: false
      label: "Skip README Update"

    - name: "kb_docs_ok"
      type: "boolean"
      required: true
      default: false
      label: "KB Documentation Updated"
      description: "Confirm all required KB documentation is updated"
```

### JSON Schema Alternative

The same structure can be represented in JSON format for programmatic generation:

```json
{
  "workflow": {
    "name": "Release Workflow",
    "version": "1.0.0",
    "type": "release",
    "description": "Automated release workflow",
    "config": {
      "version_file": "src/confidentia/version.py",
      "changelog_dir": "KB/Changelog_and_Release_Notes/Changelog_Archive",
      "main_changelog": "CHANGELOG.md"
    },
    "steps": [
      {
        "id": "step-1",
        "name": "Bump Version",
        "type": "version_bump",
        "handler": "release.version_bump",
        "config": {
          "increment_type": "patch",
          "version_file": "${config.version_file}"
        },
        "required": true,
        "dependencies": []
      }
      // ... more steps
    ],
    "parameters": [
      {
        "name": "summary",
        "type": "string",
        "required": true,
        "label": "Release Summary"
      }
      // ... more parameters
    ]
  }
}
```

---

## Core Engine Architecture

### Component Breakdown

Following the KB Architecture standard component view structure, components are organized by layer with clear responsibilities:

| Component | Layer | Responsibilities | Notes |
| --- | --- | --- | --- |
| **WorkflowParser** | Execution Engine | Parse YAML/JSON workflow definitions, extract workflow structure, handle parsing errors | Framework-agnostic Python module |
| **WorkflowValidator** | Execution Engine | Validate workflow schema, check dependencies (no cycles), validate parameter schemas, validate handler references | Framework-agnostic Python module |
| **StepOrchestrator** | Execution Engine | Resolve execution order (topological sort), execute steps in parallel when possible, manage step state, collect step outputs | Framework-agnostic Python module |
| **StepExecutor** | Execution Engine | Execute individual steps via handlers, handle step timeouts, collect step results, manage step context | Framework-agnostic Python module |
| **PluginRegistry** | Plugin System | Register workflow type plugins, register step handlers, load and instantiate plugins, provide plugin configuration schemas | Framework-agnostic Python module |
| **WorkflowTypePlugin** | Plugin System | Define workflow type metadata, provide workflow templates, validate workflow type-specific rules | Abstract base class |
| **StepHandler** | Plugin System | Execute step logic, validate step configuration, provide configuration schema, handle step errors | Abstract base class |

See `KB/Architecture/component/overview/epic01-vwmp-components.md` for complete component breakdown with all components.

#### 1. Workflow Parser & Validator

**Purpose:** Parse and validate workflow definitions

**Responsibilities:**
- Parse YAML/JSON workflow definitions
- Validate schema against workflow definition schema
- Validate step dependencies (no circular dependencies)
- Validate parameter schemas
- Validate handler references

**Interface:**
```python
class WorkflowParser:
    def parse(self, workflow_file: Path) -> WorkflowDefinition
    def validate(self, workflow: WorkflowDefinition) -> ValidationResult
```

**WorkflowDefinition Model:**
```python
@dataclass
class WorkflowDefinition:
    name: str
    version: str
    type: str  # Plugin type
    description: str
    config: Dict[str, Any]
    steps: List[StepDefinition]
    parameters: List[ParameterDefinition]

@dataclass
class StepDefinition:
    id: str
    name: str
    type: str
    handler: str  # Plugin handler path
    config: Dict[str, Any]
    required: bool
    enabled: bool
    dependencies: List[str]  # Step IDs

@dataclass
class ParameterDefinition:
    name: str
    type: str  # string, enum, boolean, textarea, etc.
    required: bool
    default: Optional[Any]
    label: str
    description: Optional[str]
    validation: Optional[Dict[str, Any]]
    options: Optional[List[Dict[str, str]]]  # For enum type
```

#### 2. Step Execution Orchestrator

**Purpose:** Orchestrate step execution based on dependencies

**Responsibilities:**
- Resolve step execution order (topological sort)
- Execute steps in parallel when possible
- Handle step failures and retries
- Manage step state (pending, running, completed, failed, skipped)
- Collect step outputs for use in dependent steps

**Interface:**
```python
class StepOrchestrator:
    def resolve_execution_order(self, workflow: WorkflowDefinition) -> List[List[str]]
    def execute_step(self, step: StepDefinition, context: ExecutionContext) -> StepResult
    def execute_workflow(self, workflow: WorkflowDefinition, params: Dict[str, Any]) -> WorkflowResult
```

**Execution Context:**
```python
@dataclass
class ExecutionContext:
    workflow: WorkflowDefinition
    parameters: Dict[str, Any]
    step_outputs: Dict[str, StepResult]  # Results from completed steps
    config: Dict[str, Any]
    workspace_root: Path

@dataclass
class StepResult:
    step_id: str
    status: StepStatus  # SUCCESS, FAILED, SKIPPED, RUNNING, PENDING
    output: Optional[Dict[str, Any]]
    error: Optional[str]
    execution_time: Optional[float]
    logs: List[str]

@dataclass
class WorkflowResult:
    workflow_id: str
    status: WorkflowStatus  # SUCCESS, FAILED, PARTIAL, RUNNING, PENDING
    step_results: Dict[str, StepResult]
    total_time: float
    started_at: datetime
    completed_at: Optional[datetime]
```

#### 3. Plugin System

**Purpose:** Extensible plugin architecture for workflow types and step handlers

**Responsibilities:**
- Register workflow type plugins
- Register step handlers
- Load and instantiate plugins
- Provide plugin configuration schemas
- Manage plugin dependencies

**Interface:**
```python
class PluginRegistry:
    def register_workflow_type(self, plugin: WorkflowTypePlugin)
    def register_step_handler(self, handler: StepHandler)
    def get_handler(self, handler_path: str) -> StepHandler
    def get_workflow_type(self, type_name: str) -> WorkflowTypePlugin

class StepHandler(ABC):
    @abstractmethod
    def execute(self, step: StepDefinition, context: ExecutionContext) -> StepResult

    @abstractmethod
    def validate_config(self, config: Dict[str, Any]) -> ValidationResult

    @abstractmethod
    def get_config_schema(self) -> Dict[str, Any]  # JSON Schema
```

**Plugin Structure:**
```
plugins/
  release/
    __init__.py
    workflow_type.py      # Release workflow type plugin
    handlers/
      version_bump.py
      changelog_create.py
      changelog_update.py
      readme_update.py
  confidentia/
    handlers/
      kanban_update.py
      run_validators.py
  git/
    handlers/
      stage_all.py
      commit.py
      create_tag.py
      push.py
```

---

## Visual Designer Architecture

### Frontend Components

#### 1. Workflow Canvas
- **Technology:** React Flow or Vue Flow
- **Features:**
  - Drag-and-drop step placement
  - Visual dependency connections (edges)
  - Step status visualization (color-coded)
  - Zoom and pan controls

#### 2. Step Configuration Panel
- **Technology:** React/Vue components
- **Features:**
  - Dynamic form generation from parameter schema
  - Input validation
  - Required field indicators
  - Default value population
  - Help text and tooltips

#### 3. Step Palette
- **Technology:** React/Vue components
- **Features:**
  - Available step types (from registered plugins)
  - Search and filter
  - Step type descriptions
  - Drag-and-drop to canvas

#### 4. Workflow Preview
- **Technology:** React/Vue components
- **Features:**
  - Visual representation of workflow
  - Execution order preview
  - Parameter summary
  - Validation status

#### 5. Execution Monitor
- **Technology:** React/Vue + WebSocket
- **Features:**
  - Real-time step status updates
  - Progress bars
  - Log streaming
  - Error display
  - Execution history

---

## Data Flow

### Workflow Creation Flow

```
1. User creates workflow in Visual Designer
   ↓
2. Designer generates workflow definition (YAML/JSON)
   ↓
3. Definition sent to API for validation
   ↓
4. Validated definition stored (file or database)
   ↓
5. Workflow available for execution
```

### Workflow Execution Flow

```
1. User selects workflow and provides parameters
   ↓
2. API validates parameters against schema
   ↓
3. Execution Engine parses workflow definition
   ↓
4. Orchestrator resolves execution order
   ↓
5. Steps executed in order (parallel when possible)
   ↓
6. Each step handler executed via Plugin System
   ↓
7. Step results collected and passed to next steps
   ↓
8. Execution status sent to frontend via WebSocket
   ↓
9. Workflow completion status returned
```

---

## Configuration System

### Configuration Hierarchy

1. **Global Configuration** (platform defaults)
2. **Workflow Type Configuration** (plugin defaults)
3. **Workflow-Level Configuration** (workflow definition)
4. **Step-Level Configuration** (step definition)
5. **Runtime Parameters** (user input)

### Configuration Resolution

Configuration values are resolved with variable substitution:
- `${config.version_file}` → Resolved from workflow config
- `${parameters.summary}` → Resolved from user input
- `${step-outputs.step-1.new_version}` → Resolved from step result
- `${env.WORKSPACE_ROOT}` → Environment variable

---

## Error Handling & Recovery

### Error Types

1. **Validation Errors:** Invalid workflow definition or parameters
2. **Execution Errors:** Step handler failures
3. **Dependency Errors:** Missing dependencies or circular references
4. **Configuration Errors:** Invalid configuration values

### Recovery Strategies

1. **Retry:** Configurable retry logic for transient failures
2. **Skip Optional Steps:** Continue workflow if optional step fails
3. **Rollback:** Undo completed steps (if supported by handler)
4. **Partial Success:** Report workflow as partially successful

---

## Security Considerations

1. **Input Validation:** All user inputs validated against schemas
2. **Handler Isolation:** Step handlers run in isolated context
3. **File System Access:** Restricted to workspace directory
4. **Git Operations:** Validate git commands before execution
5. **Permission Checks:** Verify user permissions for workflow execution

---

## Performance Considerations

1. **Parallel Execution:** Execute independent steps in parallel
2. **Caching:** Cache parsed workflow definitions
3. **Async Operations:** Use async/await for I/O operations
4. **WebSocket:** Efficient real-time updates
5. **Database Optimization:** Index workflow and execution history

---

## Extensibility Points

### Adding New Workflow Types

1. Create plugin directory structure
2. Implement `WorkflowTypePlugin` interface
3. Register workflow type in plugin registry
4. Define workflow templates (optional)

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

---

## Technology Stack Decisions

### Frontend

**Decision:** React + React Flow (see [Frontend Technology Decision ADR](E01-vwmp-frontend-technology-decision.md))

**Rationale:**
- **React Flow is purpose-built** for node-based visual editors - exactly what we need
- **Mature and proven** - used in production by many companies, active community
- **Rapid development** - CDN approach enables immediate start, no build pipeline needed initially
- **Standalone ready** - can be deployed as standalone SPA or integrated into any backend
- **Future flexibility** - can migrate to npm/Vite later for production optimization

**Technology Stack:**
- **Framework:** React 18
- **Drag-and-Drop:** React Flow 11 (purpose-built for visual editors)
- **Build Tool:** Vite (for standalone deployment - migration from CDN)
- **WebSocket:** Native WebSocket API (standard, no dependencies)

**Rationale for React over Vue:**
- React Flow is more mature and proven than Vue Flow
- Larger ecosystem and community support
- Better documentation and examples for visual editors

---

### Backend

**Decision:** FastAPI ⚠️ **RECOMMENDED** (see [FastAPI Decision ADR](E01-vwmp-fastapi-decision.md))

**Rationale:**
- **⚠️ Lightweightness (PRIMARY):** 5x less memory (~20-50MB vs ~100-200MB), 5x faster startup (<1s vs ~2-5s), 10x smaller dependencies (~5MB vs ~50MB)
- **Minimal compute overhead** when developers embed VWMP in their projects
- **Built-in WebSocket** - no additional dependencies (unlike Django Channels which requires Redis)
- **Automatic API docs** - OpenAPI/Swagger auto-generated
- **Modern and popular** - growing Python community, excellent developer experience
- **Async native** - full async/await support for efficient resource usage
- **Headless support** - API layer is optional, core engine can be used without API server

**Technology Stack:**
- **Framework:** FastAPI (modern, fast Python web framework)
- **Server:** Uvicorn (ASGI server)
- **WebSocket:** FastAPI built-in (no separate library)
- **Schema Validation:** Pydantic (built-in) + JSON Schema (via jsonschema library)
- **Async:** Python asyncio for parallel step execution (native async/await)

**Rationale for FastAPI over Django:**
- **Django is too heavy** - ~100-200MB memory overhead, ~2-5s startup time
- **Django Channels requires Redis** - additional dependency and overhead
- **Django is overkill** - VWMP doesn't need most Django features
- **High integration impact** - significant overhead on host applications

**Note:** Django REST Framework was initially used but FastAPI is **required** for lightweightness requirements.

---

### Storage

**Decision:** File-based (JSON/YAML) for workflow definitions, optional database for execution history

**Rationale:**
- **Simplicity:** No database setup required - works immediately
- **Version Control:** Workflow definitions can be versioned in Git
- **Portability:** Easy to backup and migrate workflows
- **Fast Setup:** No database configuration or migrations needed
- **Lightweight:** File-based storage has minimal overhead
- **Standalone Ready:** Perfect for single-user deployments

**Technology Stack:**
- **Workflow Definitions:** JSON/YAML files (primary, recommended for version control)
- **Execution History:** Optional database (PostgreSQL) - can be added later if needed
- **Configuration:** JSON/YAML files

**Rationale for File-based over Database:**
- **No database required** - simplifies deployment and setup
- **Version control friendly** - workflows can be tracked in Git
- **Portable** - easy to backup and restore
- **Fast** - direct file access, no database overhead
- **Sufficient** - most workflows are small enough for file storage

**Future Option:** SQLite can be added as optional storage backend for multi-user scenarios.

---

### Core Engine

**Decision:** Python 3.13+ (framework-agnostic)

**Rationale:**
- **Framework-agnostic:** No dependencies on Django or FastAPI - can be used with any backend or standalone
- **Modern Python:** Python 3.13+ provides latest features (type hints, async/await, performance improvements)
- **Plugin System:** Python's importlib enables dynamic plugin loading without heavy frameworks
- **Validation:** JSON Schema provides standard validation, jsonschema library is lightweight
- **Standalone Ready:** Core engine can be used independently, perfect for CLI or script embedding

**Technology Stack:**
- **Language:** Python 3.13+
- **Plugin System:** importlib for dynamic loading (standard library, no dependencies)
- **Validation:** JSON Schema (via jsonschema library) + custom validators
- **Dependencies:** Minimal (pyyaml, jsonschema, pydantic if using FastAPI)

**Rationale for Framework-Agnostic Design:**
- **Independence:** Core engine doesn't depend on any web framework
- **Flexibility:** Can be used with FastAPI, Flask, or standalone
- **Headless Mode:** Enables CLI and script usage without API server
- **Spin-off Ready:** Easy to extract for standalone open-source project

---

## Implementation Phases

### Phase 1: Core Engine (Foundation)
- Workflow definition schema
- Parser and validator
- Step orchestrator (basic)
- Plugin system (basic)

### Phase 2: Basic Execution
- Step handler interface
- Release workflow plugin (first use case)
- Git handlers
- Basic error handling

### Phase 3: Visual Designer
- Workflow canvas
- Step configuration panels
- Parameter input forms
- Workflow preview

### Phase 4: Full Integration
- WebSocket real-time updates
- Execution monitoring
- Workflow management (CRUD)
- Release workflow complete integration

---

## Open Questions

1. **Storage:** File-based (JSON/YAML) vs Database for workflow definitions?
   - **Decision:** File-based (JSON/YAML) ⚠️
   - **Rationale:** Simplicity (no database setup), version control friendly, portable, fast, sufficient for most use cases
   - **Future Option:** SQLite can be added for multi-user scenarios

2. **Frontend Framework:** React vs Vue.js?
   - **Decision:** React ⚠️ (see [Frontend Technology Decision ADR](E01-vwmp-frontend-technology-decision.md))
   - **Rationale:** React Flow is more mature and proven than Vue Flow, larger ecosystem, better documentation
   - **Selected:** React 18 + React Flow 11 for purpose-built visual editor functionality

3. **WebSocket Library:** Django Channels vs Socket.io vs FastAPI built-in?
   - **Decision:** FastAPI built-in WebSocket ⚠️ (see [FastAPI Decision ADR](E01-vwmp-fastapi-decision.md))
   - **Rationale:** FastAPI selected for lightweightness (5x less memory, 5x faster startup)
   - **No Redis required** (unlike Django Channels)

4. **Step Isolation:** Process isolation vs thread-based?
   - **Recommendation:** Thread-based initially, process isolation for security-critical steps

5. **Rollback Strategy:** Full rollback vs manual recovery?
   - **Recommendation:** Start with manual recovery, add rollback for critical workflows later

---

## Next Steps

1. **Review and Approve Architecture:** Stakeholder review of all C4 views and integration architecture
2. **Finalize Technology Stack:** Confirm React vs Vue, storage strategy decisions
3. **Create ADRs:** Document key architectural decisions (medium priority)
4. **Prototype Core Engine:** Build minimal viable core engine (Phase 2)
5. **Design Plugin Interface:** Define detailed plugin interfaces (Phase 2)

---

## Related Documents

### Epic & Story Documentation
- **Epic 01 Overview:** `KB/PM_and_Portfolio/epics/overview/Epic 01/Epic-21.md`
- **Story 1 Details:** `KB/PM_and_Portfolio/stories/overview/Epic 01/Story-1-Visual-Workflow-Management-Platform.md`
- **Development Plan:** `KB/PM_and_Portfolio/epics/overview/Epic 01/Epic-21-Development-Plan.md`
- **Evaluation Report:** `KB/PM_and_Portfolio/stories/overview/Epic 20/Story-10-Recommendation-Report.md`
- **Release Requirements:** `KB/PM_and_Portfolio/stories/overview/Epic 20/Story-10-Release-Requirements.md`

### KB Architecture Documentation (C4 Model Views)
- **Context View (C4 Level 1):** `KB/Architecture/context/overview/epic01-vwmp-context.md` - VWMP in relation to external actors and systems
- **Container View (C4 Level 2):** `KB/Architecture/container/overview/epic01-vwmp-container.md` - Services/modules and their interactions
- **Component View (C4 Level 3):** `KB/Architecture/component/overview/epic01-vwmp-components.md` - Internal components with responsibility tables
- **Runtime View:** `KB/Architecture/runtime/reference/epic01-vwmp-sequences.md` - Sequence diagrams for workflow creation and execution

### Integration Architecture
- **Integration Architecture:** `KB/Architecture/Integration_Architecture/vwmp/vwmp-integration-architecture.md` - Integration points with Git, KB, Kanban, validators
- **Gap Analysis:** `KB/Architecture/Integration_Architecture/vwmp/VWMP-Architecture-Gap-Analysis.md` - Alignment with KB Architecture standards
- **Completion Summary:** `KB/Architecture/Integration_Architecture/vwmp/VWMP-Architecture-Completion-Summary.md` - Summary of completed architecture documentation

### Related Architecture Patterns
- **SET2 Plugin Architecture:** `KB/Architecture/Standards_and_ADRs/adr-0005-set2-autonomous-architecture.md` - Plugin boundaries and error isolation patterns
- **SIT Architecture:** `KB/Architecture/Integration_Architecture/sit/architecture-specification.md` - Layered architecture patterns
- **Cloud Storage Integration:** `KB/Architecture/Integration_Architecture/cloud-storage-architecture.md` - Integration architecture patterns

### External Research
- Framework Evaluations (GitHub Actions, CircleCI, Jenkins) - See Epic 20 Story 10
- Comparison Matrix - See Epic 20 Story 10
- Plugin Architecture Patterns - See SET2 ADR-0005
- Visual Designer Libraries (React Flow, Vue Flow) - To be researched

---

**Document Status:** Draft - Ready for Review
**Last Updated:** 2025-11-20
**Next Review:** After stakeholder feedback
