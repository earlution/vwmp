# Release Workflow Reference

**Version:** 1.0.0
**Last Updated:** 2025-11-21
**Related:** Epic 01, Story 4 - Comprehensive VWMP Documentation

---

## 📖 Overview

This document provides a comprehensive reference for the **Release Workflow**, explaining each step, its parameters, configuration options, and how they work together. Use this as a detailed reference after completing the [Tutorial: Building Release Workflow from Scratch](tutorial-release-workflow.md).

---

## 📋 Workflow Structure

The Release Workflow consists of **10 steps** organized into phases:

### Phase 1: Version & Documentation Updates
- **Step 1:** Bump Version
- **Step 2:** Create Detailed Changelog
- **Step 3:** Update Main Changelog
- **Step 4:** Update README
- **Step 5:** Auto-update Kanban Docs

### Phase 2: Git Operations & Validation
- **Step 6:** Stage Files
- **Step 7:** Run Validators
- **Step 8:** Commit Changes
- **Step 9:** Create Git Tag
- **Step 10:** Push to Remote

---

## 🔧 Workflow-Level Configuration

Before diving into individual steps, let's understand the workflow-level configuration that applies to all steps.

### Configuration Variables

These variables are defined at the workflow level and can be referenced in any step:

```yaml
config:
  version_file: VERSION
  changelog_dir: KB/Changelog_and_Release_Notes/Changelog_Archive
  main_changelog: CHANGELOG.md
```

**Usage in steps:**
- Reference these variables using `${config.variable_name}`
- Example: `${config.version_file}` resolves to `VERSION`
- This allows easy reconfiguration without changing individual step configs

---

## 📝 Step-by-Step Reference

### Step 1: Bump Version

**Handler:** `release.version_bump`
**Category:** Version
**Icon:** 🔢
**Required:** ✅ Yes
**Default Dependencies:** None (first step)

#### Purpose

Increments the version number in the version file. This is typically the first step in a release workflow as other steps depend on the new version number.

#### Execution Flow

1. Reads current version from `version_file`
2. Increments version based on `increment_type`
3. Updates `version_file` with new version
4. Outputs `new_version` and `old_version` for use by other steps

#### Configuration Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `version_file` | string | ✅ Yes | - | Path to version file relative to workspace root (e.g., `VERSION`) |
| `increment_type` | enum | ❌ No | `patch` | How to increment the version |

#### `increment_type` Options

| Value | Description | Example |
|-------|-------------|---------|
| `patch` | Increment patch version (fourth number) | `0.21.0.1` → `0.21.0.2` |
| `minor` | Increment minor version (third number) | `0.21.0.1` → `0.21.1.0` |
| `major` | Increment major version (first number) | `0.21.0.1` → `1.0.0.0` |
| `feature` | Increment feature version (third number) | `0.21.0.1` → `0.21.1.0` |
| `feature_set` | Increment feature set version (second number) | `0.21.0.1` → `0.22.0.0` |

#### Configuration Example

```yaml
config:
  version_file: ${config.version_file}  # Uses workflow config variable
  increment_type: patch                  # Increment patch version
```

#### Step Outputs

After execution, this step outputs:

```json
{
  "new_version": "0.21.0.2",
  "old_version": "0.21.0.1"
}
```

**Accessing outputs in other steps:**
- `step-1.output.new_version` - The new version number
- `step-1.output.old_version` - The previous version number

#### Example Usage in Other Steps

Step 2 (Create Detailed Changelog) accesses the version via:
```python
version = context.step_outputs.get("step-1", {}).get("output", {}).get("new_version")
```

#### Error Handling

- **Version file not found:** Step fails with error message
- **Invalid version format:** Step fails with error message
- **File write error:** Step fails with error message

---

### Step 2: Create Detailed Changelog

**Handler:** `release.changelog_create`
**Category:** Changelog
**Icon:** 📝
**Required:** ✅ Yes
**Default Dependencies:** `step-1` (needs new version)

#### Purpose

Creates a detailed changelog file with full timestamp, release summary, and detailed changes. This file is stored in the changelog archive directory.

