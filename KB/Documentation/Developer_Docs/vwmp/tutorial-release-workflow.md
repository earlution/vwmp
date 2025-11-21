# Tutorial: Building the Release Workflow from Scratch

**Version:** 1.0.0
**Last Updated:** 2025-11-21
**Related:** Epic 01, Story 4 - Comprehensive VWMP Documentation

---

## 🎯 Overview

This tutorial will guide you through building the **Release Workflow** from scratch in the Visual Workflow Management Platform (VWMP). The Release Workflow automates the complete release process: version bumping, changelog generation, Git operations, and validation.

**By the end of this tutorial, you will:**
- Understand how to create workflows in VWMP
- Know how to configure workflow steps and their parameters
- Understand step dependencies and execution order
- Be able to build a complete release automation workflow

**Prerequisites:**
- VWMP is installed and running
- Access to the visual designer at `http://localhost:8000/vwmp/designer/`
- Basic understanding of Git and version control
- Familiarity with YAML syntax

**Estimated Time:** 30-45 minutes

---

## 📋 What We're Building

The Release Workflow consists of **10 steps** that automate the release process:

1. **Bump Version** - Increment version number
2. **Create Detailed Changelog** - Generate detailed changelog file
3. **Update Main Changelog** - Add summary entry to main CHANGELOG.md
4. **Update README** - Update version badge and latest release info
5. **Auto-update Kanban Docs** - Automatically update Kanban documentation
6. **Stage Files** - Stage all changed files with git
7. **Run Validators** - Execute validation scripts
8. **Commit Changes** - Create git commit with version and summary
9. **Create Git Tag** - Create annotated git tag
10. **Push to Remote** - Push branch and tags to remote repository

---

## 🚀 Step-by-Step Tutorial

### Step 1: Open the Visual Designer

1. **Navigate to the designer:**
   ```
   http://localhost:8000/vwmp/designer/
   ```

2. **Click "New Workflow"** to create a new workflow from scratch.

3. **Enter workflow metadata:**
   - **Name:** `Release Workflow`
   - **Description:** `Automated release workflow for version bump, changelog, commit, tag, push`
   - **Type:** Select `release` from the dropdown
   - **Version:** `1.0.0`

4. **Click "Create"** to start building your workflow.

---

### Step 2: Configure Workflow-Level Settings

Before adding steps, we need to configure the workflow-level settings. These are shared across all steps.

1. **Click "Workflow Settings"** (gear icon in the top-right)

2. **Add configuration variables:**
   ```yaml
   version_file: VERSION
   changelog_dir: KB/Changelog_and_Release_Notes/Changelog_Archive
   main_changelog: CHANGELOG.md
   ```

   These variables can be referenced in step configurations using `${config.version_file}`, `${config.changelog_dir}`, etc.

3. **Click "Save"** to apply the configuration.

---

### Step 3: Add Step 1 - Bump Version

This is the first step in our workflow. It increments the version number in the version file.

#### 3.1: Add the Step

1. **Open the Step Palette** (left sidebar)
2. **Expand the "Version" category**
3. **Drag "Bump Version"** onto the canvas

#### 3.2: Configure the Step

1. **Click on the step** to open the configuration panel
2. **Set step properties:**
   - **Step ID:** `step-1` (important: this is used for dependencies)
   - **Name:** `Bump Version`
   - **Required:** ✅ Checked (this step must succeed)
   - **Enabled:** ✅ Checked

3. **Configure step parameters:**
   ```yaml
   version_file: ${config.version_file}
   increment_type: patch
   ```

   **Parameter explanations:**
   - **`version_file`**: Path to the version file (uses workflow config variable)
   - **`increment_type`**: How to increment the version. Options:
     - `patch` - Increment patch version (0.21.0.1 → 0.21.0.2)
     - `minor` - Increment minor version (0.21.0.1 → 0.21.1.0)
     - `major` - Increment major version (0.21.0.1 → 1.0.0.0)
     - `feature` - Increment feature version (0.21.0.1 → 0.21.1.0)
     - `feature_set` - Increment feature set version (0.21.0.1 → 0.22.0.0)

4. **Click "Save"** to apply the configuration.

