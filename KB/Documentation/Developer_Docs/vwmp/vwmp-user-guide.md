# VWMP User Guide

**Version:** 1.0.0
**Last Updated:** 2025-11-21
**Related:** Epic 01 - Visual Workflow Management Platform

---

## 📖 Overview

This guide provides comprehensive instructions for using the Visual Workflow Management Platform (VWMP). You'll learn how to navigate the dashboard, use the visual designer, execute workflows, monitor execution, and manage your workflows.

**Who This Guide Is For:**
- New users getting started with VWMP
- Users creating and editing workflows
- Users executing and monitoring workflows
- Users managing workflow libraries

---

## 🚀 Getting Started

### Accessing VWMP

1. **Start the FastAPI development server:**
   ```bash
   python manage.py runserver
   ```

2. **Navigate to the VWMP dashboard:**
   ```
   http://localhost:8000/vwmp/
   ```

3. **Or go directly to the visual designer:**
   ```
   http://localhost:8000/vwmp/designer/
   ```

### First Steps

1. **Explore the Dashboard** - View existing workflows and statistics
2. **Create a New Workflow** - Click "New Workflow" in the designer
3. **Follow a Tutorial** - Build the [Release Workflow](tutorial-release-workflow.md) or [Kanban Review Workflow](tutorial-kanban-review-workflow.md)

---

## 📊 Dashboard

The VWMP dashboard (`/vwmp/`) provides an overview of all your workflows.

### Dashboard Features

**Statistics Cards:**
- **Total Workflows** - Number of workflows in your workspace
- **Total Executions** - Total number of workflow executions
- **Workflow Types** - Number of different workflow types

**Workflow List:**
- Displays all workflows with their:
  - Name and description
  - Version
  - Type (e.g., "release", "kanban_review")
  - Last executed timestamp
  - Actions (View, Execute, Delete)

### Dashboard Actions

**View Workflow:**
- Click the **"View"** button to open the workflow in the visual designer
- Allows you to edit, configure, and execute the workflow

**Execute Workflow:**
- Click the **"Execute"** button to run the workflow immediately
- Confirmation prompt appears before execution
- Execution results are shown after completion

**Delete Workflow:**
- Click the **"Delete"** button to remove a workflow
- Confirmation prompt appears before deletion
- ⚠️ **Warning:** This action cannot be undone

**Create New Workflow:**
- Click **"New Workflow"** to open the designer with a blank canvas
- Start building your workflow from scratch

---

## 🎨 Visual Designer

The visual designer (`/vwmp/designer/`) is the main interface for creating and editing workflows.

### Designer Layout

**Header:**
- Workflow name and type badge
- **New** button - Create a new workflow
- **Save** button - Save current workflow
- **Execute** button - Run the workflow
- **?** button - Show keyboard shortcuts help

**Sidebar:**
- **Step Types** - Categorized palette of available step handlers
- **Templates** - Pre-built workflow templates

**Canvas:**
- Main workspace for building your workflow
- Drag-and-drop interface
- Interactive node-based editor

### Creating a New Workflow

1. **Click "New Workflow"** in the header or sidebar
2. **Enter workflow metadata:**
   - Name (required)
   - Version (default: 1.0.0)
   - Type (e.g., "release", "kanban_review", "custom")
   - Description (optional)
3. **Add workflow configuration** (optional):
   - Define config variables used across steps
   - Example: `version_file: VERSION`
4. **Start adding steps** from the palette

### Adding Steps to Your Workflow

**Method 1: Drag from Palette**
1. Browse the **Step Types** palette in the sidebar
2. **Search or filter** by category if needed
3. **Drag a step** from the palette onto the canvas
4. **Release** to drop the step on the canvas

**Method 2: Click to Add**
1. Click on a step in the palette
2. Step is added to the center of the canvas
3. Drag to reposition

**Method 3: From Template**
1. Browse **Templates** in the sidebar
2. Click a template to load it
3. Customize as needed

### Configuring Steps