#### Execution Flow

1. Retrieves new version from Step 1 output
2. Extracts release details from workflow parameters
3. Extracts Epic/Story information from Git branch name
4. Generates full timestamp (`YYYY-MM-DD HH:MM:SS UTC`)
5. Creates changelog file at `{changelog_dir}/CHANGELOG_v{version}.md`
6. Writes changelog content with all release information

#### Configuration Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `changelog_dir` | string | ❌ No | `KB/Changelog_and_Release_Notes/Changelog_Archive` | Directory where detailed changelog files are stored |
| `format` | enum | ❌ No | `full_timestamp` | Timestamp format for release date |

#### `format` Options

| Value | Description | Example |
|-------|-------------|---------|
| `full_timestamp` | Full timestamp with date and time | `2025-11-21 15:42:58 UTC` |

#### Configuration Example

```yaml
config:
  changelog_dir: ${config.changelog_dir}  # Uses workflow config variable
  format: full_timestamp                  # Use full timestamp format
```

#### Generated Changelog Format

The step generates a changelog file with the following format:

```markdown
# Changelog v0.21.0.2

**Release Date:** 2025-11-21 15:42:58 UTC
**Epic:** Epic 01 - Visual Workflow Management Platform
**Story:** Story 3 - Implement Workflows in VWMP
**Type:** 🐞 Fix

## Summary
🐞 Fix: VWMP designer renders Release Workflow

## Changes

See commit history for detailed changes.

## Related Tasks
- E01S3T001 – Implement Release Workflow in VWMP
```

#### Data Sources

The step uses data from multiple sources:

1. **Version:** From Step 1 output (`step-1.output.new_version`)
2. **Summary:** From workflow parameter `summary`
3. **Change Type:** From workflow parameter `change_type`
4. **Detailed Changes:** From workflow parameter `detailed_changes` (optional)
5. **Epic/Story:** Extracted from Git branch name (e.g., `epic/21-workflow-platform`)
6. **Timestamp:** Generated at execution time using `get_full_timestamp()`

#### Step Outputs

After execution, this step outputs:

```json
{
  "changelog_file": "KB/Changelog_and_Release_Notes/Changelog_Archive/CHANGELOG_v0.21.0.2.md",
  "version": "0.21.0.2"
}
```

#### Error Handling

- **Version not found:** Step fails if Step 1 output is not available
- **Directory creation failed:** Step fails if changelog directory cannot be created
- **File write error:** Step fails if changelog file cannot be written

---

### Step 3: Update Main Changelog

**Handler:** `release.changelog_update`
**Category:** Changelog
**Icon:** 📋
**Required:** ✅ Yes
**Default Dependencies:** `step-2` (needs changelog to be created first)

#### Purpose

Adds a summary entry to the main CHANGELOG.md file. This provides a quick reference for recent releases without needing to open individual detailed changelog files.

#### Execution Flow

1. Retrieves version from Step 1 output
2. Retrieves release details from workflow parameters
3. Generates summary date using `date_format`
4. Creates summary entry with link to detailed changelog
5. Inserts entry into main CHANGELOG.md after "## Recent Releases" header
6. If "## Recent Releases" header doesn't exist, creates it at the top

#### Configuration Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `main_changelog` | string | ❌ No | `CHANGELOG.md` | Path to main changelog file |
| `date_format` | string | ❌ No | `DD-MM-YY` | Date format for summary entries |

#### `date_format` Options

| Value | Description | Example |
|-------|-------------|---------|
| `DD-MM-YY` | Day-Month-Year (short) | `21-11-25` |
| `YYYY-MM-DD` | Year-Month-Day (ISO) | `2025-11-21` |
| Custom format | Any strftime format | `%d/%m/%Y` → `21/11/2025` |

#### Configuration Example

```yaml
config:
  main_changelog: ${config.main_changelog}  # Uses workflow config variable
  date_format: DD-MM-YY                     # Use short date format
```

#### Generated Summary Entry Format

The step adds an entry to CHANGELOG.md like this:

```markdown
### [0.21.0.2] - 21-11-25
🐞 Fix: VWMP designer renders Release Workflow
- See [CHANGELOG_v0.21.0.2.md](KB/Changelog_and_Release_Notes/Changelog_Archive/CHANGELOG_v0.21.0.2.md)
```

#### Data Sources

1. **Version:** From Step 1 output (`step-1.output.new_version`)
2. **Summary:** From workflow parameter `summary`
3. **Change Type:** From workflow parameter `change_type` (used for emoji)
4. **Date:** Generated at execution time using `get_summary_date()`

#### Step Outputs

After execution, this step outputs:

```json
{
  "main_changelog": "CHANGELOG.md",
  "version": "0.21.0.2"
}
```

#### Error Handling

- **Version not found:** Step fails if Step 1 output is not available
- **Main changelog not found:** Step fails if CHANGELOG.md doesn't exist
- **File write error:** Step fails if CHANGELOG.md cannot be written

---

### Step 4: Update README

**Handler:** `release.readme_update`
**Category:** Documentation
**Icon:** 📖
**Required:** ❌ No (optional step)
**Default Dependencies:** `step-1` (needs new version)

#### Purpose

Updates the README.md file with the new version badge and latest release information. This keeps the README current for users viewing the project.

#### Execution Flow

1. Retrieves version from Step 1 output
2. Retrieves release details from workflow parameters
3. Updates version badge (if `update_badge: true`)
4. Updates "Latest Release" callout (if `update_latest_release: true`)
5. Writes updated README.md

#### Configuration Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `readme_file` | string | ❌ No | `README.md` | Path to README.md file |
| `update_badge` | boolean | ❌ No | `true` | Whether to update version badge |
| `update_latest_release` | boolean | ❌ No | `true` | Whether to update latest release callout |

#### Configuration Example

```yaml
config:
  readme_file: README.md
  update_badge: true
  update_latest_release: true
```

#### Updates Performed

1. **Version Badge Update:**
   - Finds badge pattern: `[![Version]...badge/version-...-blue](.../releases)`
   - Replaces with: `[![Version](https://img.shields.io/badge/version-0.21.0.2-blue)](https://github.com/earlution/vwmp/releases)`

2. **Latest Release Update:**
   - Finds pattern: `**🎉 Latest Release: v...**`
   - Replaces with: `**🎉 Latest Release: v0.21.0.2** - 🐞 Fix: VWMP designer renders Release Workflow`

#### Data Sources

1. **Version:** From Step 1 output (`step-1.output.new_version`)
2. **Summary:** From workflow parameter `summary`
3. **Change Type:** From workflow parameter `change_type` (used for emoji)

#### Step Outputs

After execution, this step outputs:

```json
{
  "readme_file": "README.md",
  "version": "0.21.0.2"
}
```

#### Error Handling

- **Version not found:** Step fails if Step 1 output is not available
- **README.md not found:** Step is skipped (not failed) if README.md doesn't exist
- **File write error:** Step fails if README.md cannot be written

---

### Step 5: Auto-update Kanban Docs

**Handler:** `vwmp.kanban_update`
**Category:** Documentation
**Icon:** 📊
**Required:** ❌ No (optional step)
**Default Dependencies:** `step-1` (needs version for task updates)

#### Purpose

Automatically updates Kanban documentation (Epic and Story markdown files) with task completion status and release versions. This keeps project documentation synchronized with releases.

#### Execution Flow

1. Retrieves version from Step 1 output
2. Extracts Epic/Story information from Git branch name
3. Extracts task IDs from workflow parameters (if provided)
4. Runs `update_kanban_docs.py` script to update documentation:
   - Marks tasks as complete in Epic/Story docs
   - Updates story status if all tasks complete
   - Updates Epic status if all stories complete
   - Adds release version to completed tasks

#### Configuration Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `epic_doc_pattern` | string | ❌ No | `KB/PM_and_Portfolio/epics/overview/Epic {epic}/Epic-{epic}.md` | Pattern for locating Epic documentation files |
| `kanban_board` | string | ❌ No | `KB/PM_and_Portfolio/epics/overview/_index.md` | Path to main Kanban board index file |