#### 3.3: Understanding Step Outputs

When this step executes, it will output:
```json
{
  "new_version": "0.21.0.2",
  "old_version": "0.21.0.1"
}
```

These outputs can be accessed by other steps via `step-1.output.new_version`.

---

### Step 4: Add Step 2 - Create Detailed Changelog

This step creates a detailed changelog file after the version is bumped.

#### 4.1: Add the Step

1. **Expand the "Changelog" category** in the Step Palette
2. **Drag "Create Detailed Changelog"** onto the canvas
3. **Position it to the right of Step 1**

#### 4.2: Configure the Step

1. **Click on the step** to open the configuration panel
2. **Set step properties:**
   - **Step ID:** `step-2`
   - **Name:** `Create Detailed Changelog`
   - **Required:** ✅ Checked
   - **Enabled:** ✅ Checked

3. **Add dependency:**
   - **Dependencies:** `step-1` (this step runs after Step 1 completes)

4. **Configure step parameters:**
   ```yaml
   changelog_dir: ${config.changelog_dir}
   format: full_timestamp
   ```

   **Parameter explanations:**
   - **`changelog_dir`**: Directory where detailed changelog files are stored
   - **`format`**: Timestamp format. Currently only `full_timestamp` is supported (format: `YYYY-MM-DD HH:MM:SS UTC`)

5. **Click "Save"** to apply the configuration.

#### 4.3: Connect the Steps

1. **Click on Step 1** (Bump Version)
2. **Hover over the output handle** (right side of the step)
3. **Drag a connection** to Step 2 (Create Detailed Changelog)
4. **The dependency is now visible** as an arrow between the steps

**Why this dependency?**
- Step 2 needs the new version number from Step 1
- It accesses this via `step-1.output.new_version`
- The dependency ensures Step 1 completes before Step 2 starts

---

### Step 5: Add Step 3 - Update Main Changelog

This step adds a summary entry to the main CHANGELOG.md file.

#### 5.1: Add and Configure

1. **Drag "Update Main Changelog"** from the Changelog category
2. **Position it to the right of Step 2**

3. **Configure:**
   - **Step ID:** `step-3`
   - **Name:** `Update Main Changelog`
   - **Required:** ✅ Checked
   - **Dependencies:** `step-2`
   - **Parameters:**
     ```yaml
     main_changelog: ${config.main_changelog}
     date_format: DD-MM-YY
     ```

   **Parameter explanations:**
   - **`main_changelog`**: Path to the main CHANGELOG.md file
   - **`date_format`**: Date format for summary entries. Options: `DD-MM-YY`, `YYYY-MM-DD`, etc.

4. **Connect Step 2 → Step 3**

---

### Step 6: Add Step 4 - Update README

This step updates the README.md with the new version badge and latest release information.

#### 6.1: Add and Configure

1. **Drag "Update README"** from the Documentation category
2. **Position it below Step 1**

3. **Configure:**
   - **Step ID:** `step-4`
   - **Name:** `Update README`
   - **Required:** ❌ Unchecked (optional step)
   - **Enabled:** ✅ Checked
   - **Dependencies:** `step-1` (needs the new version)
   - **Parameters:**
     ```yaml
     readme_file: README.md
     update_badge: true
     update_latest_release: true
     ```

   **Parameter explanations:**
   - **`readme_file`**: Path to README.md
   - **`update_badge`**: Whether to update the version badge (e.g., `![Version](.../version-0.21.0.2-blue)`)
   - **`update_latest_release`**: Whether to update the "Latest Release" callout

4. **Connect Step 1 → Step 4**

---

### Step 7: Add Step 5 - Auto-update Kanban Docs

This step automatically updates Kanban documentation (Epic/Story markdown files).

#### 7.1: Add and Configure

1. **Drag "Auto-update Kanban Docs"** from the Documentation category
2. **Position it below Step 4**

