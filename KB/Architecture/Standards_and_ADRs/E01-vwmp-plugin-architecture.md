# Epic 01: VWMP Plugin Architecture Design

**Status:** Draft - In Progress
**Epic:** Epic 01 - Visual Workflow Management Platform (VWMP)
**Task:** E01:S01:T002 - Design plugin-based architecture for independence (open-source potential)
**Created:** 2025-11-20
**Version:** v0.21.0.2+

---

## Executive Summary

This document defines the plugin architecture for the Visual Workflow Management Platform (VWMP), designed for extensibility, independence, and future open-source spin-off potential. The architecture follows established patterns from SET2/SIT while providing clear plugin boundaries for workflow types, step handlers, and integrations.

**Key Design Principles:**
- **Clear Plugin Boundaries:** Project-specific-specific code isolated in adapter plugins
- **Framework-Agnostic Core:** Core engine independent of project-specific integrations
- **Configuration-Driven:** No hardcoded paths or assumptions
- **Open-Source Ready:** Architecture designed for standalone deployment
- **Extensibility:** Easy to add new workflow types and step handlers

**Rationale for Plugin Architecture:**
- **Separation of Concerns:** Isolates Project-specific-specific code from generic core engine
- **Spin-Off Ready:** Core engine and generic plugins can be open-sourced independently
- **Extensibility:** Allows community to add custom workflow types and step handlers
- **Maintainability:** Clear boundaries make codebase easier to understand and maintain
- **Reusability:** Generic plugins can be used across different projects

---

## Plugin Architecture Overview

### Plugin Types

The VWMP plugin system supports three primary plugin types:

1. **Workflow Type Plugins** - Define workflow types (Release, CI/CD, Deployment, etc.)
2. **Step Handler Plugins** - Implement individual workflow step handlers
3. **Integration Adapter Plugins** - Provide integration adapters (Git, KB, Kanban, etc.)

### Plugin Hierarchy

```
┌─────────────────────────────────────────────────────────────────┐
│                    Plugin System                                 │
│                    (Core Platform)                               │
└────────────────────┬────────────────────────────────────────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
┌───────▼──────────┐   ┌──────────▼──────────┐
│ Workflow Type    │   │ Step Handler        │
│ Plugins          │   │ Plugins             │
│                  │   │                     │
│ - Release        │   │ - Version Bump      │
│ - CI/CD          │   │ - Changelog Create  │
│ - Deployment     │   │ - Git Commit        │
│ - Testing        │   │ - Kanban Update     │
│ - Documentation  │   │ - Validator Run     │
│                  │   │ - Custom Steps      │
└──────────────────┘   └─────────────────────┘
                                │
                     ┌──────────┴──────────┐
                     │                     │
           ┌─────────▼─────────┐  ┌───────▼──────────┐
           │ Integration       │  │ Project-specific      │
           │ Adapters          │  │ Adapters         │
           │                   │  │                  │
           │ - Git             │  │ - KB Updater     │
           │ - File System     │  │ - Kanban Updater │
           │ - HTTP/API        │  │ - Validators     │
           │ - Database        │  │                  │
           └───────────────────┘  └──────────────────┘
```

---

## Plugin Interface Design

### Base Plugin Interfaces

Following existing codebase patterns (SET2, spec_importer), VWMP uses Python ABC (Abstract Base Classes) with clear interface contracts.

#### 1. WorkflowTypePlugin Interface

**Purpose:** Define workflow types (Release, CI/CD, Deployment, etc.)

