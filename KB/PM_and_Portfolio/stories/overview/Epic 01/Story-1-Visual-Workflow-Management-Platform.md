# Epic 01, Story 1: Visual Workflow Management Platform (VWMP)

**Status:** COMPLETE
**Priority:** MEDIUM
**Estimated Effort:** 240-380 hours (full platform development)
**Actual Effort:** - (as implemented)
**Started:** 2025-11-20
**Completed:** 2025-11-20
**Version:** v0.21.0.11
**Code:** E01S1

**Note:** Initial implementation focuses on release workflows as the first use case, but architecture is designed for general-purpose workflow management.

**Moved From:** Originally E20:S09, spun off to Epic 01 on 2025-11-25 due to scope (240-380 hours) and strategic value (open-source potential). Epic 20 focuses on maintenance work; this is feature development.

**Effort Breakdown (from Story 10 analysis):**
- Architecture design: 20-40 hours
- Core workflow engine: 60-100 hours
- Visual designer: 80-120 hours
- KB/Kanban integrations: 40-60 hours (can leverage existing scripts)
- Testing: 40-60 hours
- **Total: 240-380 hours**

## Task Index
- E01:S01:T001 – Design general-purpose workflow management platform architecture
- E01:S01:T002 – Design plugin-based architecture for independence (open-source potential)
- E01:S01:T003 – Design visual workflow designer UI/UX
- E01:S01:T004 – Implement workflow definition schema and configuration system
- E01:S01:T005 – Implement visual workflow designer (drag-and-drop, step configuration)
- E01:S01:T006 – Implement workflow execution engine
- E01:S01:T007 – Implement workflow monitoring and progress tracking
- E01:S01:T008 – Implement workflow management (versioning, updates, persistence)
- E01:S01:T009 – Implement release workflow adapter (first use case)
- E01:S01:T010 – Integrate platform with existing release workflow script

## Task Checklist
- [x] **E01:S01:T001 – Design general-purpose workflow management platform architecture** (v0.21.0.2)
- [x] **E01:S01:T002 – Design plugin-based architecture for independence (open-source potential)** (v0.21.0.3)
- [x] **E01:S01:T003 – Design visual workflow designer UI/UX** (v0.21.0.4)
- [x] **E01:S01:T004 – Implement workflow definition schema and configuration system** (v0.21.0.5)
- [x] **E01:S01:T005 – Implement visual workflow designer backend API** (v0.21.0.11)
- [x] **E01:S01:T006 – Implement workflow execution engine** (v0.21.0.6)
- [x] **E01:S01:T007 – Implement workflow monitoring and progress tracking** (v0.21.0.7)
- [x] **E01:S01:T008 – Implement workflow management (versioning, updates, persistence)** (v0.21.0.8)
- [x] **E01:S01:T009 – Implement release workflow adapter (first use case)** (v0.21.0.9)
- [x] **E01:S01:T010 – Integrate platform with existing release workflow script** (v0.21.0.10)

## Overview
Create a **fully configurable, visual workflow management platform** for defining, maintaining, and operating workflows in software projects. The platform provides a visual, intuitive interface for workflow design, configuration, execution, and monitoring - making complex workflows accessible to all team members.

**Initial Use Case:** Release workflow automation (replacing/enhancing current CLI script)
**Long-term Vision:** General-purpose workflow management for any software project workflow (CI/CD, deployments, testing, documentation, etc.)

**Critical Requirements:**
- **Visual and intuitive** - NOT CLI-based, must be accessible to non-technical users
- **Fully configurable** - Users can define, modify, and maintain workflows visually
- **Workflow designer** - Visual editor for creating and editing workflow definitions
- **Workflow execution** - Execute workflows with visual monitoring and feedback
- **Workflow maintenance** - Version, update, and manage workflow definitions
- **General-purpose** - Not limited to release workflows, extensible to any workflow type

## Decision Rationale: Why Build From Scratch

After comprehensive evaluation (see [Story 10](../Epic%2020/Story-10-Evaluate-Release-Automation-Frameworks.md)), we committed to building our own general-purpose workflow management platform. This decision was informed by:

### Evaluation Summary
- **External Frameworks Evaluated:** GitHub Actions, CircleCI, Jenkins
- **Alternative Considered:** Custom Jenkins plugin with Blue Ocean
- **Research Documents:** See [Story 10 Recommendation Report](../Epic%2020/Story-10-Recommendation-Report.md) for complete analysis