3. **Configure:**
   - **Step ID:** `step-5`
   - **Name:** `Auto-update Kanban Docs`
   - **Required:** ❌ Unchecked (optional step)
   - **Enabled:** ✅ Checked
   - **Automatic:** ✅ Checked (runs automatically during workflow execution)
   - **Dependencies:** `step-1` (needs version to update task checkboxes)
   - **Parameters:**
     ```yaml
     epic_doc_pattern: KB/PM_and_Portfolio/epics/overview/Epic {epic}/Epic-{epic}.md
     kanban_board: KB/PM_and_Portfolio/epics/overview/_index.md
     ```

   **Parameter explanations:**
   - **`epic_doc_pattern`**: Pattern for locating Epic documentation files. `{epic}` is replaced with the epic number extracted from the Git branch name
   - **`kanban_board`**: Path to the main Kanban board index file

4. **Connect Step 1 → Step 5**

---

### Step 8: Add Step 6 - Stage Files

This step stages all changed files using `git add -A`.

#### 8.1: Add and Configure

1. **Drag "Stage Files"** from the Git category
2. **Position it below Step 3**

3. **Configure:**
   - **Step ID:** `step-6`
   - **Name:** `Stage Files`
   - **Required:** ✅ Checked
   - **Dependencies:** `step-1`, `step-2`, `step-3`, `step-4`, `step-5` (wait for all file changes)
   - **Parameters:**
     ```yaml
     paths:
       - "*"
     ```

   **Parameter explanations:**
   - **`paths`**: Array of file paths to stage. Use `["*"]` to stage all files (equivalent to `git add -A`), or specify specific paths like `["README.md", "CHANGELOG.md"]`

4. **Connect all previous steps → Step 6**
   - Step 1 → Step 6
   - Step 2 → Step 6
   - Step 3 → Step 6
   - Step 4 → Step 6
   - Step 5 → Step 6

**Why multiple dependencies?**
- Step 6 must wait for all file-modifying steps to complete
- This ensures all changes are staged together before committing

---

### Step 9: Add Step 7 - Run Validators

This step executes validation scripts to ensure the release is valid.

#### 9.1: Add and Configure

1. **Drag "Run Validators"** from the Validation category
2. **Position it to the right of Step 6**

3. **Configure:**
   - **Step ID:** `step-7`
   - **Name:** `Run Validators`
   - **Required:** ✅ Checked
   - **Dependencies:** `step-6` (run validators after files are staged)
   - **Parameters:**
     ```yaml
     validators:
       - scripts/validation/validate_branch_context.py
       - scripts/validation/validate_changelog_format.py
     strict_mode: true
     ```

   **Parameter explanations:**
   - **`validators`**: Array of validator script paths (relative to workspace root). Each script is executed in order, and all must pass for the step to succeed
   - **`strict_mode`**: If `true`, validators run with `--strict` flag. If any validator fails, the workflow stops.

4. **Connect Step 6 → Step 7**

**What do these validators check?**
- **`validate_branch_context.py`**: Ensures current Git branch matches version schema and CHANGELOG entries
- **`validate_changelog_format.py`**: Ensures detailed changelog files have the correct format (full timestamp)

---

### Step 10: Add Step 8 - Commit Changes

This step creates a git commit with the version and summary.

#### 10.1: Add and Configure

1. **Drag "Commit Changes"** from the Git category
2. **Position it to the right of Step 7**

3. **Configure:**
   - **Step ID:** `step-8`
   - **Name:** `Commit Changes`
   - **Required:** ✅ Checked
   - **Dependencies:** `step-7` (commit only after validators pass)
   - **Parameters:**
     ```yaml
     message_template: "{version} - {summary}"
     ```

   **Parameter explanations:**
   - **`message_template`**: Template for the commit message. Placeholders:
     - `{version}` - Replaced with the new version (from Step 1 output)
     - `{summary}` - Replaced with the release summary (from workflow parameters)

   **Example commit message:**
   ```
   0.21.0.2 - 🐞 Fix: VWMP designer renders Release Workflow
   ```

4. **Connect Step 7 → Step 8**

---

### Step 11: Add Step 9 - Create Git Tag

This step creates an annotated git tag for the release.

#### 11.1: Add and Configure

1. **Drag "Create Git Tag"** from the Git category
2. **Position it below Step 8**

