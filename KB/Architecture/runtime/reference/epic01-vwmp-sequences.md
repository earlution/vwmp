# Epic 01 – VWMP Runtime Sequence Diagrams

**Status:** Draft
**Epic:** Epic 01 - Visual Workflow Management Platform (VWMP)
**Last Updated:** 2025-11-20
**Related Artifacts:**
- `KB/Architecture/component/overview/epic01-vwmp-components.md`
- `KB/Architecture/Standards_and_ADRs/E01-vwmp-platform-architecture.md`
- `KB/Architecture/Diagrams/Runtime/epic01-vwmp-sequences.md`

---

## 1. Workflow Creation Sequence

### Creating a New Workflow

```mermaid
sequenceDiagram
    autonumber
    participant User as User/Developer
    participant Designer as Visual Designer
    participant API as Workflow Management API
    participant Parser as Workflow Parser
    participant Validator as Workflow Validator
    participant Storage as Workflow Storage
    participant Registry as Plugin Registry

    User->>Designer: Create new workflow
    User->>Designer: Drag steps from palette
    Designer->>Registry: Get available step types
    Registry-->>Designer: Step type list (from plugins)
    User->>Designer: Configure steps (parameters)
    User->>Designer: Connect steps (dependencies)
    User->>Designer: Preview workflow
    Designer->>Designer: Generate workflow definition (YAML/JSON)
    User->>Designer: Save workflow
    Designer->>API: POST /api/workflows (workflow definition)
    API->>Parser: Parse workflow definition
    Parser-->>API: Parsed workflow object
    API->>Validator: Validate workflow
    Validator->>Registry: Validate handler references
    Registry-->>Validator: Handler validation result
    Validator->>Validator: Check dependencies (no cycles)
    Validator->>Validator: Validate parameter schemas
    Validator-->>API: Validation result
    alt Validation Success
        API->>Storage: Save workflow definition
        Storage-->>API: Workflow ID
        API-->>Designer: 201 Created (workflow ID)
        Designer-->>User: Workflow saved successfully
    else Validation Failed
        API-->>Designer: 400 Bad Request (validation errors)
        Designer-->>User: Display validation errors
    end
```

---

## 2. Workflow Execution Sequence (Release Workflow)

### Executing a Release Workflow

```mermaid
sequenceDiagram
    autonumber
    participant User as Release Manager
    participant Designer as Visual Designer
    participant API as Workflow Management API
    participant Engine as Execution Engine
    participant Orchestrator as Step Orchestrator
    participant Storage as Workflow Storage
    participant Registry as Plugin Registry
    participant Executor as Step Executor
    participant Handler as Step Handler (Release Plugin)
    participant Git as Git Repository
    participant KB as Knowledge Base
    participant Validator as Validator Scripts

    User->>Designer: Select release workflow
    User->>Designer: Enter parameters (summary, type, etc.)
    User->>Designer: Review workflow preview
    User->>Designer: Execute workflow
    Designer->>API: POST /api/workflows/{id}/execute (parameters)
    API->>API: Validate parameters against schema
    API->>Storage: Load workflow definition
    Storage-->>API: Workflow definition
    API->>Engine: Execute workflow (workflow, parameters)
    Engine->>Orchestrator: Resolve execution order
    Orchestrator->>Orchestrator: Topological sort (dependency resolution)
    Orchestrator-->>Engine: Execution plan (step order)

    Note over Engine,Handler: Execute Step 1: Bump Version
    Engine->>Executor: Execute step (step-1, context)
    Executor->>Registry: Get handler (release.version_bump)
    Registry-->>Executor: VersionBumpHandler instance
    Executor->>Handler: Execute (step config, context)
    Handler->>Handler: Read current version from file
    Handler->>Handler: Increment version
    Handler->>Handler: Update version file
    Handler-->>Executor: StepResult (new_version, status: SUCCESS)
    Executor-->>Engine: Step result (step-1)
    Engine->>API: WebSocket: Step status (step-1: SUCCESS)
    API-->>Designer: Real-time update (WebSocket)
    Designer-->>User: Show step 1 completed

    Note over Engine,Handler: Execute Step 2: Create Detailed Changelog
    Engine->>Executor: Execute step (step-2, context with step-1 output)
    Executor->>Handler: Execute (changelog_create, context)
    Handler->>Handler: Get timestamp, epic, story from branch
    Handler->>KB: Write CHANGELOG_v{version}.md
    KB-->>Handler: File created
    Handler-->>Executor: StepResult (status: SUCCESS)
    Executor-->>Engine: Step result (step-2)
    Engine->>API: WebSocket: Step status (step-2: SUCCESS)
    API-->>Designer: Real-time update
    Designer-->>User: Show step 2 completed

    Note over Engine,Validator: Execute Step 7: Run Validators
    Engine->>Executor: Execute step (step-7, context)
    Executor->>Handler: Execute (run_validators, context)
    Handler->>Validator: Execute validate_branch_context.py
    Validator-->>Handler: Validation result
    Handler->>Validator: Execute validate_changelog_format.py
    Validator-->>Handler: Validation result
    Handler-->>Executor: StepResult (status: SUCCESS)
    Executor-->>Engine: Step result (step-7)

    Note over Engine,Git: Execute Step 8: Commit
    Engine->>Executor: Execute step (step-8, context)
    Executor->>Handler: Execute (git.commit, context)
    Handler->>Git: git commit -m "{version} - {summary}"
    Git-->>Handler: Commit created
    Handler-->>Executor: StepResult (status: SUCCESS)
    Executor-->>Engine: Step result (step-8)

    Note over Engine: Parallel Execution (if no dependencies)
    par Execute Step 9: Create Tag (optional)
        Engine->>Executor: Execute step (step-9, context)
        Executor->>Handler: Execute (git.create_tag, context)
        Handler->>Git: git tag -a v{version}
        Git-->>Handler: Tag created
        Handler-->>Executor: StepResult (status: SUCCESS)
        Executor-->>Engine: Step result (step-9)
    and Execute Step 10: Push (optional)
        Engine->>Executor: Execute step (step-10, context)
        Executor->>Handler: Execute (git.push, context)
        Handler->>Git: git push origin {branch}
        Git-->>Handler: Push successful
        Handler->>Git: git push origin v{version}
        Git-->>Handler: Tag pushed
        Handler-->>Executor: StepResult (status: SUCCESS)
        Executor-->>Engine: Step result (step-10)
    end

    Engine->>Storage: Store execution history
    Engine->>API: WebSocket: Workflow completed (SUCCESS)
    API-->>Designer: Final status update
    Designer-->>User: Workflow execution completed successfully
    Engine-->>API: WorkflowResult (all steps successful)
    API-->>Designer: Execution summary
    Designer-->>User: Display execution summary and history
```