### Key Findings

1. **Visual Dashboard Gap (Critical):**
   - None of the external frameworks provide visual, intuitive dashboards with toggles and step-by-step configuration
   - Jenkins Blue Ocean provides visual pipeline editor, but still requires significant custom development for our needs
   - This is a **hard requirement** that justifies custom development

2. **Development Effort Comparison:**
   - **From Scratch:** 240-380 hours (Python, can reuse existing code)
   - **Jenkins Plugin:** 260-420 hours (Java, new learning curve)
   - **External Frameworks:** Would still require 200-300+ hours of custom development

3. **Stack Alignment:**
   - Building in Python allows us to leverage existing scripts (`release_workflow.py`, `update_kanban_docs.py`)
   - Use our existing Python expertise
   - Maintain consistency with our codebase

4. **Requirement Coverage:**
   - Our current custom script meets **92% of requirements** with full support
   - External frameworks meet only **33-38%** of requirements
   - Bespoke requirements (KB structure, Kanban auto-update, dual changelog format) not well-served by generic frameworks

5. **General-Purpose Vision:**
   - We're building a platform for **any software project workflow**, not just CI/CD
   - Requires full control over architecture and design
   - Better suited as standalone platform than Jenkins plugin

6. **Open-Source Potential:**
   - Standalone platform easier to spin off as open-source project
   - Addresses clear market gap (visual workflow management for software projects)
   - No vendor lock-in

### Research References

All research that informed this decision:
- **Requirements Document:** [`Story-10-Release-Requirements.md`](../Epic%2020/Story-10-Release-Requirements.md)
- **Framework Research:** [`Story-10-GitHub-Actions-Research.md`](../Epic%2020/Story-10-GitHub-Actions-Research.md), [`Story-10-CircleCI-Research.md`](../Epic%2020/Story-10-CircleCI-Research.md), [`Story-10-Jenkins-Research.md`](../Epic%2020/Story-10-Jenkins-Research.md)
- **Comparison Matrix:** [`Story-10-Comparison-Matrix.md`](../Epic%2020/Story-10-Comparison-Matrix.md)
- **Final Recommendation:** [`Story-10-Recommendation-Report.md`](../Epic%2020/Story-10-Recommendation-Report.md) - Includes Jenkins plugin analysis

## Goals
- [ ] **Visual Workflow Designer** - Visual editor for defining and editing workflow definitions
- [ ] **Workflow Configuration** - Configure workflow steps, parameters, conditions, and dependencies visually
- [ ] **Workflow Execution Engine** - Execute workflows with visual monitoring, progress tracking, and error handling
- [ ] **Workflow Management** - Version, update, and maintain workflow definitions
- [ ] **General-Purpose Architecture** - Not limited to release workflows, extensible to any workflow type
- [ ] **Visual Interface** - Intuitive, accessible to all team members (NOT CLI-based)
- [ ] **Plugin System** - Extensible architecture for custom workflow types and integrations
- [ ] **Configuration-Driven** - Workflows defined via configuration (YAML/JSON), not hardcoded
- [ ] **Initial Implementation** - Release workflow as first use case, demonstrating general-purpose capabilities
- [ ] **Architectural Independence** - Designed for open-source spin-off as standalone workflow management platform

## Tasks

- [ ] **E01:S01:T001 – Design general-purpose workflow management platform architecture**
  - **Input:** Current release workflow script, general-purpose workflow requirements, architectural independence goals
  - **Deliverable:** Platform architecture document defining:
    - Core workflow engine (framework-agnostic)
    - Workflow definition schema (YAML/JSON)
    - Plugin system for workflow types and integrations
    - Execution engine architecture
    - Visual designer architecture
    - Configuration system
    - Separation of concerns (core vs adapters)
  - **Dependencies:** E20:S06:T005 (release workflow script must exist) - ✅ COMPLETE
  - **Blocker:** None - can design architecture independently
  - **Parallel Development Candidacy:** Safe - architecture design is independent
  - **Strategic Value:** Foundation for general-purpose workflow management platform

- [ ] **E01:S01:T002 – Design plugin-based architecture for independence (open-source potential)**
  - **Input:** Platform architecture (T001), architectural independence requirements
  - **Deliverable:** Plugin architecture document defining:
    - Plugin system for workflow types (Release, CI/CD, Deployment, etc.)
    - Abstract interfaces (WorkflowExecutor, StepHandler, IntegrationAdapter)
    - Configuration-driven design (no hardcoded project-specific paths)
    - Extensibility points and plugin development guide
    - Open-source packaging strategy
  - **Dependencies:** E01:S01:T001 (platform architecture must be defined)
  - **Blocker:** E01:S01:T001
  - **Parallel Development Candidacy:** Blocked until platform architecture is complete
  - **Strategic Value:** Enables future open-source spin-off as standalone workflow management platform