3. **Configure:**
   - **Step ID:** `step-9`
   - **Name:** `Create Git Tag`
   - **Required:** ❌ Unchecked (optional step)
   - **Enabled:** ✅ Checked
   - **Dependencies:** `step-8` (tag after commit)
   - **Parameters:**
     ```yaml
     tag_template: v{version}
     message_template: "Release {tag}: {summary}"
     annotated: true
     ```

   **Parameter explanations:**
   - **`tag_template`**: Template for the tag name. Placeholders:
     - `{version}` - Replaced with the new version
   - **`message_template`**: Template for the tag message. Placeholders:
     - `{tag}` - Replaced with the tag name (e.g., `v0.21.0.2`)
     - `{summary}` - Replaced with the release summary
   - **`annotated`**: If `true`, creates an annotated tag (recommended for releases). If `false`, creates a lightweight tag.

   **Example tag:**
   ```
   Tag name: v0.21.0.2
   Tag message: Release v0.21.0.2: 🐞 Fix: VWMP designer renders Release Workflow
   ```

4. **Connect Step 8 → Step 9**

---

### Step 12: Add Step 10 - Push to Remote

This step pushes the branch and tags to the remote repository.

#### 12.1: Add and Configure

1. **Drag "Push to Remote"** from the Git category
2. **Position it to the right of Step 9**

3. **Configure:**
   - **Step ID:** `step-10`
   - **Name:** `Push to Remote`
   - **Required:** ❌ Unchecked (optional step)
   - **Enabled:** ✅ Checked
   - **Dependencies:** `step-8`, `step-9` (push after commit and tag)
   - **Parameters:**
     ```yaml
     push_tags: true
     remote: origin
     ```

   **Parameter explanations:**
   - **`push_tags`**: If `true`, pushes tags along with the branch (`git push --tags`)
   - **`remote`**: Git remote name (default: `origin`)

4. **Connect Step 8 → Step 10**
5. **Connect Step 9 → Step 10**

**Note:** Step 10 depends on both Step 8 (commit) and Step 9 (tag) because:
- It needs the commit to push the branch
- It needs the tag to push tags (if `push_tags: true`)

---

### Step 13: Configure Workflow Parameters

Workflow parameters are values provided when executing the workflow. Let's add the parameters that the Release Workflow needs.

1. **Click "Workflow Parameters"** (in the workflow metadata panel)

2. **Add Parameter 1: Summary**
   - **Name:** `summary`
   - **Type:** `string`
   - **Required:** ✅ Checked
   - **Label:** `Release Summary`
   - **Description:** `Brief description of the release`
   - **Default:** (leave empty)

3. **Add Parameter 2: Change Type**
   - **Name:** `change_type`
   - **Type:** `enum`
   - **Required:** ✅ Checked
   - **Default:** `tooling`
   - **Label:** `Release Type`
   - **Description:** `Type of release`
   - **Options:** (click "Add Option" for each)
     - `feature` → `Feature 🚀`
     - `fix` → `Fix 🐞`
     - `infrastructure` → `Infrastructure 🔧`
     - `security` → `Security 🔒`
     - `cleanup` → `Cleanup 🧹`
     - `refactor` → `Refactor 🔄`
     - `documentation` → `Documentation 📚`
     - `tooling` → `Tooling 🧰`
     - `maintenance` → `Maintenance 🚧`

4. **Add Parameter 3: Detailed Changes**
   - **Name:** `detailed_changes`
   - **Type:** `textarea`
   - **Required:** ❌ Unchecked
   - **Label:** `Detailed Changes`
   - **Description:** `Optional detailed list of changes`

5. **Add Parameter 4: Skip README**
   - **Name:** `skip_readme`
   - **Type:** `boolean`
   - **Required:** ❌ Unchecked
   - **Default:** `false`
   - **Label:** `Skip README Update`

6. **Add Parameter 5: KB Docs OK**
   - **Name:** `kb_docs_ok`
   - **Type:** `boolean`
   - **Required:** ✅ Checked
   - **Default:** `false`
   - **Label:** `KB Documentation Updated`
   - **Description:** `Confirm all required KB documentation is updated`

7. **Click "Save"** to apply all parameters.

---

### Step 14: Review and Validate Workflow

Before saving, let's review the complete workflow:

1. **Check the workflow graph:**
   - All 10 steps should be visible
   - Dependencies should be correctly connected
   - The workflow should flow from left to right

2. **Expected workflow structure:**
   ```
   Step 1 (Bump Version)
     ├── Step 2 (Create Detailed Changelog)
     │   └── Step 3 (Update Main Changelog)
     ├── Step 4 (Update README)
     ├── Step 5 (Auto-update Kanban Docs)
     └── Step 6 (Stage Files) ← depends on 1,2,3,4,5
         └── Step 7 (Run Validators)
             └── Step 8 (Commit Changes)
                 ├── Step 9 (Create Git Tag)
                 └── Step 10 (Push to Remote) ← depends on 8,9
   ```

3. **Click "Validate Workflow"** (checkmark icon in the top-right)
   - If validation passes, you'll see a green checkmark ✅
   - If validation fails, errors will be displayed. Fix them before proceeding.

---

### Step 15: Save the Workflow

1. **Click "Save Workflow"** (save icon or Ctrl+S / Cmd+S)
2. **Enter a workflow ID** (or leave blank to auto-generate)
3. **Click "Save"**

Your workflow is now saved and ready to use!

---

## 🎓 Understanding the Workflow Structure

### Execution Order

The workflow executes steps based on dependencies:

1. **Step 1** runs first (no dependencies)
2. **Steps 2, 4, 5** run in parallel (all depend on Step 1)
3. **Step 3** runs after Step 2 completes
4. **Step 6** runs after Steps 1, 2, 3, 4, 5 all complete
5. **Steps 7, 8, 9, 10** run sequentially after Step 6

### Step Statuses

During execution, steps show different statuses:
- **🟡 Pending** - Not started yet
- **🔵 Running** - Currently executing
- **🟢 Success** - Completed successfully
- **🔴 Failed** - Failed with error
- **⚪ Skipped** - Skipped (optional step that was disabled or failed dependency)

### Parameter Flow

1. **Workflow Parameters** are provided when starting execution
2. **Step Outputs** from earlier steps are available to later steps
3. **Config Variables** are available to all steps via `${config.variable_name}`

---

## 🧪 Testing the Workflow

1. **Click "Execute Workflow"** (play icon)
2. **Fill in the parameters:**
   - **Summary:** `Test release workflow`
   - **Change Type:** `tooling`
   - **Detailed Changes:** (optional)
   - **Skip README:** `false`
   - **KB Docs OK:** `true`
3. **Click "Run"**

4. **Monitor execution:**
   - Watch steps execute in real-time
   - View logs in the execution panel
   - Check step statuses

5. **Review results:**
   - If successful, all steps should show 🟢 Success
   - If any step fails, review the error logs

---

## 🔍 Troubleshooting

### Common Issues

**Issue: Step fails with "Version not found"**
- **Cause:** Step 2 depends on Step 1 but can't access the version
- **Fix:** Ensure Step 1 completes successfully and outputs `new_version`

**Issue: "Validator failed" error**
- **Cause:** One of the validation scripts failed
- **Fix:** Check the validator logs, fix the issue, and run the workflow again

**Issue: "Git push failed"**
- **Cause:** Remote repository is not accessible or branch doesn't exist
- **Fix:** Ensure you're on the correct branch and have push permissions

---

## 📚 Next Steps

Now that you've built the Release Workflow:

1. **Review the [Release Workflow Reference](release-workflow-reference.md)** for detailed explanations of each step
2. **Explore [Workflow Parameters Guide](workflow-parameters.md)** to understand parameter configuration
3. **Read [Best Practices](best-practices.md)** for workflow design recommendations
4. **Create your own workflows** using the same principles

---

## 📖 Related Documentation

- **[Release Workflow Reference](release-workflow-reference.md)** - Detailed step-by-step reference
- **[Visual Designer Guide](visual-designer-guide.md)** - Using the visual designer
- **[Workflow Execution Guide](execution-guide.md)** - Running and monitoring workflows
- **[Release Workflow Usage](release-workflow-usage.md)** - Using the Release Workflow

---

**Last Updated:** 2025-11-21
**Tutorial Version:** 1.0.0
