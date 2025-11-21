# Epic 01 – VWMP Context View (C4 Level 1)

**Status:** Draft
**Epic:** Epic 01 - Visual Workflow Management Platform (VWMP)
**Last Updated:** 2025-11-20
**Related Artifacts:**
- `KB/Architecture/container/overview/epic01-vwmp-container.md`
- `KB/Architecture/component/overview/epic01-vwmp-components.md`
- `KB/Architecture/Standards_and_ADRs/E01-vwmp-platform-architecture.md`
- `KB/Architecture/Diagrams/Context/epic01-vwmp-context.puml`

---

## 1. Context & Goals

The Visual Workflow Management Platform (VWMP) provides a visual, intuitive interface for defining, executing, and monitoring workflows in software projects. It addresses the gap left by CLI-based tools by making workflow management accessible to all team members, starting with the release workflow as the first use case.

**Primary Goals:**
- Visual workflow definition and management (NOT CLI-based)
- General-purpose workflow platform extensible to any workflow type
- Plugin-based architecture for independence and open-source potential
- Intuitive interface accessible to non-technical users

---

## 2. System Context Diagram

![C4 Context Diagram](../../Diagrams/Context/epic01-vwmp-context.puml)

### External Actors & Systems

| Actor/System | Type | Description | Interactions |
| --- | --- | --- | --- |
| **Developers** | Person | Software developers using VWMP to execute workflows | Use Visual Designer to create/edit workflows, execute workflows, monitor execution |
| **Project Managers** | Person | Non-technical users who need to manage workflows | Use Visual Designer to configure workflows, view execution history |
| **Release Manager** | Person | Primary user for release workflow use case | Execute release workflows, configure release parameters, monitor releases |
| **Git Repository** | System | Version control system storing code and workflow definitions | VWMP reads/writes workflow definitions, executes git operations (commit, tag, push) |
| **Knowledge Base (KB)** | System | Confidentia knowledge base with changelogs and documentation | VWMP creates detailed changelogs, updates main changelog, auto-updates Kanban docs |
| **Kanban Board** | System | Project management board (Trello/synced via Epic 18) | VWMP auto-updates epic/story metadata, updates Kanban board entries |
| **Version File** | System | Project version file (`src/confidentia/version.py`) | VWMP reads current version, bumps version for releases |
| **README.md** | System | Project README file | VWMP updates version badge and latest release callout (optional) |
| **Validator Scripts** | System | Python validation scripts (`validate_branch_context.py`, `validate_changelog_format.py`) | VWMP executes validators before commits, handles validation errors |
| **File System** | System | Local file system for workflow definitions and execution artifacts | VWMP stores workflow definitions (YAML/JSON), execution history, logs |

---

## 3. Key Interactions

### Workflow Creation Flow
1. **Developer/PM** uses Visual Designer to create new workflow
2. **VWMP** validates workflow definition and stores in file system
3. **VWMP** makes workflow available for execution

### Workflow Execution Flow (Release Workflow Example)
1. **Release Manager** selects release workflow and provides parameters (summary, type, etc.)
2. **VWMP** reads current version from Version File
3. **VWMP** executes workflow steps:
   - Bumps version in Version File
   - Creates detailed changelog in KB
   - Updates main CHANGELOG.md
   - Optionally updates README.md
   - Auto-updates Kanban Board via Epic 18 integration
   - Stages files in Git Repository
   - Executes Validator Scripts
   - Commits changes to Git Repository
   - Optionally creates git tag
   - Optionally pushes to Git Repository
4. **VWMP** provides real-time execution status to Release Manager
5. **Release Manager** monitors execution progress and handles any errors

### Workflow Management Flow
1. **Developer** uses Visual Designer to edit existing workflow
2. **VWMP** validates changes and updates workflow definition
3. **VWMP** maintains version history of workflow definitions
4. **Developer** can view execution history and analyze workflow performance

---

## 4. System Boundaries

### In Scope (VWMP System)
- Visual workflow designer and UI
- Workflow execution engine
- Plugin system for workflow types
- Workflow definition storage and versioning
- Execution monitoring and history
- Integration adapters for common systems (Git, KB, Kanban)

### Out of Scope (External Systems)
- Git repository management (VWMP executes git commands but doesn't manage repositories)
- KB content creation (VWMP writes files but doesn't manage KB structure)
- Kanban board management (VWMP updates entries but doesn't manage boards)
- Validation logic (VWMP executes validators but validators are separate systems)
- Workflow execution logic (handled by step handlers/plugins)

---

## 5. Integration Points

### Git Integration
- **Purpose:** Version control operations (stage, commit, tag, push)
- **Method:** Execute git commands via subprocess
- **Data Flow:** VWMP → Git commands → Git Repository
- **Error Handling:** Validate git operations, handle errors gracefully

### Knowledge Base Integration
- **Purpose:** Create changelogs, update documentation
- **Method:** File system operations (read/write markdown files)
- **Data Flow:** VWMP → File writes → KB files
- **Error Handling:** Validate file paths, handle write failures

### Kanban Integration
- **Purpose:** Auto-update epic/story metadata and Kanban board
- **Method:** Via Epic 18 MCP integration (Trello sync) or direct file updates
- **Data Flow:** VWMP → Kanban updater → Kanban Board / Files
- **Error Handling:** Graceful degradation if Kanban update fails (workflow continues)

### Validator Integration
- **Purpose:** Validate branch context and changelog format before commits
- **Method:** Execute validator scripts via subprocess
- **Data Flow:** VWMP → Validator scripts → Validation results
- **Error Handling:** Block workflow if validation fails, show errors to user

---

## 6. Non-Functional Requirements

### Usability
- **Visual Interface:** Must be intuitive, no CLI knowledge required
- **Accessibility:** Accessible to non-technical users (project managers)
- **Documentation:** Clear help text and tooltips for all workflow steps

### Performance
- **Workflow Execution:** Typical release workflow completes in <30 seconds
- **UI Responsiveness:** Real-time updates via WebSocket for execution monitoring
- **Scalability:** Support hundreds of workflow definitions and thousands of executions

### Reliability
- **Error Handling:** Graceful error handling with clear error messages
- **Retry Logic:** Configurable retry for transient failures
- **Recovery:** Partial success reporting, rollback capabilities (future)

### Security
- **Input Validation:** All user inputs validated against schemas
- **File System Access:** Restricted to workspace directory
- **Git Operations:** Validate git commands before execution
- **Permission Checks:** Verify user permissions for workflow execution

---

## 7. Open Questions

1. **Authentication:** How do users authenticate? Django auth or separate auth system?
2. **Multi-tenancy:** Will VWMP support multiple projects/workspaces?
3. **Cloud Deployment:** Will VWMP be deployable as standalone service or only integrated with Confidentia?
4. **Workflow Sharing:** Can workflows be shared across projects or organizations?

---

## 8. Related Documents

- **Container View:** `KB/Architecture/container/overview/epic01-vwmp-container.md`
- **Component View:** `KB/Architecture/component/overview/epic01-vwmp-components.md`
- **Integration Architecture:** `KB/Architecture/Integration_Architecture/vwmp/vwmp-integration-architecture.md`
- **Main Architecture:** `KB/Architecture/Standards_and_ADRs/E01-vwmp-platform-architecture.md`
- **Epic 01 Story:** `KB/PM_and_Portfolio/stories/overview/Epic 01/Story-1-Visual-Workflow-Management-Platform.md`

---

**Next Steps:** Proceed to Container View for service/module breakdown
