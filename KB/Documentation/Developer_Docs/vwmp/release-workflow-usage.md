# Release Workflow Usage Guide

**Version:** 1.0.0
**Last Updated:** 2025-11-21
**Related:** Epic 01, Story 3 - Implement Workflows in VWMP

---

## Overview

The Release Workflow is now fully integrated into the Visual Workflow Management Platform (VWMP). This workflow automates the complete release process: version bumping, changelog generation, README updates, Kanban documentation updates, Git operations, and validation.

**Key Features:**
- ✅ Visual workflow designer interface
- ✅ Drag-and-drop workflow editing
- ✅ Step-by-step configuration
- ✅ Real-time execution monitoring
- ✅ Full integration with existing release workflow functionality

---

## Accessing the Release Workflow

### Via Visual Designer

1. **Navigate to the Designer:**
   ```
   http://localhost:8000/vwmp/designer/
   ```

2. **Load Existing Workflow:**
   - The Release Workflow should appear in the workflow list
   - Click "Load" or navigate directly to:
   ```
   http://localhost:8000/vwmp/designer/<workflow-id>/
   ```

3. **Get Workflow ID:**
   - Check the workflow metadata file: `workflows/workflows_metadata.json`
   - Or list workflows via API: `GET /api/vwmp/workflows/`
   - Or use the import command output

### Via API

**List Workflows:**
```bash
curl -X GET http://localhost:8000/api/vwmp/workflows/ \
  -H "Authorization: Bearer <token>"
```

**Get Workflow:**
```bash
curl -X GET http://localhost:8000/api/vwmp/workflows/<workflow-id>/ \
  -H "Authorization: Bearer <token>"
```

**Execute Workflow:**
```bash
curl -X POST http://localhost:8000/api/vwmp/workflows/<workflow-id>/execute/ \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "summary": "🚀 Feature: New feature description",
    "change_type": "feature",
    "kb_docs_ok": true
  }'
```

---

## Importing the Release Workflow

If the workflow hasn't been imported yet, use the management command:

```bash
python manage.py import_workflow workflows/release-workflow.yaml
```

**Options:**
- `--workflow-id <id>` - Specify a custom workflow ID
- `--storage-root <path>` - Custom storage root directory
- `--dry-run` - Validate without importing
- `--overwrite` - Overwrite existing workflow

**Example:**
```bash
# Dry-run to validate
python manage.py import_workflow workflows/release-workflow.yaml --dry-run

# Import the workflow
python manage.py import_workflow workflows/release-workflow.yaml

# Import with custom ID
python manage.py import_workflow workflows/release-workflow.yaml --workflow-id release-workflow
```

---

## Workflow Configuration

### Workflow Steps

The Release Workflow consists of 10 steps:

1. **Bump Version** - Increments version in `VERSION`
2. **Create Detailed Changelog** - Creates detailed changelog file with full timestamp
3. **Update Main Changelog** - Updates summary in `CHANGELOG.md`
4. **Update README** - Updates version badge and latest release callout
5. **Auto-update Kanban Docs** - Updates Epic/Story documentation
6. **Stage Files** - Stages all modified files with `git add -A`
7. **Run Validators** - Executes branch context and changelog format validators
8. **Commit Changes** - Commits with versioned message template
9. **Create Git Tag** - Creates annotated tag with version
10. **Push to Remote** - Pushes branch and tags to remote repository

### Workflow Parameters

When executing the workflow, provide these parameters:

**Required:**
- `summary` (string) - Brief description of the release
- `change_type` (enum) - Type of release:
  - `feature` 🚀
  - `fix` 🐞
  - `infrastructure` 🔧
  - `security` 🔒
  - `cleanup` 🧹
  - `refactor` 🔄
  - `documentation` 📚
  - `tooling` 🧰
  - `maintenance` 🚧
- `kb_docs_ok` (boolean) - Confirm KB documentation is updated

**Optional:**
- `detailed_changes` (textarea) - Detailed list of changes
- `skip_readme` (boolean) - Skip README update step

### Workflow Configuration

Workflow-level configuration (in YAML):

```yaml
config:
  version_file: VERSION
  changelog_dir: KB/Changelog_and_Release_Notes/Changelog_Archive
  main_changelog: CHANGELOG.md
```

---

## Editing the Workflow

### Via Visual Designer

1. **Open Workflow:**
   - Navigate to `/vwmp/designer/<workflow-id>/`
   - Workflow loads automatically

2. **Edit Steps:**
   - Drag steps from palette onto canvas
   - Click on nodes to configure step parameters
   - Configure dependencies by connecting nodes
   - Use the right panel for step configuration