**Interface:**
```python
from abc import ABC, abstractmethod
from typing import Dict, Any, Optional, List
from dataclasses import dataclass

@dataclass
class WorkflowTypeMetadata:
    """Metadata for a workflow type."""
    name: str  # e.g., "release"
    version: str  # Plugin version
    description: str
    author: Optional[str] = None
    icon: Optional[str] = None  # Emoji or icon identifier

@dataclass
class WorkflowTemplate:
    """Template for creating new workflows of this type."""
    name: str
    description: str
    template: Dict[str, Any]  # Workflow definition YAML/JSON structure
    default_config: Dict[str, Any]  # Default configuration values

class WorkflowTypePlugin(ABC):
    """Base class for workflow type plugins."""

    @abstractmethod
    def get_metadata(self) -> WorkflowTypeMetadata:
        """Return workflow type metadata."""
        pass

    @abstractmethod
    def get_templates(self) -> List[WorkflowTemplate]:
        """Return available workflow templates for this type."""
        pass

    @abstractmethod
    def validate_workflow(self, workflow: Dict[str, Any]) -> ValidationResult:
        """Validate workflow definition for this type."""
        pass

    @abstractmethod
    def get_default_step_types(self) -> List[str]:
        """Return default step types available for this workflow type."""
        pass

    def get_config_schema(self) -> Dict[str, Any]:
        """Return JSON Schema for workflow-level configuration."""
        return {}

    def initialize(self, config: Dict[str, Any]) -> None:
        """Initialize plugin with configuration (optional)."""
        pass

    def cleanup(self) -> None:
        """Cleanup plugin resources (optional)."""
        pass
```

**Example Implementation:**
```python
class ReleaseWorkflowTypePlugin(WorkflowTypePlugin):
    """Plugin for release workflow type."""

    def get_metadata(self) -> WorkflowTypeMetadata:
        return WorkflowTypeMetadata(
            name="release",
            version="1.0.0",
            description="Release workflow for version bump, changelog, commit, tag, push",
            author="Project-specific Team",
            icon="🚀"
        )

    def get_templates(self) -> List[WorkflowTemplate]:
        return [
            WorkflowTemplate(
                name="Standard Release",
                description="Standard release workflow with all steps",
                template={...},  # Workflow definition
                default_config={...}
            ),
            WorkflowTemplate(
                name="Quick Release",
                description="Release workflow without README update",
                template={...},
                default_config={"skip_readme": True}
            )
        ]

    def validate_workflow(self, workflow: Dict[str, Any]) -> ValidationResult:
        # Validate release workflow-specific rules
        # e.g., must have version_bump step, changelog steps, etc.
        pass

    def get_default_step_types(self) -> List[str]:
        return [
            "release.version_bump",
            "release.changelog_create",
            "release.changelog_update",
            "release.readme_update",
            "vwmp.kanban_update",
            "git.stage_all",
            "vwmp.run_validators",
            "git.commit",
            "git.create_tag",
            "git.push"
        ]
```

---

#### 2. StepHandler Interface

**Purpose:** Implement individual workflow step handlers

**Interface:**
```python
from abc import ABC, abstractmethod
from typing import Dict, Any, Optional
from dataclasses import dataclass

@dataclass
class StepContext:
    """Execution context for a step."""
    workflow_id: str
    workflow_name: str
    step_id: str
    step_name: str
    step_config: Dict[str, Any]
    parameters: Dict[str, Any]  # User-provided parameters
    workspace_root: Path
    step_outputs: Dict[str, StepResult]  # Results from previous steps
    config: Dict[str, Any]  # Workflow-level configuration

@dataclass
class StepResult:
    """Result of step execution."""
    step_id: str
    status: StepStatus  # SUCCESS, FAILED, SKIPPED, RUNNING, PENDING
    output: Optional[Dict[str, Any]]  # Step output (can be used by dependent steps)
    error: Optional[str]  # Error message if failed
    execution_time: Optional[float]  # Execution time in seconds
    logs: List[str]  # Step execution logs

class StepStatus(Enum):
    PENDING = "pending"
    RUNNING = "running"
    SUCCESS = "success"
    FAILED = "failed"
    SKIPPED = "skipped"

class StepHandler(ABC):
    """Base class for step handlers."""

    @abstractmethod
    def execute(self, context: StepContext) -> StepResult:
        """Execute the step."""
        pass

    @abstractmethod
    def validate_config(self, config: Dict[str, Any]) -> ValidationResult:
        """Validate step configuration against schema."""
        pass

    @abstractmethod
    def get_config_schema(self) -> Dict[str, Any]:
        """Return JSON Schema for step configuration."""
        pass

    @abstractmethod
    def get_step_info(self) -> Dict[str, Any]:
        """Return step metadata (name, description, icon, etc.)."""
        pass

    def get_dependencies(self, config: Dict[str, Any]) -> List[str]:
        """Return required step outputs for this step (optional)."""
        return []

    def can_retry(self, error: str) -> bool:
        """Determine if step can be retried after error (optional)."""
        return False

    def get_retry_delay(self, attempt: int) -> float:
        """Get retry delay in seconds (exponential backoff)."""
        return min(2 ** attempt, 60)  # Max 60 seconds
```