---

## 3. Error Handling Sequence

### Handling Step Failure with Retry

```mermaid
sequenceDiagram
    autonumber
    participant Engine as Execution Engine
    participant Orchestrator as Step Orchestrator
    participant Executor as Step Executor
    participant Handler as Step Handler
    participant System as External System (Git/KB/etc.)
    participant API as Workflow Management API
    participant Designer as Visual Designer

    Engine->>Orchestrator: Execute workflow
    Orchestrator->>Executor: Execute step (step-N)
    Executor->>Handler: Execute (step config, context)
    Handler->>System: Execute operation (e.g., git push)
    System-->>Handler: Error (e.g., authentication failed)
    Handler-->>Executor: StepResult (status: FAILED, error: "...")
    Executor->>Executor: Check retry configuration
    alt Retry Configured and Retries Remaining
        Note over Executor: Wait for retry delay
        Executor->>Handler: Retry execute (attempt 2)
        Handler->>System: Retry operation
        alt Retry Success
            System-->>Handler: Success
            Handler-->>Executor: StepResult (status: SUCCESS)
            Executor-->>Engine: Step result (SUCCESS after retry)
        else Retry Failed
            Handler-->>Executor: StepResult (status: FAILED)
            Executor-->>Engine: Step result (FAILED after retries)
        end
    else No Retry or Retries Exhausted
        Executor-->>Engine: Step result (FAILED)
    end

    Engine->>Engine: Check step requirements
    alt Step is Required
        Engine->>Orchestrator: Abort workflow (required step failed)
        Orchestrator->>Engine: WorkflowResult (status: FAILED)
        Engine->>API: WebSocket: Workflow failed (required step failed)
        API-->>Designer: Error notification
        Designer-->>User: Display error: Required step failed
    else Step is Optional
        Engine->>Orchestrator: Continue workflow (skip optional step)
        Orchestrator->>Engine: Continue with next steps
        Engine->>API: WebSocket: Step skipped (optional step failed)
        API-->>Designer: Warning notification
        Designer-->>User: Display warning: Optional step failed, continuing
    end
```

---

## 4. Real-Time Monitoring Sequence

### WebSocket Real-Time Updates

```mermaid
sequenceDiagram
    autonumber
    participant User as User
    participant Designer as Visual Designer
    participant WebSocket as WebSocket Handler
    participant Engine as Execution Engine
    participant Executor as Step Executor

    User->>Designer: Execute workflow
    Designer->>WebSocket: Establish WebSocket connection
    WebSocket-->>Designer: Connection established

    Note over Designer,Engine: Workflow Execution Starts
    Designer->>Engine: Execute workflow (via API)
    Engine->>WebSocket: Workflow started (workflow_id, status: RUNNING)
    WebSocket-->>Designer: Event: workflow_status_changed
    Designer-->>User: Show workflow started

    Note over Engine,Executor: Step Execution
    Engine->>Executor: Execute step (step-1)
    Executor->>Engine: Step started (step-1, status: RUNNING)
    Engine->>WebSocket: Step status (step-1, status: RUNNING)
    WebSocket-->>Designer: Event: step_status_changed
    Designer-->>User: Update step status (running)

    Executor->>Executor: Execute step logic
    Executor->>Engine: Step progress (step-1, progress: 50%, log: "...")
    Engine->>WebSocket: Step progress update
    WebSocket-->>Designer: Event: step_progress_updated
    Designer-->>User: Update progress bar, show logs

    Executor->>Engine: Step completed (step-1, status: SUCCESS, output: {...})
    Engine->>WebSocket: Step status (step-1, status: SUCCESS)
    WebSocket-->>Designer: Event: step_status_changed
    Designer-->>User: Show step completed (green checkmark)

    Note over Engine: Workflow Completion
    Engine->>Engine: All steps completed
    Engine->>WebSocket: Workflow completed (status: SUCCESS, summary)
    WebSocket-->>Designer: Event: workflow_completed
    Designer-->>User: Show workflow completed, display summary
    Designer->>WebSocket: Close connection
    WebSocket-->>Designer: Connection closed
```