#### Configuration Example

```yaml
config:
  epic_doc_pattern: KB/PM_and_Portfolio/epics/overview/Epic {epic}/Epic-{epic}.md
  kanban_board: KB/PM_and_Portfolio/epics/overview/_index.md
```

#### Pattern Variables

The `epic_doc_pattern` supports variable substitution:
- `{epic}` - Replaced with Epic number extracted from Git branch name

**Example:**
- Branch: `epic/21-workflow-platform`
- Epic extracted: `21`
- Pattern: `KB/PM_and_Portfolio/epics/overview/Epic {epic}/Epic-{epic}.md`
- Resolved: `KB/PM_and_Portfolio/epics/overview/Epic 01/Epic-01.md`

#### What Gets Updated

The script updates the following in Kanban docs:

1. **Task Checkboxes:**
   - Changes `- [ ] E01:S01:T001` to `- [x] E01:S01:T001 (v0.21.0.2)`
   - Marks tasks as complete with version number

2. **Story Status:**
   - Updates story status to `COMPLETE` if all tasks are marked complete
   - Adds completion date and version

3. **Epic Status:**
   - Updates Epic status to `COMPLETE` if all stories are marked complete
   - Adds completion date and version

#### Data Sources

1. **Version:** From Step 1 output (`step-1.output.new_version`)
2. **Epic/Story:** Extracted from Git branch name
3. **Task IDs:** Extracted from workflow parameters (if provided) or from release summary/changelog

#### Step Outputs

After execution, this step outputs:

```json
{
  "epic_doc": "KB/PM_and_Portfolio/epics/overview/Epic 01/Epic-01.md",
  "tasks_updated": 1,
  "version": "0.21.0.2"
}
```

#### Error Handling

- **Version not found:** Step fails if Step 1 output is not available
- **Epic/Story extraction failed:** Step fails if branch name cannot be parsed
- **Script execution failed:** Step fails if `update_kanban_docs.py` returns non-zero exit code

---

### Step 6: Stage Files

**Handler:** `git.stage_all`
**Category:** Git
**Icon:** 📦
**Required:** ✅ Yes
**Default Dependencies:** `step-1`, `step-2`, `step-3`, `step-4`, `step-5` (wait for all file changes)

#### Purpose

Stages all changed files using `git add`. This prepares files for committing after all documentation updates are complete.

#### Execution Flow

1. Collects file paths from `paths` configuration
2. If `paths` contains `"*"`, executes `git add -A` (stage all files)
3. Otherwise, executes `git add <paths>` for each specified path
4. Verifies files were staged successfully

#### Configuration Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `paths` | array | ❌ No | `["*"]` | Array of file paths to stage. Use `["*"]` for all files |

#### Configuration Example

```yaml
config:
  paths:
    - "*"  # Stage all files (equivalent to git add -A)
```

**Alternative: Stage specific files:**
```yaml
config:
  paths:
    - "README.md"
    - "CHANGELOG.md"
    - "VERSION"
```

#### Why Multiple Dependencies?

This step depends on **all previous file-modifying steps** to ensure:
- Version file is updated (Step 1)
- Detailed changelog is created (Step 2)
- Main changelog is updated (Step 3)
- README is updated (Step 4)
- Kanban docs are updated (Step 5)

**All changes are staged together** before committing, ensuring atomic commits.

#### Step Outputs

After execution, this step outputs:

```json
{
  "paths_staged": ["*"]
}
```

#### Error Handling

- **Git command failed:** Step fails if `git add` returns non-zero exit code
- **File not found:** Step fails if a specified path doesn't exist (only if using specific paths)

---

### Step 7: Run Validators

**Handler:** `vwmp.run_validators`
**Category:** Validation
**Icon:** ✅
**Required:** ✅ Yes
**Default Dependencies:** `step-6` (run validators after files are staged)

#### Purpose

Executes validation scripts to ensure the release meets quality standards. This catches issues before committing.

#### Execution Flow

1. Iterates through each validator script in `validators` array
2. Executes each validator script with Python
3. If `strict_mode: true`, passes `--strict` flag to validators
4. Collects output and error messages from each validator
5. If any validator fails, the step fails and stops execution
6. If all validators pass, the step succeeds