**Example Implementation:**
```python
class VersionBumpHandler(StepHandler):
    """Handler for version bump step."""

    def execute(self, context: StepContext) -> StepResult:
        try:
            version_file = Path(context.workspace_root) / context.step_config["version_file"]
            current_version = self._read_version(version_file)
            increment_type = context.step_config.get("increment_type", "patch")
            new_version = self._increment_version(current_version, increment_type)
            self._write_version(version_file, new_version)

            return StepResult(
                step_id=context.step_id,
                status=StepStatus.SUCCESS,
                output={"new_version": new_version, "old_version": current_version},
                logs=[f"Incremented version: {current_version} → {new_version}"]
            )
        except Exception as e:
            return StepResult(
                step_id=context.step_id,
                status=StepStatus.FAILED,
                error=str(e),
                logs=[f"Error: {e}"]
            )

    def validate_config(self, config: Dict[str, Any]) -> ValidationResult:
        required_fields = ["version_file"]
        for field in required_fields:
            if field not in config:
                return ValidationResult(
                    valid=False,
                    errors=[f"Missing required field: {field}"]
                )
        return ValidationResult(valid=True)

    def get_config_schema(self) -> Dict[str, Any]:
        return {
            "type": "object",
            "properties": {
                "version_file": {
                    "type": "string",
                    "description": "Path to version file relative to workspace root"
                },
                "increment_type": {
                    "type": "string",
                    "enum": ["patch", "minor", "major", "feature", "feature_set"],
                    "default": "patch",
                    "description": "Version increment type"
                }
            },
            "required": ["version_file"]
        }

    def get_step_info(self) -> Dict[str, Any]:
        return {
            "name": "Bump Version",
            "description": "Increment version in version file",
            "icon": "🔢",
            "category": "version"
        }
```

---

#### 3. IntegrationAdapter Interface

**Purpose:** Provide integration adapters for external systems (Git, KB, Kanban, etc.)

**Interface:**
```python
class IntegrationAdapter(ABC):
    """Base class for integration adapters."""

    @abstractmethod
    def get_adapter_name(self) -> str:
        """Return adapter name (e.g., 'git', 'kb', 'kanban')."""
        pass

    @abstractmethod
    def initialize(self, config: Dict[str, Any]) -> None:
        """Initialize adapter with configuration."""
        pass

    @abstractmethod
    def validate_config(self, config: Dict[str, Any]) -> ValidationResult:
        """Validate adapter configuration."""
        pass

    @abstractmethod
    def get_config_schema(self) -> Dict[str, Any]:
        """Return JSON Schema for adapter configuration."""
        pass

    def cleanup(self) -> None:
        """Cleanup adapter resources (optional)."""
        pass
```

**Example Implementation:**
```python
class GitAdapter(IntegrationAdapter):
    """Adapter for Git integration."""

    def get_adapter_name(self) -> str:
        return "git"

    def initialize(self, config: Dict[str, Any]) -> None:
        self.workspace_root = Path(config.get("workspace_root", "."))
        self.remote = config.get("remote", "origin")

    def validate_config(self, config: Dict[str, Any]) -> ValidationResult:
        # Validate git configuration
        pass

    def get_config_schema(self) -> Dict[str, Any]:
        return {
            "type": "object",
            "properties": {
                "workspace_root": {
                    "type": "string",
                    "default": ".",
                    "description": "Workspace root directory"
                },
                "remote": {
                    "type": "string",
                    "default": "origin",
                    "description": "Git remote name"
                }
            }
        }

    def stage_files(self, paths: List[str]) -> subprocess.CompletedProcess:
        """Stage files via git add."""
        # Implementation

    def commit(self, message: str) -> subprocess.CompletedProcess:
        """Create git commit."""
        # Implementation

    def create_tag(self, tag_name: str, message: str, annotated: bool = True) -> subprocess.CompletedProcess:
        """Create git tag."""
        # Implementation

    def push(self, ref: str, push_tags: bool = False) -> subprocess.CompletedProcess:
        """Push to remote."""
        # Implementation
```