1. **Click a step** on the canvas to select it
2. **Configuration panel** appears on the right
3. **Configure step parameters:**
   - Enter values in form fields
   - Use `${config.variable_name}` to reference workflow config
   - Required fields are marked with asterisk (*)
4. **Set dependencies:**
   - Click "Dependencies" section
   - Select which steps must complete before this step
5. **Set step properties:**
   - Required (can this step be skipped?)
   - Enabled (is this step active?)
6. **Changes are saved automatically** to the workflow state

### Connecting Steps (Dependencies)

**Visual Connection:**
1. **Hover over a step** to see connection handles
2. **Click and drag** from the output handle of one step
3. **Drag to** the input handle of another step
4. **Release** to create the dependency

**Via Configuration:**
1. Select the step that should depend on another
2. Open the **Dependencies** section in the config panel
3. **Select dependencies** from the list of available steps

**Dependency Rules:**
- Steps can depend on multiple other steps
- Circular dependencies are prevented automatically
- Steps without dependencies run first
- Steps with dependencies wait for all dependencies to complete

### Editing Workflow Metadata

1. **Click "Workflow Config"** button in the header
2. **Edit metadata:**
   - Name, version, type, description
3. **Add configuration variables:**
   - Define variables used across steps
   - Example: `kanban_root: KB/PM_and_Portfolio`
4. **Add workflow parameters:**
   - Define parameters users can provide when executing
   - Example: `auto_apply` (boolean, default: false)

### Keyboard Shortcuts

Press **?** in the designer to see all available shortcuts:

**Workflow Management:**
- `Ctrl+N` / `Cmd+N` - Create new workflow
- `Ctrl+S` / `Cmd+S` - Save workflow
- `Ctrl+E` / `Cmd+E` - Execute workflow
- `Ctrl+/` - Show keyboard shortcuts help

**Canvas Navigation:**
- `Ctrl+Mouse Wheel` - Zoom in/out
- `Space+Drag` - Pan canvas
- `Delete` / `Backspace` - Delete selected step
- `Ctrl+F` - Fit view to show all steps
- `Escape` - Deselect all

**Step Management:**
- `Double-click step` - Open configuration panel
- `Right-click step` - Context menu (delete, duplicate, etc.)

### Saving Workflows

**Automatic Saving:**
- Workflow state is saved to browser local storage
- Prevents data loss if browser closes unexpectedly

**Manual Save:**
1. Click **"Save"** button in the header (or press `Ctrl+S`)
2. If workflow is new, it will be created
3. If workflow exists, it will be updated
4. **Success notification** appears when saved

**Save Requirements:**
- Workflow must have a name
- All steps must have valid handlers
- No circular dependencies allowed

### Workflow Validation

Before saving or executing, VWMP validates:
- ✅ All steps have valid handler paths
- ✅ No circular dependencies exist
- ✅ Required step parameters are provided
- ✅ Dependencies reference valid step IDs
- ✅ Workflow name and type are set

**Validation Errors:**
- Displayed as red badges on steps
- Listed in the validation panel
- Must be fixed before execution

---

## ▶️ Executing Workflows

### Starting Execution

**From Designer:**
1. **Click "Execute"** button in the header (or press `Ctrl+E`)
2. **Review parameters** if the workflow has any
3. **Click "Execute"** to start
4. **Execution log panel** opens automatically

**From Dashboard:**
1. Find the workflow in the list
2. Click **"Execute"** button
3. Confirmation prompt appears
4. Results shown after completion

### Execution Monitoring

**Real-Time Status Updates:**
- Steps change color based on status:
  - **Gray** - Pending (not started)
  - **Blue** - Running (in progress)
  - **Green** - Success (completed successfully)
  - **Red** - Failed (error occurred)
  - **Orange** - Skipped (not executed)

**Execution Log Panel:**
- **Opens automatically** when execution starts
- Shows real-time logs from each step
- **Log levels:** Info, Success, Warning, Error
- **Filter logs** by step or log level
- **Scroll automatically** to latest log entry