- [ ] **E01:S01:T003 – Design visual workflow designer UI/UX**
  - **Input:** Platform architecture (T001), workflow definition schema, UI/UX requirements
  - **Deliverable:** Visual designer UI/UX design:
    - Drag-and-drop workflow builder
    - Step configuration panels
    - Parameter input forms
    - Dependency visualization
    - Workflow preview
    - Technology stack decision (React/Vue/etc.)
  - **Dependencies:** E01:S01:T001 (platform architecture must be defined)
  - **Blocker:** E01:S01:T001
  - **Parallel Development Candidacy:** Safe - can design in parallel with T002

- [ ] **E01:S01:T004 – Implement workflow definition schema and configuration system**
  - **Input:** Platform architecture (T001), plugin architecture (T002)
  - **Deliverable:**
    - Workflow definition schema (YAML/JSON format)
    - Configuration parser and validator
    - Workflow registry system
    - Configuration persistence layer
  - **Dependencies:** E01:S01:T001, E01:S01:T002 (architecture must be defined)
  - **Blocker:** E01:S01:T001, E01:S01:T002
  - **Parallel Development Candidacy:** Blocked until architecture is complete

- [ ] **E01:S01:T005 – Implement visual workflow designer (drag-and-drop, step configuration)**
  - **Input:** UI/UX design (T003), workflow definition schema (T004)
  - **Deliverable:** Visual workflow designer:
    - Drag-and-drop interface for creating workflows
    - Step configuration panels with form controls
    - Parameter input and validation
    - Dependency management UI
    - Workflow preview
  - **Dependencies:** E01:S01:T003 (UI/UX design), E01:S01:T004 (schema)
  - **Blocker:** E01:S01:T003, E01:S01:T004
  - **Parallel Development Candidacy:** Blocked until design and schema are complete

- [ ] **E01:S01:T006 – Implement workflow execution engine**
  - **Input:** Platform architecture (T001), workflow definition schema (T004), plugin architecture (T002)
  - **Deliverable:** Workflow execution engine:
    - Workflow parser and validator
    - Step execution orchestrator
    - Dependency resolution
    - Error handling and recovery
    - Integration with plugin system
  - **Dependencies:** E01:S01:T001, E01:S01:T002, E01:S01:T004 (architecture and schema must be defined)
  - **Blocker:** E01:S01:T001, E01:S01:T002, E01:S01:T004
  - **Parallel Development Candidacy:** Blocked until core architecture is complete

- [ ] **E01:S01:T007 – Implement workflow monitoring and progress tracking**
  - **Input:** Execution engine (T006), UI/UX design (T003)
  - **Deliverable:** Workflow monitoring system:
    - Real-time progress tracking
    - Step status visualization
    - Log aggregation and display
    - Error reporting and visualization
    - Execution history
  - **Dependencies:** E01:S01:T006 (execution engine must exist)
  - **Blocker:** E01:S01:T006
  - **Parallel Development Candidacy:** Blocked until execution engine is complete

- [ ] **E01:S01:T008 – Implement workflow management (versioning, updates, persistence)**
  - **Input:** Workflow definition schema (T004), configuration system
  - **Deliverable:** Workflow management system:
    - Workflow versioning
    - Update and migration system
    - Workflow persistence (file-based or database)
    - Workflow library/catalog
    - Import/export functionality
  - **Dependencies:** E01:S01:T004 (schema must be defined)
  - **Blocker:** None - can be implemented in parallel with execution engine
  - **Parallel Development Candidacy:** Safe - can work in parallel with T006/T007

- [ ] **E01:S01:T009 – Implement release workflow adapter (first use case)**
  - **Input:** Plugin architecture (T002), execution engine (T006), current release workflow script
  - **Deliverable:** Release workflow plugin/adapter:
    - Release workflow type definition
    - Step handlers for release steps (version bump, changelog, etc.)
    - Project-specific integrations (KB, Kanban) as plugin
    - Integration with existing release workflow script
  - **Dependencies:** E01:S01:T002 (plugin architecture), E01:S01:T006 (execution engine), E20:S06:T005 (release script) - ✅ COMPLETE
  - **Blocker:** E01:S01:T002, E01:S01:T006
  - **Parallel Development Candidacy:** Blocked until plugin system and execution engine are complete