---

## Plugin Registry Design

### Plugin Registry Interface

**Purpose:** Manage plugin discovery, registration, and instantiation

**Interface:**
```python
class PluginRegistry:
    """Registry for managing VWMP plugins."""

    def __init__(self, plugin_directories: Optional[List[Path]] = None):
        self.workflow_types: Dict[str, WorkflowTypePlugin] = {}
        self.step_handlers: Dict[str, StepHandler] = {}  # handler_path -> handler
        self.integration_adapters: Dict[str, IntegrationAdapter] = {}
        self.plugin_directories = plugin_directories or []

    def register_workflow_type(self, plugin: WorkflowTypePlugin) -> None:
        """Register a workflow type plugin."""
        metadata = plugin.get_metadata()
        self.workflow_types[metadata.name] = plugin

    def register_step_handler(self, handler_path: str, handler: StepHandler) -> None:
        """Register a step handler.

        Args:
            handler_path: Handler path (e.g., 'release.version_bump', 'git.commit')
            handler: Step handler instance
        """
        self.step_handlers[handler_path] = handler

    def register_integration_adapter(self, adapter: IntegrationAdapter) -> None:
        """Register an integration adapter."""
        name = adapter.get_adapter_name()
        self.integration_adapters[name] = adapter

    def get_workflow_type(self, type_name: str) -> Optional[WorkflowTypePlugin]:
        """Get workflow type plugin by name."""
        return self.workflow_types.get(type_name)

    def get_handler(self, handler_path: str) -> Optional[StepHandler]:
        """Get step handler by path."""
        return self.step_handlers.get(handler_path)

    def get_adapter(self, adapter_name: str) -> Optional[IntegrationAdapter]:
        """Get integration adapter by name."""
        return self.integration_adapters.get(adapter_name)

    def discover_plugins(self) -> None:
        """Discover and load plugins from plugin directories."""
        for plugin_dir in self.plugin_directories:
            self._discover_plugins_in_directory(plugin_dir)

    def list_workflow_types(self) -> List[WorkflowTypeMetadata]:
        """List all registered workflow types."""
        return [plugin.get_metadata() for plugin in self.workflow_types.values()]

    def list_step_handlers(self) -> List[Dict[str, Any]]:
        """List all registered step handlers."""
        return [handler.get_step_info() for handler in self.step_handlers.values()]
```

---

## Plugin Discovery Mechanisms

### Option 1: File-Based Discovery (Recommended)

**Pattern:** Scan plugin directories for plugin modules

**Structure:**
```
plugins/
  release/
    __init__.py
    workflow_type.py      # ReleaseWorkflowTypePlugin
    handlers/
      __init__.py
      version_bump.py     # VersionBumpHandler
      changelog_create.py # ChangelogCreateHandler
      changelog_update.py # ChangelogUpdateHandler
      readme_update.py    # ReadmeUpdateHandler
  git/
    __init__.py
    adapter.py            # GitAdapter
    handlers/
      __init__.py
      stage_all.py        # GitStageHandler
      commit.py           # GitCommitHandler
      create_tag.py       # GitTagHandler
      push.py             # GitPushHandler
  project/
    __init__.py
    adapter.py            # Project-specificAdapter (if needed)
    handlers/
      __init__.py
      kanban_update.py    # KanbanUpdateHandler
      run_validators.py   # ValidatorExecutionHandler
```

**Discovery Process:**
1. Scan `plugins/` directory for plugin packages
2. Load `__init__.py` from each plugin package
3. Look for registration functions or entry points
4. Instantiate and register plugins

**Example `plugins/release/__init__.py`:**
```python
from .workflow_type import ReleaseWorkflowTypePlugin
from .handlers.version_bump import VersionBumpHandler
from .handlers.changelog_create import ChangelogCreateHandler
# ... more handlers

def register_plugin(registry: PluginRegistry) -> None:
    """Register release workflow plugin."""
    # Register workflow type
    registry.register_workflow_type(ReleaseWorkflowTypePlugin())

    # Register step handlers
    registry.register_step_handler("release.version_bump", VersionBumpHandler())
    registry.register_step_handler("release.changelog_create", ChangelogCreateHandler())
    # ... register more handlers
```