#### Configuration Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `validators` | array | ✅ Yes | - | Array of validator script paths (relative to workspace root) |
| `strict_mode` | boolean | ❌ No | `true` | Whether to run validators in strict mode |

#### Configuration Example

```yaml
config:
  validators:
    - scripts/validation/validate_branch_context.py
    - scripts/validation/validate_changelog_format.py
  strict_mode: true
```

#### What Do These Validators Check?

1. **`validate_branch_context.py`:**
   - Current Git branch matches version schema (e.g., Epic 01 = `0.21.x.x`)
   - CHANGELOG entries match branch (no cross-epic contamination)
   - Version file matches branch schema
   - No forbidden files modified for this branch

2. **`validate_changelog_format.py`:**
   - Detailed changelog files have full timestamp format (`YYYY-MM-DD HH:MM:SS UTC`)
   - Required fields are present (Release Date, Epic, Story, Type, Summary)
   - Changelog format is valid

#### Strict Mode

When `strict_mode: true`:
- Validators receive `--strict` flag
- Any warning or error causes validator to fail
- Workflow stops if any validator fails

When `strict_mode: false`:
- Validators may issue warnings without failing
- Workflow continues even with warnings (use with caution)

#### Step Outputs

After execution, this step outputs:

```json
{
  "validators_run": 2
}
```

#### Error Handling

- **Validator script not found:** Step fails if a validator script doesn't exist
- **Validator execution failed:** Step fails if a validator returns non-zero exit code
- **Strict mode violation:** Step fails if any validator issues an error in strict mode

#### Example Validator Output

If validators pass:
```
✅ scripts/validation/validate_branch_context.py passed
✅ scripts/validation/validate_changelog_format.py passed
```

If a validator fails:
```
❌ scripts/validation/validate_branch_context.py failed
Error: Version 0.21.0.2 does not match branch schema for epic/21-workflow-platform
Expected: 0.21.x.x
Actual: 0.21.0.2
```

---

### Step 8: Commit Changes

**Handler:** `git.commit`
**Category:** Git
**Icon:** 💾
**Required:** ✅ Yes
**Default Dependencies:** `step-7` (commit only after validators pass)

#### Purpose

Creates a git commit with the version and summary in the commit message. This permanently records the release changes.

#### Execution Flow

1. Retrieves version from Step 1 output
2. Retrieves summary from workflow parameters
3. Builds commit message from `message_template`
4. Executes `git commit -m "<message>"`
5. Extracts commit hash from output (if available)

#### Configuration Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `message_template` | string | ❌ No | `"{version} - {summary}"` | Commit message template |

#### Message Template Variables

The `message_template` supports variable substitution:
- `{version}` - Replaced with new version from Step 1 output
- `{summary}` - Replaced with release summary from workflow parameters

#### Configuration Example

```yaml
config:
  message_template: "{version} - {summary}"
```

**Example commit message:**
```
0.21.0.2 - 🐞 Fix: VWMP designer renders Release Workflow
```

**Custom template example:**
```yaml
config:
  message_template: "Release v{version}: {summary}"
```
**Result:**
```
Release v0.21.0.2: 🐞 Fix: VWMP designer renders Release Workflow
```

#### Data Sources

1. **Version:** From Step 1 output (`step-1.output.new_version`)
2. **Summary:** From workflow parameter `summary`

#### Step Outputs

After execution, this step outputs:

```json
{
  "commit_message": "0.21.0.2 - 🐞 Fix: VWMP designer renders Release Workflow",
  "commit_hash": "ac28a1e"
}
```

#### Error Handling

- **Version not found:** Step fails if Step 1 output is not available
- **Git commit failed:** Step fails if `git commit` returns non-zero exit code
- **No changes to commit:** Step fails if there are no staged changes

---

### Step 9: Create Git Tag

**Handler:** `git.create_tag`
**Category:** Git
**Icon:** 🏷️
**Required:** ❌ No (optional step)
**Default Dependencies:** `step-8` (tag after commit)

