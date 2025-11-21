# Epic 01 – VWMP Component View (C4 Level 3)

**Status:** Draft
**Epic:** Epic 01 - Visual Workflow Management Platform (VWMP)
**Last Updated:** 2025-11-20
**Related Artifacts:**
- `KB/Architecture/container/overview/epic01-vwmp-container.md`
- `KB/Architecture/Standards_and_ADRs/E01-vwmp-platform-architecture.md`
- `KB/Architecture/Diagrams/Component/epic01-vwmp-components.puml`

---

## 1. Overview

This document decomposes the VWMP platform into major components aligned with the Visual Designer, API layer, Execution Engine, Plugin System, and Storage layers. It highlights responsibilities, key interactions, and extensibility points supporting Tasks E01:S01:T001–E01:S01:T010.

---

## 2. Component Responsibilities

| Component | Layer | Responsibilities | Notes |
| --- | --- | --- | --- |
| **WorkflowCanvas** | Visual Designer | Drag-and-drop workflow builder UI, step placement, dependency visualization, zoom/pan controls | React/Vue component using React Flow/Vue Flow |
| **StepConfigurationPanel** | Visual Designer | Dynamic form generation from parameter schemas, input validation, required field indicators, help text | React/Vue component with form library |
| **StepPalette** | Visual Designer | Available step types display, search and filter, step descriptions, drag-to-canvas functionality | React/Vue component with plugin registry integration |
| **WorkflowPreview** | Visual Designer | Visual workflow representation, execution order preview, parameter summary, validation status | React/Vue component |
| **ExecutionMonitor** | Visual Designer | Real-time step status updates, progress bars, log streaming, error display, execution history | React/Vue component + WebSocket client |
| **WorkflowController** | API Layer | Workflow CRUD operations, parameter validation, execution request handling, status endpoint management | FastAPI Router |
| **ExecutionController** | API Layer | Execution control (start, stop, pause), status streaming, WebSocket connection management | FastAPI + WebSocket |
| **WebSocketHandler** | API Layer | Real-time status updates, log streaming, execution events, connection management | FastAPI WebSocket endpoint |
| **WorkflowParser** | Execution Engine | Parse YAML/JSON workflow definitions, validate schema, check syntax errors | Framework-agnostic Python module |
| **WorkflowValidator** | Execution Engine | Validate workflow structure, check dependencies (no cycles), validate parameter schemas, validate handler references | Framework-agnostic Python module |
| **StepOrchestrator** | Execution Engine | Resolve execution order (topological sort), execute steps in parallel when possible, manage step state, collect step outputs | Framework-agnostic Python module |
| **StepExecutor** | Execution Engine | Execute individual steps via handlers, handle step timeouts, collect step results, manage step context | Framework-agnostic Python module |
| **PluginRegistry** | Plugin System | Register workflow type plugins, register step handlers, load and instantiate plugins, provide plugin configuration schemas | Framework-agnostic Python module |
| **WorkflowTypePlugin** | Plugin System | Define workflow type metadata, provide workflow templates, validate workflow type-specific rules | Abstract base class |
| **StepHandler** | Plugin System | Execute step logic, validate step configuration, provide configuration schema, handle step errors | Abstract base class |
| **ReleaseWorkflowType** | Release Plugin | Release workflow type definition, release-specific validation, workflow templates | Implements WorkflowTypePlugin |
| **VersionBumpHandler** | Release Plugin | Read current version, increment version, update version file | Implements StepHandler |
| **ChangelogCreateHandler** | Release Plugin | Create detailed changelog file with full timestamp, extract epic/story from branch | Implements StepHandler |
| **ChangelogUpdateHandler** | Release Plugin | Update main CHANGELOG.md with summary entry, insert in Recent Releases section | Implements StepHandler |
| **ReadmeUpdateHandler** | Release Plugin | Update README.md version badge and latest release callout | Implements StepHandler |
| **GitStageHandler** | Git Plugin | Stage files via git add, validate git operations | Implements StepHandler |
| **GitCommitHandler** | Git Plugin | Create git commit with version number in message | Implements StepHandler |
| **GitTagHandler** | Git Plugin | Create annotated git tag, validate tag name | Implements StepHandler |
| **GitPushHandler** | Git Plugin | Push commits and tags to remote, handle authentication | Implements StepHandler |
| **KanbanUpdateHandler** | Project Plugin | Auto-update Kanban documentation (epic/story docs, board entries) | Implements StepHandler, uses update_kanban_docs.py |
| **ValidatorExecutionHandler** | Project Plugin | Execute validator scripts (branch context, changelog format), handle validation errors | Implements StepHandler |
| **WorkflowDefinitionStore** | Storage | Persist workflow definitions (YAML/JSON), version history, workflow metadata | File-based or database |
| **ExecutionHistoryStore** | Storage | Store execution history, step results, execution logs, metrics | File-based or database (PostgreSQL) |
| **ConfigurationStore** | Storage | Store workflow configurations, user preferences, plugin settings | File-based (JSON/YAML) |

See the component diagram for visual layout.

---

## 3. Key Interactions

### Workflow Creation Interaction
1. **User** drags steps from **StepPalette** onto **WorkflowCanvas**
2. **User** configures steps via **StepConfigurationPanel**
3. **WorkflowCanvas** generates workflow definition (YAML/JSON)
4. **WorkflowPreview** displays workflow with execution order
5. **User** saves workflow, **WorkflowController** validates and stores via **WorkflowDefinitionStore**