3. **Edit Workflow Metadata:**
   - Click the workflow name in the header
   - Edit name, description, type, and parameters
   - Configure workflow-level config variables

4. **Save Changes:**
   - Click "Save" button or use `Ctrl+S`
   - Workflow is saved to VWMP storage

### Via YAML File

The workflow YAML file is stored at:
```
workflows/workflows/<workflow-id>-<workflow_name>.yaml
```

You can edit this file directly, then re-import:
```bash
python manage.py import_workflow workflows/workflows/<workflow-id>-Release_Workflow.yaml --overwrite
```

---

## Executing the Workflow

### Via Visual Designer

1. **Open Workflow:**
   - Navigate to `/vwmp/designer/<workflow-id>/`

2. **Configure Parameters:**
   - Click "Execute" button
   - Fill in required parameters (summary, change_type, kb_docs_ok)
   - Optionally add detailed changes

3. **Monitor Execution:**
   - Watch real-time status updates on nodes
   - View execution log in bottom panel
   - Track progress with progress bar

4. **Review Results:**
   - Check execution log for errors
   - Verify all steps completed successfully
   - Confirm version bump, changelog, Git operations

### Via API

```bash
curl -X POST http://localhost:8000/api/vwmp/workflows/<workflow-id>/execute/ \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "summary": "📚 Documentation: Update workflow usage guide",
    "change_type": "documentation",
    "kb_docs_ok": true,
    "detailed_changes": "Added comprehensive usage guide for Release Workflow"
  }'
```

### Via Legacy Script (Backwards Compatible)

The legacy `release_workflow.py` script still works and can use the VWMP engine:

```bash
python scripts/automation/release_workflow.py \
  --summary "🚀 Feature: New feature" \
  --type feature \
  --kb-docs-ok \
  --use-vwmp
```

---

## Workflow Storage

Workflows are stored in:
- **Storage Root:** `workflows/`
- **Workflow Files:** `workflows/workflows/<workflow-id>-<name>.yaml`
- **Metadata Index:** `workflows/workflows_metadata.json`
- **Version History:** `workflows/versions/<workflow-id>/`

The storage system provides:
- ✅ Versioning and history tracking
- ✅ Metadata management
- ✅ Workflow discovery and listing
- ✅ Search and filtering

---

## Testing the Workflow

### Validation Test

Validate the workflow without executing:
```bash
python manage.py import_workflow workflows/release-workflow.yaml --dry-run
```

### End-to-End Test

1. **Import Workflow:**
   ```bash
   python manage.py import_workflow workflows/release-workflow.yaml
   ```

2. **Load in Designer:**
   - Open `/vwmp/designer/<workflow-id>/`
   - Verify all steps are visible
   - Check step configuration

3. **Test Execution:**
   - Use test parameters (summary, change_type, etc.)
   - Execute and monitor
   - Verify results match legacy script output

### Comparison with Legacy Script

The VWMP workflow should produce identical results to:
```bash
python scripts/automation/release_workflow.py \
  --summary "Test release" \
  --type documentation \
  --kb-docs-ok \
  --auto-go
```

---

## Troubleshooting

### Workflow Not Found

**Problem:** Workflow doesn't appear in designer or API.

**Solution:**
1. Check if workflow was imported:
   ```bash
   cat workflows/workflows_metadata.json
   ```
2. Re-import if needed:
   ```bash
   python manage.py import_workflow workflows/release-workflow.yaml --overwrite
   ```

### Validation Errors

**Problem:** Workflow validation fails.

**Solution:**
1. Check workflow YAML syntax
2. Verify all step handlers exist in plugin registry
3. Check step dependencies are valid
4. Ensure workflow type plugin is registered

### Execution Errors

**Problem:** Workflow execution fails.

**Solution:**
1. Check execution log in designer
2. Verify all required parameters are provided
3. Check step handler implementation
4. Verify file paths and permissions

---

## Related Documentation

- **[VWMP Architecture](../../../../Architecture/Standards_and_ADRs/E01-vwmp-platform-architecture.md)** - Platform architecture overview
- **[Visual Designer UI/UX](../../../../Architecture/Standards_and_ADRs/E01-vwmp-visual-designer-uiux.md)** - Designer interface documentation
- **[Plugin Architecture](../../../../Architecture/Standards_and_ADRs/E01-vwmp-plugin-architecture.md)** - Plugin system documentation
- **[Release Workflow Plugin](../../../../../../src/vwmp/plugins/release/)** - Release workflow plugin implementation

---

## Version History

- **v1.0.0** (2025-11-21) - Initial Release Workflow implementation in VWMP