---

### Option 2: Entry Point Discovery (Future)

**Pattern:** Use Python entry points (setuptools/pkg_resources)

**setup.py / pyproject.toml:**
```python
setup(
    name="vwmp-release-plugin",
    entry_points={
        "vwmp.workflow_types": [
            "release = release.workflow_type:ReleaseWorkflowTypePlugin",
        ],
        "vwmp.step_handlers": [
            "release.version_bump = release.handlers.version_bump:VersionBumpHandler",
            "release.changelog_create = release.handlers.changelog_create:ChangelogCreateHandler",
        ],
    }
)
```

**Discovery Process:**
1. Use `pkg_resources.iter_entry_points()` or `importlib.metadata.entry_points()`
2. Load entry points and instantiate plugins
3. Register with plugin registry

**Advantages:**
- Standard Python plugin mechanism
- Works well for installable plugins
- Better for open-source distribution

**Disadvantages:**
- Requires package installation
- More complex for development

**Recommendation:** Start with file-based discovery, add entry point support later for open-source distribution.

---

## Configuration-Driven Design

### Plugin Configuration Schema

**Purpose:** Enable plugin configuration without hardcoded paths or assumptions

**Global Plugin Configuration:**
```yaml
# vwmp_config.yaml (optional, can use environment variables)
plugins:
  enabled: true
  discovery:
    method: "file_based"  # or "entry_points"
    directories:
      - "plugins/"
      - "/usr/local/lib/vwmp/plugins"
  auto_load: true

workflow_types:
  release:
    enabled: true
    config:
      version_file: "VERSION"
      changelog_dir: "KB/Changelog_and_Release_Notes/Changelog_Archive"
      main_changelog: "CHANGELOG.md"

step_handlers:
  release.version_bump:
    enabled: true
    retry:
      enabled: true
      max_attempts: 3
      backoff: "exponential"

integration_adapters:
  git:
    enabled: true
    config:
      workspace_root: "."  # Can be overridden per workflow
      remote: "origin"
```

### Configuration Resolution

**Hierarchy (highest to lowest priority):**
1. **Runtime Parameters** - User-provided parameters (e.g., workflow execution)
2. **Workflow-Level Config** - Workflow definition configuration
3. **Plugin-Level Config** - Plugin-specific configuration
4. **Global Config** - Global VWMP configuration
5. **Plugin Defaults** - Plugin-provided defaults
6. **Environment Variables** - System environment variables

**Variable Substitution:**
- `${config.version_file}` → Resolved from workflow config
- `${parameters.summary}` → Resolved from user input
- `${step-outputs.step-1.new_version}` → Resolved from step result
- `${env.WORKSPACE_ROOT}` → Environment variable
- `${adapter.git.remote}` → Integration adapter config

---

## Separation of Concerns

### Core Engine (Framework-Agnostic)

**Location:** `src/vwmp/core/` (or standalone package)

**Responsibilities:**
- Workflow parsing and validation
- Step orchestration
- Plugin system management
- Configuration resolution
- Execution engine

**Dependencies:**
- Python standard library
- `jsonschema` for validation
- `typing` for type hints
- No FastAPI, no Project-specific-specific code

### Project-specific Adapters (Project-specific-Specific)

**Location:** `src/vwmp/plugins/project/`

**Responsibilities:**
- Project-specific-specific integrations (KB, Kanban)
- Project-specific-specific step handlers
- Project-specific-specific workflow types (if any)

**Dependencies:**
- Core engine (framework-agnostic)
- Project-specific-specific modules (KB structure, Kanban format)
- Can import Project-specific code but core engine cannot

### Generic Plugins (Reusable)

**Location:** `src/vwmp/plugins/` or standalone packages

**Responsibilities:**
- Generic integrations (Git, File System, HTTP/API)
- Generic step handlers
- Reusable workflow types