### Workflow Execution Interaction
1. **User** selects workflow and provides parameters via **StepConfigurationPanel**
2. **WorkflowController** validates parameters and triggers **StepOrchestrator**
3. **StepOrchestrator** loads workflow from **WorkflowDefinitionStore** via **WorkflowParser**
4. **WorkflowValidator** validates workflow definition
5. **StepOrchestrator** resolves execution order and calls **StepExecutor** for each step
6. **StepExecutor** loads step handler from **PluginRegistry** and executes step
7. **StepExecutor** collects step result and passes to dependent steps
8. **StepOrchestrator** streams status via **WebSocketHandler** to **ExecutionMonitor**
9. **ExecutionMonitor** displays real-time progress to user
10. **StepOrchestrator** stores execution history in **ExecutionHistoryStore**

### Plugin Registration Interaction
1. **PluginRegistry** discovers plugins via entry points or file scanning
2. **PluginRegistry** loads plugin module and validates plugin interface
3. **WorkflowTypePlugin** registers workflow type metadata
4. **StepHandler** registers step handler with handler path
5. **PluginRegistry** provides plugin metadata to **StepPalette** for UI display

### Error Handling Interaction
1. **StepExecutor** detects step failure
2. **StepExecutor** applies retry logic (if configured)
3. **StepExecutor** reports error to **StepOrchestrator**
4. **StepOrchestrator** streams error via **WebSocketHandler** to **ExecutionMonitor**
5. **ExecutionMonitor** displays error to user
6. **User** can retry step, skip optional step, or cancel workflow

---

## 4. Layer Separation

### Visual Designer Layer (Frontend)
- **Components:** WorkflowCanvas, StepConfigurationPanel, StepPalette, WorkflowPreview, ExecutionMonitor
- **Responsibilities:** UI rendering, user interaction, real-time updates
- **Technology:** React/Vue.js, React Flow/Vue Flow
- **Communication:** REST API and WebSocket

### API Layer (Backend)
- **Components:** WorkflowController, ExecutionController, WebSocketHandler
- **Responsibilities:** HTTP request handling, WebSocket management, authentication, authorization
- **Technology:** FastAPI, WebSocket
- **Communication:** REST API, WebSocket

### Execution Engine Layer (Core)
- **Components:** WorkflowParser, WorkflowValidator, StepOrchestrator, StepExecutor
- **Responsibilities:** Workflow parsing, validation, orchestration, step execution
- **Technology:** Python 3.13+ (framework-agnostic)
- **Communication:** Direct function calls, plugin registry

### Plugin System Layer
- **Components:** PluginRegistry, WorkflowTypePlugin, StepHandler, plugin implementations
- **Responsibilities:** Plugin discovery, registration, instantiation, execution
- **Technology:** Python importlib, abstract base classes
- **Communication:** Plugin interface contracts

### Storage Layer
- **Components:** WorkflowDefinitionStore, ExecutionHistoryStore, ConfigurationStore
- **Responsibilities:** Data persistence, version history, query interface
- **Technology:** File system (JSON/YAML) or PostgreSQL
- **Communication:** Storage interface

---

## 5. Extension Points

### Adding New Workflow Types
1. Create plugin directory: `plugins/<workflow_type>/`
2. Implement `WorkflowTypePlugin` interface
3. Register in plugin registry
4. Define workflow templates (optional)
5. Add to Visual Designer palette

### Adding New Step Handlers
1. Create step handler class implementing `StepHandler` interface
2. Implement `execute()`, `validate_config()`, `get_config_schema()`
3. Register handler in plugin registry
4. Add handler to step palette in Visual Designer

### Custom Storage Backends
1. Implement storage interface (file-based or database)
2. Register storage backend in configuration
3. Execution engine uses registered backend

### Custom Validation Rules
1. Extend `WorkflowValidator` with custom validation rules
2. Register validators in validator chain
3. Validators apply during workflow validation

---

## 6. Component Interfaces

### WorkflowParser Interface
```python
class WorkflowParser:
    def parse(self, workflow_file: Path) -> WorkflowDefinition
    def validate(self, workflow: WorkflowDefinition) -> ValidationResult
```

### StepOrchestrator Interface
```python
class StepOrchestrator:
    def resolve_execution_order(self, workflow: WorkflowDefinition) -> List[List[str]]
    def execute_workflow(self, workflow: WorkflowDefinition, params: Dict[str, Any]) -> WorkflowResult
```

### StepHandler Interface
```python
class StepHandler(ABC):
    @abstractmethod
    def execute(self, step: StepDefinition, context: ExecutionContext) -> StepResult

    @abstractmethod
    def validate_config(self, config: Dict[str, Any]) -> ValidationResult

    @abstractmethod
    def get_config_schema(self) -> Dict[str, Any]
```

### Storage Interface
```python
class WorkflowDefinitionStore:
    def save(self, workflow: WorkflowDefinition) -> None
    def load(self, workflow_id: str) -> WorkflowDefinition
    def list(self) -> List[WorkflowMetadata]
    def delete(self, workflow_id: str) -> None
```

---

## 7. Open Questions

1. **Component Granularity:** Should we split StepOrchestrator into smaller components?
2. **Error Recovery:** Should we add dedicated ErrorRecovery component?
3. **Caching:** Should we add WorkflowDefinitionCache component for performance?
4. **Metrics:** Should we add MetricsCollector component for observability?

---

## 8. Related Documents

- **Container View:** `KB/Architecture/container/overview/epic01-vwmp-container.md`
- **Runtime View:** `KB/Architecture/runtime/reference/epic01-vwmp-sequences.md`
- **Main Architecture:** `KB/Architecture/Standards_and_ADRs/E01-vwmp-platform-architecture.md`

---

_Next steps_: Align component tests and implementation backlog with this decomposition, ensure plugin interfaces map cleanly to execution engine contracts.