#### Purpose

Creates an annotated git tag for the release. This provides a permanent reference point for the release version.

#### Execution Flow

1. Retrieves version from Step 1 output
2. Retrieves summary from workflow parameters
3. Builds tag name from `tag_template`
4. Builds tag message from `message_template`
5. Creates annotated tag (if `annotated: true`) or lightweight tag
6. Verifies tag was created successfully

#### Configuration Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `tag_template` | string | ❌ No | `"v{version}"` | Tag name template |
| `message_template` | string | ❌ No | `"Release {tag}: {summary}"` | Tag message template |
| `annotated` | boolean | ❌ No | `true` | Whether to create an annotated tag |

#### Template Variables

**`tag_template` supports:**
- `{version}` - Replaced with new version from Step 1 output

**`message_template` supports:**
- `{tag}` - Replaced with tag name (e.g., `v0.21.0.2`)
- `{summary}` - Replaced with release summary from workflow parameters

#### Configuration Example

```yaml
config:
  tag_template: v{version}
  message_template: "Release {tag}: {summary}"
  annotated: true
```

**Example tag:**
- **Tag name:** `v0.21.0.2`
- **Tag message:** `Release v0.21.0.2: 🐞 Fix: VWMP designer renders Release Workflow`

#### Annotated vs Lightweight Tags

**Annotated tags** (`annotated: true`):
- Store additional metadata (author, date, message)
- Recommended for releases
- Can be signed with GPG
- Created with `git tag -a -m "<message>"`

**Lightweight tags** (`annotated: false`):
- Simple pointer to a commit
- No metadata
- Created with `git tag`

#### Data Sources

1. **Version:** From Step 1 output (`step-1.output.new_version`)
2. **Summary:** From workflow parameter `summary`

#### Step Outputs

After execution, this step outputs:

```json
{
  "tag_name": "v0.21.0.2",
  "tag_message": "Release v0.21.0.2: 🐞 Fix: VWMP designer renders Release Workflow"
}
```

#### Error Handling

- **Version not found:** Step fails if Step 1 output is not available
- **Git tag failed:** Step fails if `git tag` returns non-zero exit code
- **Tag already exists:** Step fails if tag with same name already exists

---

### Step 10: Push to Remote

**Handler:** `git.push`
**Category:** Git
**Icon:** 📤
**Required:** ❌ No (optional step)
**Default Dependencies:** `step-8`, `step-9` (push after commit and tag)

#### Purpose

Pushes the branch and tags to the remote repository. This makes the release available to others.

#### Execution Flow

1. Retrieves current Git branch name
2. Pushes branch to remote using `git push <remote> <branch>`
3. If `push_tags: true`, pushes tags using `git push <remote> --tags`
4. Verifies push was successful

#### Configuration Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `push_tags` | boolean | ❌ No | `true` | Whether to push tags along with branch |
| `remote` | string | ❌ No | `origin` | Git remote name |

#### Configuration Example

```yaml
config:
  push_tags: true
  remote: origin
```

**Executed commands:**
```bash
git push origin epic/21-workflow-platform  # Push branch
git push origin --tags                      # Push tags (if push_tags: true)
```

#### Why Two Dependencies?

This step depends on both Step 8 (commit) and Step 9 (tag) because:
- **Step 8 dependency:** Ensures commit exists before pushing branch
- **Step 9 dependency:** Ensures tag exists before pushing tags (if `push_tags: true`)

**Note:** If Step 9 is skipped (tag not created), Step 10 will still execute (pushing branch only, skipping tag push).

#### Step Outputs

After execution, this step outputs:

```json
{
  "remote": "origin",
  "branch": "epic/21-workflow-platform",
  "tags_pushed": ["v0.21.0.2"]
}
```

#### Error Handling

- **Git push failed:** Step fails if `git push` returns non-zero exit code
- **Remote not accessible:** Step fails if remote repository is not accessible
- **Authentication failed:** Step fails if Git credentials are invalid

---

## 📋 Workflow Parameters

The Release Workflow accepts the following parameters when executing:

### Parameter 1: Summary