- [ ] **E01:S01:T010 – Integrate platform with existing release workflow script**
  - **Input:** Complete platform, release workflow adapter (T009), existing release script
  - **Deliverable:** Integration layer:
    - Bridge between platform and existing script
    - Migration path from script to platform
    - Backward compatibility
    - Testing and validation
  - **Dependencies:** E01:S01:T009 (release adapter must be complete)
  - **Blocker:** E01:S01:T009
  - **Parallel Development Candidacy:** Blocked until release adapter is complete

## Acceptance Criteria

### General-Purpose Platform
- [ ] **Workflow Designer** - Visual drag-and-drop interface for creating and editing workflows
- [ ] **Workflow Definition** - Workflows defined via configuration (YAML/JSON), not hardcoded
- [ ] **Workflow Execution** - Execute workflows with visual monitoring and progress tracking
- [ ] **Workflow Management** - Version, update, and maintain workflow definitions
- [ ] **Plugin System** - Extensible architecture for custom workflow types and integrations
- [ ] **General-Purpose** - Not limited to release workflows, extensible to any workflow type

### Visual Interface
- [ ] **Visual and Intuitive** - Accessible to all team members, NOT CLI-based
- [ ] **Step Configuration** - Visual configuration panels with form controls
- [ ] **Parameter Management** - Input validation, defaults, required/optional indicators
- [ ] **Dependency Visualization** - Visual representation of step dependencies
- [ ] **Progress Tracking** - Real-time visual feedback during execution
- [ ] **Error Handling** - Clear error messages and recovery options

### Release Workflow (First Use Case)
- [ ] **Release Workflow Plugin** - Release workflow implemented as plugin/adapter
- [ ] **Step Configuration** - All release steps configurable visually
- [ ] **Optional Steps** - Toggle controls for optional steps (README, tag, push)
- [ ] **Integration** - Seamless integration with existing release workflow script
- [ ] **Backward Compatibility** - Migration path from script to platform

### Architecture & Independence
- [ ] **Plugin-Based Architecture** - Core engine framework-agnostic, integrations as plugins
- [ ] **Configuration-Driven** - No hardcoded project-specific paths or assumptions
- [ ] **Abstract Interfaces** - Clear interfaces for extensibility (WorkflowExecutor, StepHandler, etc.)
- [ ] **Project Isolation** - Project-specific code isolated in adapter plugins
- [ ] **Architecture Documentation** - Complete documentation of plugin system, schema, extensibility
- [ ] **Open-Source Ready** - Architecture designed for future standalone open-source project

## Implementation Notes

### Technology Stack Options
- **Web-based (RECOMMENDED):** HTML/CSS/JavaScript (vanilla or framework like React/Vue) - Visual, intuitive, accessible
- **Hybrid:** Web UI served locally via Flask/FastAPI - Visual dashboard accessible via browser
- **Desktop:** Electron app or Python GUI (Tkinter, PyQt) - Visual but requires installation
- **CLI-based:** ❌ **NOT ACCEPTABLE** - Does not meet visual dashboard requirement

**Decision Criteria:** Must provide visual, intuitive UX with toggles, dropdowns, and clear visual feedback. CLI-based solutions are explicitly excluded.

### Key Requirements
1. **Step Listing:** Display all release workflow steps in order:
   - Version bump
   - Detailed changelog creation
   - Main changelog update
   - README update (optional)
   - Kanban doc auto-update (automatic)
   - File staging
   - Validator execution
   - Commit
   - Tag creation (optional)
   - Push (optional)

2. **Configuration Per Step (Visual Interface):**
   - Show step description with clear visual hierarchy
   - Display default values in input fields/dropdowns
   - Highlight required inputs visually (e.g., red asterisks, borders)
   - Allow parameter editing with intuitive form controls (text inputs, dropdowns, checkboxes)
   - Provide visual feedback for valid/invalid inputs