**Progress Indicator:**
- Shows overall workflow progress
- Displays completed/total steps
- Updates in real-time

### Viewing Execution Results

**After Execution Completes:**
- **Execution summary** appears
- Shows overall status (Success/Failed)
- Displays execution time
- Lists step-by-step results

**Step Results:**
- Click a step to see its result
- View output data (if any)
- See error messages (if failed)
- Check execution time per step

**Execution History:**
- Last executed timestamp saved to workflow metadata
- View in dashboard workflow list
- Helps track workflow usage

### Execution Parameters

Some workflows accept parameters when executing:

**Providing Parameters:**
1. Click **"Execute"** button
2. **Parameter form** appears (if workflow has parameters)
3. **Enter values** for each parameter
4. Parameters reference `${parameters.param_name}` in steps

**Example Parameters:**
- `auto_apply` (boolean) - Auto-apply fixes in Kanban Review
- `bloat_threshold` (integer) - Document size threshold
- `summary` (string) - Release summary text

---

## 🔍 Monitoring and Debugging

### Execution Logs

**Log Panel Features:**
- **Real-time updates** as workflow executes
- **Color-coded by level:**
  - Info (blue) - General information
  - Success (green) - Successful operations
  - Warning (orange) - Warnings
  - Error (red) - Errors and failures
- **Filter by step** - Show logs for specific steps
- **Filter by level** - Show only errors, warnings, etc.
- **Search logs** - Find specific log entries
- **Export logs** - Copy logs for analysis

**Reading Logs:**
- Each log entry includes:
  - Timestamp
  - Step name/ID
  - Log level
  - Message
- **Error logs** show stack traces when available
- **Step output** shown for successful steps

### Debugging Failed Workflows

**Identify the Problem:**
1. **Check execution log** for error messages
2. **Identify failed step** (red status)
3. **Review step configuration** - ensure all parameters are correct
4. **Check dependencies** - ensure prerequisite steps completed

**Common Issues:**
- **Missing required parameters** - Check step config panel
- **Invalid file paths** - Verify paths exist and are correct
- **Permission errors** - Check file/directory permissions
- **Dependency failures** - Fix upstream steps first

**Debugging Steps:**
1. **Isolate the problem** - Enable/disable steps to find issue
2. **Check step output** - Review output from previous steps
3. **Validate configuration** - Ensure all config values are correct
4. **Test individually** - Run problematic step in isolation if possible
5. **Review findings files** - Some workflows save findings (e.g., `kanban_review_findings.json`)

### Workflow Status Indicators

**In Designer:**
- **Green checkmark** - Workflow is valid and ready
- **Red error badge** - Validation errors exist
- **Yellow warning** - Warnings (non-blocking)

**On Canvas:**
- **Step colors** reflect execution status
- **Connection lines** show dependency flow
- **Animated indicators** show active steps during execution

---

## 💾 Workflow Management

### Loading Workflows

**From Dashboard:**
1. Click **"View"** button next to a workflow
2. Opens workflow in designer

**From Designer:**
1. Click **"New Workflow"** dropdown (if available)
2. Select **"Load Workflow"**
3. Choose from list of workflows

**Direct URL:**
- Navigate to `/vwmp/designer/<workflow-id>/`
- Workflow loads automatically

### Saving Workflows

**First Save:**
1. Configure workflow name and metadata
2. Add at least one step
3. Click **"Save"** button
4. Workflow is created with unique ID
5. URL updates to include workflow ID

**Subsequent Saves:**
- Click **"Save"** button
- Workflow is updated
- No confirmation needed (unless there are unsaved changes)

**Auto-Save:**
- Workflow state saved to browser storage
- Prevents data loss
- Manual save still required to persist to server

### Versioning Workflows

**Workflow Version:**
- Set in workflow metadata
- Default: `1.0.0`
- Update when making significant changes
- Follow semantic versioning if desired

**Version History:**
- Currently, each save overwrites previous version
- Future: Version history and rollback support

### Deleting Workflows