**Type:** `string`
**Required:** ✅ Yes
**Label:** Release Summary
**Description:** Brief description of the release

**Example:**
```
🐞 Fix: VWMP designer renders Release Workflow
```

---

### Parameter 2: Change Type

**Type:** `enum`
**Required:** ✅ Yes
**Default:** `tooling`
**Label:** Release Type
**Description:** Type of release

**Options:**
| Value | Label | Emoji |
|-------|-------|-------|
| `feature` | Feature 🚀 | 🚀 |
| `fix` | Fix 🐞 | 🐞 |
| `infrastructure` | Infrastructure 🔧 | 🔧 |
| `security` | Security 🔒 | 🔒 |
| `cleanup` | Cleanup 🧹 | 🧹 |
| `refactor` | Refactor 🔄 | 🔄 |
| `documentation` | Documentation 📚 | 📚 |
| `tooling` | Tooling 🧰 | 🧰 |
| `maintenance` | Maintenance 🚧 | 🚧 |

**Usage:**
- Used in changelog entries for categorization
- Used in commit messages and tags
- Determines emoji in changelog and README updates

---

### Parameter 3: Detailed Changes

**Type:** `textarea`
**Required:** ❌ No
**Label:** Detailed Changes
**Description:** Optional detailed list of changes

**Example:**
```markdown
- Fixed React Flow component access from CDN
- Added server-side workflow data loading
- Converted WorkflowParameter objects to dictionaries for JSON serialization
- Fixed node labels to use strings instead of JSX objects
```

**Usage:**
- Included in detailed changelog file
- Provides context for users reviewing the release

---

### Parameter 4: Skip README

**Type:** `boolean`
**Required:** ❌ No
**Default:** `false`
**Label:** Skip README Update

**Usage:**
- If `true`, Step 4 (Update README) is skipped
- Useful if README.md is manually maintained or not needed

---

### Parameter 5: KB Docs OK

**Type:** `boolean`
**Required:** ✅ Yes
**Default:** `false`
**Label:** KB Documentation Updated
**Description:** Confirm all required KB documentation is updated

**Usage:**
- Safety check to ensure documentation is up to date before releasing
- Must be explicitly set to `true` to proceed
- Prevents accidental releases with outdated documentation

---

## 🔄 Execution Flow

### Dependency Graph

```
Step 1 (Bump Version)
  ├─→ Step 2 (Create Detailed Changelog)
  │     └─→ Step 3 (Update Main Changelog)
  ├─→ Step 4 (Update README)
  ├─→ Step 5 (Auto-update Kanban Docs)
  └─→ Step 6 (Stage Files) ← depends on 1,2,3,4,5
        └─→ Step 7 (Run Validators)
              └─→ Step 8 (Commit Changes)
                    ├─→ Step 9 (Create Git Tag)
                    └─→ Step 10 (Push to Remote) ← depends on 8,9
```

### Execution Phases

**Phase 1: Version & Documentation Updates** (Steps 1-5)
- Steps 2, 4, 5 run in parallel after Step 1 completes
- Step 3 runs after Step 2 completes

**Phase 2: Git Operations & Validation** (Steps 6-10)
- Step 6 waits for all Phase 1 steps to complete
- Steps 7, 8, 9, 10 run sequentially
- Step 10 waits for both Step 8 and Step 9

### Parallel Execution

Steps that can run in parallel (after dependencies are met):
- **Steps 2, 4, 5:** All depend on Step 1, but don't depend on each other
- **Steps 9, 10:** Step 10 depends on Step 8 (which doesn't depend on Step 9)

---

## 📚 Related Documentation

- **[Tutorial: Building Release Workflow from Scratch](tutorial-release-workflow.md)** - Step-by-step tutorial
- **[Release Workflow Usage](release-workflow-usage.md)** - How to use the Release Workflow
- **[Visual Designer Guide](visual-designer-guide.md)** - Using the visual designer
- **[Workflow Execution Guide](execution-guide.md)** - Running and monitoring workflows

---

**Last Updated:** 2025-11-21
**Reference Version:** 1.0.0