3. **Toggle Controls (Visual UX):**
   - Enable/disable optional steps (README, tag, push) with visual toggle switches/checkboxes
   - Prevent disabling mandatory steps (visual indication that they're locked/required)
   - Visual indication of step status (enabled/disabled, completed, in-progress, error)
   - Color coding and icons for quick visual understanding

4. **Preview Generation (Visual Feedback):**
   - Generate exact command: `python scripts/automation/release_workflow.py --summary "..." --type ... --flags`
   - Show what will be executed in a visually distinct preview panel
   - Allow review before execution with clear "Execute" button
   - Visual confirmation of what will happen (step-by-step visual preview)

5. **Preference Persistence:**
   - Store in `.release_preferences.yaml` or `.release_preferences.json`
   - Remember last used values
   - Per-repo configuration

### Integration Points
- Dashboard must call `scripts/automation/release_workflow.py` with generated flags
- Capture stdout/stderr for display
- Handle exit codes and errors gracefully
- Show progress/status during execution

### Architectural Independence (Open-Source Potential)

**Strategic Consideration:** The visual dashboard requirement represents a gap in the market that external frameworks don't address. The architecture should be designed to enable future open-source spin-off as an independent project.

**Architectural Requirements for Independence:**

1. **Plugin-Based Architecture:**
   - Core workflow engine should be framework-agnostic
   - Project-specific integrations (KB structure, Kanban format) should be plugins/adapters
   - Abstract interfaces for all integrations (KB updates, Kanban updates, versioning)

2. **Configuration-Driven Design:**
   - No hardcoded paths or Project-specific assumptions
   - KB structure defined via configuration (YAML/JSON)
   - Kanban format defined via configuration
   - Version schema configurable
   - Changelog formats configurable

3. **Separation of Concerns:**
   - **Core Engine:** Workflow orchestration, step execution, validation
   - **Project Adapter:** KB/Kanban integrations specific to Project
   - **Dashboard UI:** Visual interface (framework-agnostic)
   - **Configuration Layer:** Project-specific settings

4. **Abstract Interfaces:**
   - `KBUpdater` interface (implemented by `ProjectKBUpdater`)
   - `KanbanUpdater` interface (implemented by `ProjectKanbanUpdater`)
   - `VersionManager` interface (implemented by `ProjectVersionManager`)
   - `ChangelogManager` interface (implemented by `ProjectChangelogManager`)

5. **Extensibility Points:**
   - Plugin system for custom integrations
   - Configuration schema for different project structures
   - Hook system for custom validation/transformation
   - Template system for changelog formats

6. **Documentation for Independence:**
   - Architecture documentation explaining plugin system
   - Configuration schema documentation
   - Plugin development guide
   - Examples for different project types

**Implementation Strategy:**
- Phase 1: Build for Project with independence in mind (abstract interfaces, configuration-driven)
- Phase 2: Extract Project-specific code into adapter plugins
- Phase 3: Create generic configuration templates
- Phase 4: Package as standalone open-source project with plugin ecosystem

**Benefits:**
- Enables future open-source spin-off
- Makes codebase more maintainable and testable
- Allows other projects to use the dashboard with their own integrations
- Creates potential value for the broader developer community

## Parallel Development Dependencies

**Story-Level Summary:**
- Depends on E20:S06:T005 (release workflow script must exist and be stable) - ✅ COMPLETE
- **Blocked by E20:S10** (evaluation must complete first) - ✅ COMPLETE - Evaluation complete, decision made to build from scratch

**Task-Level Analysis:**
- **E01:S01:T001:** Safe - design can proceed independently (dependencies complete)
- **E01:S01:T002:** Safe - architecture design can proceed independently (parallel with T001)
- **E01:S01:T003:** Blocked by T001 (design must be complete) and T002 (architecture must be defined)
- **E01:S01:T004:** Blocked by T001, T002 (architecture must be defined)
- **E01:S01:T005:** Blocked by T003, T004 (design and schema must be complete)
- **E01:S01:T006:** Blocked by T001, T002, T004 (architecture and schema must be defined)
- **E01:S01:T007:** Blocked by T006 (execution engine must exist)
- **E01:S01:T008:** Safe - can work in parallel with T006/T007 once T004 is done
- **E01:S01:T009:** Blocked by T002, T006 (plugin system and execution engine must be complete)
- **E01:S01:T010:** Blocked by T009 (release adapter must be complete)

**Parallel Development Candidacy:** **Safe** - All external dependencies (E20:S06, E20:S10) are complete. Can proceed with platform development.

## Completion Summary

**Completed:** v0.21.0.11 (2025-11-20)

**Phase 1 Deliverables:**
- ✅ Complete backend infrastructure (workflow engine, plugin system, API)
- ✅ Workflow definition schema and configuration system
- ✅ Execution engine with monitoring and progress tracking
- ✅ Plugin system with release workflow adapter
- ✅ Integration with existing release workflow script
- ✅ Basic dashboard UI
- ✅ Complete architecture documentation (C4 views, integration architecture)

**Next:** Phase 2 (Story 2) - Visual Workflow Designer UI implementation

## Dependencies
**Blocks:**
- None (platform is an enhancement, not a blocker)

**Blocked By:**
- E20:S06:T005 (release workflow script must exist and be stable) - ✅ COMPLETE
- **E20:S10 (evaluation story must complete first)** - ✅ COMPLETE - Evaluation complete, decision made

**Coordinates With:**
- E20:S10 (evaluation story) - ✅ COMPLETE - Informed technology decision and scope
- E20:S06 (Infrastructure & Tooling) - ✅ COMPLETE - Release workflow script is the foundation

## Risks
- **Risk:** Dashboard may add complexity without sufficient value → *Mitigation:* Validate UX with users before full implementation
- **Risk:** Technology stack choice may not integrate well with existing tooling → *Mitigation:* Evaluate integration options early
- **Risk:** Preference persistence may conflict with script updates → *Mitigation:* Version preference schema and handle migrations

## Notes & References

### Epic & Planning Documents
- **[Epic 01 Overview](../../../epics/overview/Epic%2001/Epic-01.md)** - Complete epic overview with goals, stories, and status
- **[Development Engagement Plan](../../../epics/overview/Epic%2001/Epic-01-Development-Plan.md)** - Full development plan with all phases, tasks, estimates, and dependencies
- **[Phase 2 Implementation Plan](../../../epics/overview/Epic%2001/Epic-01-Phase-2-Plan.md)** - Detailed frontend visual designer implementation plan

### Architecture Documentation (Phase 1 - Completed)
- **[Platform Architecture](../../../../Architecture/Standards_and_ADRs/E01-vwmp-platform-architecture.md)** - High-level platform architecture design (T001 deliverable)
- **[Plugin Architecture](../../../../Architecture/Standards_and_ADRs/E01-vwmp-plugin-architecture.md)** - Plugin system design for extensibility (T002 deliverable)
- **[Visual Designer UI/UX Design](../../../../Architecture/Standards_and_ADRs/E01-vwmp-visual-designer-uiux.md)** - UI/UX design document (T003 deliverable)
- **[Frontend Technology Decision](../../../../Architecture/Standards_and_ADRs/E01-vwmp-frontend-technology-decision.md)** - React + React Flow technology choice (Phase 2)

### C4 Model Architecture Views
- **[Context View](../../../../Architecture/context/overview/epic01-vwmp-context.md)** - VWMP in relation to external actors and systems
- **[Container View](../../../../Architecture/container/overview/epic01-vwmp-container.md)** - Services/modules and their interactions
- **[Component View](../../../../Architecture/component/overview/epic01-vwmp-components.md)** - Internal components with responsibility tables
- **[Runtime View](../../../../Architecture/runtime/reference/epic01-vwmp-sequences.md)** - Sequence diagrams for workflow creation and execution

### Integration Architecture
- **[Integration Architecture](../../../../Architecture/Integration_Architecture/vwmp/vwmp-integration-architecture.md)** - Integration with Git, KB, Kanban, validators
- **[Architecture Gap Analysis](../../../../Architecture/Integration_Architecture/vwmp/VWMP-Architecture-Gap-Analysis.md)** - Alignment with KB Architecture standards
- **[Architecture Completion Summary](../../../../Architecture/Integration_Architecture/vwmp/VWMP-Architecture-Completion-Summary.md)** - Summary of completed architecture documentation

### Research & Decision History
- **[E20:S10 Recommendation Report](../Epic%2020/Story-10-Recommendation-Report.md)** - Decision to build from scratch vs use external frameworks
- **[E20:S10 Framework Evaluation](../Epic%2020/Story-10-Evaluate-Release-Automation-Frameworks.md)** - GitHub Actions, CircleCI, Jenkins evaluation
- **Release workflow script:** `scripts/automation/release_workflow.py`
- **Epic 20 Story 6:** [Infrastructure & Tooling Updates](../Epic%2020/Story-6-Infrastructure-and-Tooling-Updates.md)
- **Epic 20 Story 10:** [Evaluate Release Automation Frameworks](../Epic%2020/Story-10-Evaluate-Release-Automation-Frameworks.md)