**From Dashboard:**
1. Find workflow in list
2. Click **"Delete"** button
3. Confirm deletion
4. ⚠️ **Warning:** Cannot be undone

**From Designer:**
- Delete functionality may be available via workflow settings
- Or return to dashboard to delete

### Workflow Templates

**Using Templates:**
1. Browse **Templates** section in sidebar
2. Click a template to load it
3. Customize steps and configuration
4. Save as new workflow

**Available Templates:**
- Release Workflow
- Kanban Review Workflow
- Custom templates (if configured)

---

## 🎯 Best Practices

### Workflow Design

**Plan Before Building:**
1. **Define workflow goals** - What should this workflow accomplish?
2. **Identify required steps** - What operations are needed?
3. **Map dependencies** - Which steps depend on others?
4. **Consider error handling** - What if a step fails?

**Organize Steps:**
- **Group related steps** visually on canvas
- **Use clear step names** - Describe what each step does
- **Add step descriptions** if handler supports it
- **Minimize dependencies** where possible for parallel execution

**Configuration:**
- **Use workflow config** for shared values
- **Reference config variables** in steps: `${config.variable_name}`
- **Define parameters** for user-provided values
- **Document configuration** in workflow description

### Execution

**Test Before Production:**
- **Test with sample data** first
- **Verify all steps work** individually
- **Check error handling** with invalid inputs
- **Validate output** meets expectations

**Monitor Execution:**
- **Watch execution logs** in real-time
- **Review step results** after completion
- **Check for warnings** even if execution succeeds
- **Save logs** for analysis if needed

**Parameter Management:**
- **Document all parameters** in workflow description
- **Provide sensible defaults** where possible
- **Validate parameter values** in workflow config
- **Use type validation** (boolean, integer, string, etc.)

### Maintenance

**Keep Workflows Updated:**
- **Update step handlers** when new versions available
- **Review workflow config** periodically
- **Test after updates** to ensure compatibility
- **Version workflows** when making significant changes

**Documentation:**
- **Add descriptions** to workflows and steps
- **Document configuration** variables
- **Explain parameter usage**
- **Note any special requirements**

---

## 🔗 Integration

### API Usage

VWMP provides REST API endpoints for programmatic access:

**List Workflows:**
```
GET /api/vwmp/workflows/
```

**Get Workflow:**
```
GET /api/vwmp/workflows/<workflow-id>/
```

**Create Workflow:**
```
POST /api/vwmp/workflows/
Body: { workflow definition }
```

**Update Workflow:**
```
PUT /api/vwmp/workflows/<workflow-id>/
Body: { workflow definition }
```

**Execute Workflow:**
```
POST /api/vwmp/workflows/<workflow-id>/execute/
Body: { "parameters": {...} }
```

**Get Step Handlers:**
```
GET /api/vwmp/step-handlers/
```

**Get Step Handler Schema:**
```
GET /api/vwmp/step-handlers/<handler-path>/schema/
```

**Authentication:**
- API endpoints require authentication
- Use Authentication system
- Include credentials in API requests

### Command-Line Integration

**Workflow Execution:**
- Some workflows can be executed from command line
- Example: Release Workflow can be run via `release_workflow.py`
- Check individual workflow documentation for CLI usage

---

## 📚 Next Steps

- **[Tutorial: Building Release Workflow](tutorial-release-workflow.md)** - Learn by building a complete workflow
- **[Tutorial: Building Kanban Review Workflow](tutorial-kanban-review-workflow.md)** - Build another example workflow
- **[Release Workflow Reference](release-workflow-reference.md)** - Detailed reference for Release Workflow
- **[Kanban Review Workflow Reference](kanban-review-workflow-reference.md)** - Detailed reference for Kanban Review Workflow
- **[Best Practices Guide](best-practices.md)** - Workflow design patterns and recommendations
- **[Troubleshooting Guide](troubleshooting.md)** - Common issues and solutions

---

**Last Updated:** 2025-11-21
**Guide Version:** 1.0.0