---

## 5. Plugin Discovery and Registration Sequence

### Plugin Loading at Startup

```mermaid
sequenceDiagram
    autonumber
    participant App as Application Startup
    participant Registry as Plugin Registry
    participant FS as File System
    participant Plugin1 as Release Plugin
    participant Plugin2 as Git Plugin
    participant Plugin3 as Project Integration Plugin

    App->>Registry: Initialize plugin registry
    Registry->>FS: Scan plugins/ directory
    FS-->>Registry: Plugin directories list

    Note over Registry,Plugin1: Load Release Plugin
    Registry->>FS: Discover plugins/release/
    FS-->>Registry: Plugin files
    Registry->>Plugin1: Import release.plugin
    Plugin1->>Plugin1: Register workflow type (type: "release")
    Plugin1->>Registry: Register WorkflowTypePlugin
    Plugin1->>Registry: Register StepHandler (release.version_bump)
    Plugin1->>Registry: Register StepHandler (release.changelog_create)
    Plugin1->>Registry: Register StepHandler (release.changelog_update)
    Plugin1->>Registry: Register StepHandler (release.readme_update)
    Registry-->>Plugin1: Registration confirmed

    Note over Registry,Plugin2: Load Git Plugin
    Registry->>Plugin2: Import git.plugin
    Plugin2->>Registry: Register StepHandler (git.stage_all)
    Plugin2->>Registry: Register StepHandler (git.commit)
    Plugin2->>Registry: Register StepHandler (git.create_tag)
    Plugin2->>Registry: Register StepHandler (git.push)

    Note over Registry,Plugin3: Load Project Integration Plugin
    Registry->>Plugin3: Import vwmp.plugin
    Plugin3->>Registry: Register StepHandler (vwmp.kanban_update)
    Plugin3->>Registry: Register StepHandler (vwmp.run_validators)

    Registry->>Registry: Build handler registry map
    Registry->>App: Plugin registry initialized
    App->>App: Application ready
```

---

## 6. Workflow Definition Storage Sequence

### Loading and Validating Workflow

```mermaid
sequenceDiagram
    autonumber
    participant User as User
    participant Designer as Visual Designer
    participant API as Workflow Management API
    participant Parser as Workflow Parser
    participant Validator as Workflow Validator
    participant Storage as Workflow Storage
    participant Registry as Plugin Registry
    participant FS as File System

    User->>Designer: Edit existing workflow
    Designer->>API: GET /api/workflows/{id}
    API->>Storage: Load workflow definition
    Storage->>FS: Read workflow YAML/JSON file
    FS-->>Storage: Workflow file content
    Storage->>Parser: Parse workflow definition
    Parser->>Parser: Parse YAML/JSON
    Parser-->>Storage: Parsed WorkflowDefinition object
    Storage-->>API: WorkflowDefinition
    API->>Validator: Validate workflow
    Validator->>Validator: Validate schema structure
    Validator->>Registry: Validate handler references
    Registry-->>Validator: Handler availability check
    Validator->>Validator: Check dependencies (no cycles)
    Validator->>Validator: Validate parameter schemas
    Validator-->>API: ValidationResult (valid)
    API-->>Designer: Workflow definition (YAML/JSON)
    Designer-->>User: Load workflow in designer

    User->>Designer: Make changes
    User->>Designer: Save workflow
    Designer->>API: PUT /api/workflows/{id} (updated definition)
    API->>Parser: Parse updated definition
    API->>Validator: Validate updated workflow
    Validator-->>API: ValidationResult
    alt Validation Success
        API->>Storage: Save workflow definition
        Storage->>FS: Write workflow file (with versioning)
        FS-->>Storage: File saved
        Storage-->>API: Workflow saved
        API-->>Designer: 200 OK
        Designer-->>User: Workflow saved successfully
    else Validation Failed
        API-->>Designer: 400 Bad Request (validation errors)
        Designer-->>User: Display validation errors
    end
```

---

## 7. Related Documents

- **Component View:** `KB/Architecture/component/overview/epic01-vwmp-components.md`
- **Container View:** `KB/Architecture/container/overview/epic01-vwmp-container.md`
- **Integration Architecture:** `KB/Architecture/Integration_Architecture/vwmp/vwmp-integration-architecture.md`
- **Main Architecture:** `KB/Architecture/Standards_and_ADRs/E01-vwmp-platform-architecture.md`

---

_Next steps_: Create PlantUML versions of these diagrams for KB Diagrams directory