**Dependencies:**
- Core engine only
- No Project-specific-specific code

---

## Plugin Isolation Strategy

### Error Isolation

**Plugin Errors Don't Cascade:**
- Step handler failures caught and logged
- Failed steps don't crash execution engine
- Optional steps can fail without stopping workflow
- Required steps cause workflow failure but engine continues

**Implementation:**
```python
def execute_step(self, step: StepDefinition, context: ExecutionContext) -> StepResult:
    try:
        handler = self.registry.get_handler(step.handler)
        if not handler:
            return StepResult(
                step_id=step.id,
                status=StepStatus.FAILED,
                error=f"Handler not found: {step.handler}"
            )

        result = handler.execute(context)
        return result
    except Exception as e:
        logger.error(f"Step execution error: {e}", exc_info=True)
        return StepResult(
            step_id=step.id,
            status=StepStatus.FAILED,
            error=str(e)
        )
```

### Plugin Sandboxing (Future)

**Process Isolation:**
- Run plugins in separate processes for security-critical operations
- Use subprocess or multiprocessing for plugin execution
- Isolated file system access
- Restricted network access

**Thread Isolation (Initial):**
- Run plugins in separate threads
- Shared memory (faster but less secure)
- Sufficient for trusted plugins initially

---

## Open-Source Packaging Strategy

### Package Structure for Independence

**Standalone VWMP Package:**
```
vwmp/
  __init__.py
  core/
    __init__.py
    engine.py          # Execution engine
    parser.py          # Workflow parser
    validator.py       # Workflow validator
    orchestrator.py    # Step orchestrator
    registry.py        # Plugin registry
    config.py          # Configuration system
  plugins/
    __init__.py
    git/               # Git plugin (generic)
    filesystem/        # File system plugin (generic)
    http/              # HTTP/API plugin (generic)
  interfaces/
    __init__.py
    workflow_type.py   # WorkflowTypePlugin interface
    step_handler.py    # StepHandler interface
    adapter.py         # IntegrationAdapter interface
```

**Project-specific Plugin Package (Separate):**
```
vwmp-project/
  __init__.py
  plugins/
    project/       # Project-specific-specific plugin
      __init__.py
      handlers/
        kanban_update.py
        run_validators.py
      adapter.py       # KB/Kanban adapter
```

**Benefits:**
- Core VWMP can be open-sourced independently
- Project-specific-specific code remains proprietary
- Users can install `vwmp` and create their own plugins
- Clear separation of concerns

---

## Plugin Development Guide

### Creating a New Workflow Type Plugin

**Steps:**
1. Create plugin directory: `plugins/<workflow_type>/`
2. Implement `WorkflowTypePlugin` interface
3. Define workflow templates (optional)
4. Register plugin in `__init__.py`
5. Test plugin registration and validation

**Example Structure:**
```
plugins/ci_cd/
  __init__.py
  workflow_type.py
  handlers/
    __init__.py
    build.py
    test.py
    deploy.py
```

### Creating a New Step Handler

**Steps:**
1. Create step handler class implementing `StepHandler`
2. Implement `execute()`, `validate_config()`, `get_config_schema()`, `get_step_info()`
3. Register handler in plugin `__init__.py`
4. Test handler execution

**Example:**
```python
class CustomStepHandler(StepHandler):
    def execute(self, context: StepContext) -> StepResult:
        # Implementation
        pass
```

### Creating a New Integration Adapter

**Steps:**
1. Create adapter class implementing `IntegrationAdapter`
2. Implement adapter methods
3. Register adapter in plugin `__init__.py`
4. Use adapter in step handlers

---

## Extension Points

### 1. Workflow Type Extensions
- Add new workflow types via plugins
- Provide workflow templates
- Define workflow-specific validation rules
- Customize workflow configuration schema

### 2. Step Handler Extensions
- Add new step handlers via plugins
- Extend existing handlers with new functionality
- Create composite handlers (handlers that call other handlers)
- Customize step execution logic

### 3. Integration Adapter Extensions
- Add new integration adapters (Slack, Email, etc.)
- Extend existing adapters with new methods
- Create adapter factories for dynamic adapter creation
- Customize adapter configuration

### 4. Configuration Extensions
- Add custom configuration schemas
- Define configuration validators
- Create configuration templates
- Support configuration inheritance

### 5. Validation Extensions
- Add custom workflow validators
- Define workflow-level validation rules
- Create step-level validation rules
- Customize validation error messages

---

## Architectural Independence Requirements

### Core Engine Independence

**Must NOT Depend On:**
- FastAPI or any web framework
- Project-specific-specific modules
- Project-specific paths or assumptions
- External services (must be configurable)

**Must Support:**
- Configuration-driven behavior
- Plugin-based extensibility
- Framework-agnostic execution
- Standalone deployment

### Project-specific Isolation

**Project-specific-Specific Code:**
- Isolated in `vwmp/plugins/project/`
- Can import Project-specific modules
- Cannot be imported by core engine
- Clear boundaries via adapter interfaces

**Adapter Pattern:**
- Core engine uses abstract adapter interfaces
- Project-specific adapters implement interfaces
- Core engine doesn't know about Project-specific specifics

---

## Plugin Configuration Examples

### Release Workflow Plugin Configuration

```yaml
# plugins/release/config.yaml
workflow_type:
  name: "release"
  templates:
    - name: "Standard Release"
      description: "Full release workflow with all steps"
      template:
        steps:
          - type: "version_bump"
            handler: "release.version_bump"
            required: true
          - type: "changelog_create"
            handler: "release.changelog_create"
            required: true
          # ... more steps

step_handlers:
  version_bump:
    config_schema:
      version_file:
        type: "string"
        default: "VERSION"
        description: "Path to version file"
    retry:
      enabled: false

  changelog_create:
    config_schema:
      changelog_dir:
        type: "string"
        default: "KB/Changelog_and_Release_Notes/Changelog_Archive"
      format:
        type: "string"
        enum: ["full_timestamp"]
        default: "full_timestamp"
    retry:
      enabled: false
```

### Git Plugin Configuration

```yaml
# plugins/git/config.yaml
adapter:
  name: "git"
  config_schema:
    workspace_root:
      type: "string"
      default: "."
    remote:
      type: "string"
      default: "origin"
    branch:
      type: "string"
      default: null  # Auto-detect from git

step_handlers:
  stage_all:
    config_schema:
      paths:
        type: "array"
        items:
          type: "string"
        default: ["*"]
    retry:
      enabled: false

  commit:
    config_schema:
      message_template:
        type: "string"
        default: "{version} - {summary}"
    retry:
      enabled: false

  create_tag:
    config_schema:
      tag_template:
        type: "string"
        default: "v{version}"
      annotated:
        type: "boolean"
        default: true
    retry:
      enabled: false

  push:
    config_schema:
      push_tags:
        type: "boolean"
        default: true
      remote:
        type: "string"
        default: "${adapter.git.remote}"
    retry:
      enabled: true
      max_attempts: 3
      backoff: "exponential"
```

---

## Plugin Lifecycle

### 1. Plugin Discovery (Application Startup)

1. **Load Plugin Directories:** Scan configured plugin directories
2. **Discover Plugins:** Find plugin packages (directories with `__init__.py`)
3. **Load Plugin Modules:** Import plugin modules
4. **Register Plugins:** Call plugin registration functions
5. **Validate Plugins:** Validate plugin interfaces and configurations
6. **Initialize Plugins:** Call plugin `initialize()` methods

### 2. Plugin Execution (Runtime)

1. **Get Plugin:** Retrieve plugin from registry
2. **Validate Configuration:** Validate plugin configuration
3. **Initialize Plugin:** Initialize plugin with configuration (if not already)
4. **Execute Plugin:** Execute plugin functionality
5. **Handle Errors:** Catch and log errors, apply retry logic if configured
6. **Collect Results:** Collect plugin output for dependent steps

### 3. Plugin Cleanup (Application Shutdown)

1. **Cleanup Plugins:** Call plugin `cleanup()` methods
2. **Release Resources:** Release plugin resources
3. **Unregister Plugins:** Remove plugins from registry

---

## Error Handling & Recovery

### Plugin Error Types

1. **Plugin Not Found:** Handler or adapter not registered
2. **Configuration Errors:** Invalid plugin configuration
3. **Execution Errors:** Plugin execution failures
4. **Dependency Errors:** Missing dependencies or resources

### Error Handling Strategy

**Plugin Not Found:**
- Return error result immediately
- Log error for debugging
- Fail workflow if step is required

**Configuration Errors:**
- Validate during workflow validation (before execution)
- Return validation errors to user
- Prevent workflow execution if invalid

**Execution Errors:**
- Catch exceptions in plugin execution
- Log error with context
- Apply retry logic if configured
- Return error result to orchestrator

**Dependency Errors:**
- Check dependencies before execution
- Return error if dependencies missing
- Provide clear error messages

---

## Security Considerations

### Plugin Security Model

**Trusted Plugins (Initial):**
- Plugins loaded from configured directories
- All plugins trusted (same security context)
- Thread-based execution (shared memory)

**Untrusted Plugins (Future):**
- Plugin signature validation
- Process isolation for untrusted plugins
- Restricted file system access
- Restricted network access
- Sandboxed execution environment

### Security Best Practices

1. **Input Validation:** Validate all plugin inputs
2. **Output Sanitization:** Sanitize plugin outputs
3. **Resource Limits:** Limit plugin resource usage (time, memory)
4. **Access Control:** Restrict plugin file system access
5. **Audit Logging:** Log all plugin operations

---

## Testing Strategy

### Plugin Testing

**Unit Tests:**
- Test plugin interfaces
- Test plugin configuration validation
- Test plugin execution logic
- Mock dependencies

**Integration Tests:**
- Test plugin registration and discovery
- Test plugin execution in workflow context
- Test error handling and recovery
- Test plugin isolation

**Acceptance Tests:**
- Test complete workflows with plugins
- Test plugin interactions
- Test configuration resolution
- Test real-world scenarios

---

## Migration Path to Open-Source

### Phase 1: Integrated Plugin System
- Plugins in `src/vwmp/plugins/`
- Project-specific-specific plugins mixed with generic plugins
- File-based discovery
- Core engine as FastAPI module

### Phase 2: Plugin Separation
- Move core engine to `src/vwmp/core/`
- Move generic plugins to `src/vwmp/plugins/`
- Move Project-specific-specific plugins to `src/vwmp/plugins/project/`
- Add adapter interfaces for separation

### Phase 3: Standalone Package
- Extract core engine to standalone `vwmp` package
- Extract generic plugins to `vwmp` package
- Create separate `vwmp-project` package
- Use entry points for plugin discovery

### Phase 4: Open-Source Release
- Publish `vwmp` as open-source package
- Keep `vwmp-project` proprietary
- Provide plugin development documentation
- Create plugin ecosystem

---

## Open Questions

1. **Plugin Discovery:** File-based vs entry points initially?
   - **Recommendation:** Start with file-based, add entry points later

2. **Plugin Isolation:** Thread-based vs process-based initially?
   - **Recommendation:** Thread-based initially, add process isolation for security-critical plugins later

3. **Plugin Versioning:** How to handle plugin version conflicts?
   - **Recommendation:** Plugin versioning scheme, compatibility checks

4. **Plugin Updates:** How to handle plugin updates without breaking workflows?
   - **Recommendation:** Workflow versioning, plugin compatibility matrix

5. **Plugin Dependencies:** How to handle plugin dependencies?
   - **Recommendation:** Plugin dependency declaration, dependency resolution

---

## Related Documents

- **Main Architecture:** `KB/Architecture/Standards_and_ADRs/E01-vwmp-platform-architecture.md`
- **SET2 Plugin Architecture:** `KB/Architecture/Standards_and_ADRs/adr-0005-set2-autonomous-architecture.md`
- **Container View:** `KB/Architecture/container/overview/epic01-vwmp-container.md`
- **Component View:** `KB/Architecture/component/overview/epic01-vwmp-components.md`
- **Epic 01 Story:** `KB/PM_and_Portfolio/stories/overview/Epic 01/Story-1-Visual-Workflow-Management-Platform.md`

---

**Document Status:** Draft - Ready for Review
**Last Updated:** 2025-11-20
**Next Review:** After stakeholder feedback
