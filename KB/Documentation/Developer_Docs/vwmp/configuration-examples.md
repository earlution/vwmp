# VWMP Configuration Examples

**Version:** 1.0.0
**Last Updated:** 2025-11-21
**Related:** Epic 01, Story 4 - Comprehensive VWMP Documentation

---

## 📖 Overview

This document provides real-world configuration examples for all workflow steps in VWMP. Use these examples as templates when configuring your own workflows.

**Who This Guide Is For:**
- Workflow designers configuring steps
- Developers building custom workflows
- Users customizing existing workflows

---

## 🔧 Workflow-Level Configuration

### Basic Workflow Configuration

**Example: Minimal Workflow**
```yaml
name: My Workflow
version: 1.0.0
type: custom
description: A simple custom workflow

config:
  # Define shared configuration variables
  workspace_path: /path/to/workspace

steps:
  # Steps will reference config variables
```

**Example: Release Workflow Configuration**
```yaml
name: Release Workflow
version: 1.0.0
type: release
description: Automated release workflow

config:
  version_file: VERSION
  changelog_dir: KB/Changelog_and_Release_Notes/Changelog_Archive
  main_changelog: CHANGELOG.md
```

**Example: Kanban Review Workflow Configuration**
```yaml
name: Kanban Review Workflow
version: 1.0.0
type: kanban_review
description: Automated Kanban documentation review

config:
  kanban_root: KB/PM_and_Portfolio
  epic_template: KB/Documentation/Document_Templates/Kanban/EPIC_TEMPLATE.md
  story_template: KB/Documentation/Document_Templates/Kanban/STORY_TEMPLATE.md
  bloat_threshold: 500
  min_text_length: 20
```

---

## 📝 Release Workflow Step Examples

### Step 1: Bump Version

**Basic Configuration:**
```yaml
- id: step-1
  name: Bump Version
  type: version_bump
  handler: release.version_bump
  required: true
  dependencies: []
  config:
    increment_type: patch  # patch, minor, major, build
    version_file: ${config.version_file}
```

**Advanced Configuration:**
```yaml
- id: step-1
  name: Bump Version
  handler: release.version_bump
  config:
    increment_type: minor  # Increment minor version
    version_file: VERSION
    reset_build: true  # Reset build metadata when incrementing patch
```

**Configuration Options:**
- `increment_type`: `patch`, `minor`, `major`, or `build`
- `version_file`: Path to version file (required)
- `reset_build`: Reset build metadata when incrementing (optional, default: `true`)

---

### Step 2: Create Detailed Changelog

**Basic Configuration:**
```yaml
- id: step-2
  name: Create Detailed Changelog
  type: changelog_create
  handler: release.changelog_create
  required: true
  dependencies:
    - step-1
  config:
    changelog_dir: ${config.changelog_dir}
    format: full_timestamp
```

**Advanced Configuration:**
```yaml
- id: step-2
  name: Create Detailed Changelog
  handler: release.changelog_create
  config:
    changelog_dir: KB/Changelog_and_Release_Notes/Changelog_Archive
    format: full_timestamp  # full_timestamp or date_only
    include_summary: true
    include_tasks: true
```

**Configuration Options:**
- `changelog_dir`: Directory for changelog files (required)
- `format`: `full_timestamp` or `date_only` (default: `full_timestamp`)
- `include_summary`: Include release summary (optional, default: `true`)
- `include_tasks`: Include related tasks (optional, default: `true`)

---

### Step 3: Update Main Changelog

**Basic Configuration:**
```yaml
- id: step-3
  name: Update Main Changelog
  type: changelog_update
  handler: release.changelog_update
  required: true
  dependencies:
    - step-2
  config:
    main_changelog: ${config.main_changelog}
    date_format: DD-MM-YY
```

**Advanced Configuration:**
```yaml
- id: step-3
  name: Update Main Changelog
  handler: release.changelog_update
  config:
    main_changelog: CHANGELOG.md
    date_format: DD-MM-YY  # Date format for summary entries
    prepend: true  # Add to top of changelog (default: true)
    link_to_detailed: true  # Link to detailed changelog
```

**Configuration Options:**
- `main_changelog`: Path to main changelog file (required)
- `date_format`: Date format string (default: `DD-MM-YY`)
- `prepend`: Add to top of changelog (optional, default: `true`)
- `link_to_detailed`: Link to detailed changelog (optional, default: `true`)

---

### Step 4: Update README

**Basic Configuration:**
```yaml
- id: step-4
  name: Update README
  type: readme_update
  handler: release.readme_update
  required: false
  enabled: true
  dependencies:
    - step-1
  config:
    readme_file: README.md
    update_badge: true
    update_latest_release: true
```

**Advanced Configuration:**
```yaml
- id: step-4
  name: Update README
  handler: release.readme_update
  config:
    readme_file: README.md
    update_badge: true  # Update version badge
    update_latest_release: true  # Update latest release callout
    badge_format: "version-{version}-blue"  # Custom badge format
```

**Configuration Options:**
- `readme_file`: Path to README file (required)
- `update_badge`: Update version badge (optional, default: `true`)
- `update_latest_release`: Update latest release callout (optional, default: `true`)
- `badge_format`: Custom badge format string (optional)

---

### Step 5: Auto-update Kanban Docs

**Basic Configuration:**
```yaml
- id: step-5
  name: Auto-update Kanban Docs
  type: kanban_update
  handler: vwmp.kanban_update
  required: false
  automatic: true
  dependencies:
    - step-1
  config:
    epic_doc_pattern: KB/PM_and_Portfolio/epics/overview/Epic {epic}/Epic-{epic}.md
    kanban_board: KB/PM_and_Portfolio/epics/overview/_index.md
```

**Advanced Configuration:**
```yaml
- id: step-5
  name: Auto-update Kanban Docs
  handler: vwmp.kanban_update
  config:
    epic_doc_pattern: KB/PM_and_Portfolio/epics/overview/Epic {epic}/Epic-{epic}.md
    kanban_board: KB/PM_and_Portfolio/epics/overview/_index.md
    update_stories: true  # Update story docs too
    update_last_updated: true  # Update last updated timestamp
```

**Configuration Options:**
- `epic_doc_pattern`: Pattern for Epic doc paths (required)
- `kanban_board`: Path to Kanban board file (required)
- `update_stories`: Update story documents (optional, default: `true`)
- `update_last_updated`: Update last updated timestamp (optional, default: `true`)

---

### Step 6: Stage Files

**Basic Configuration:**
```yaml
- id: step-6
  name: Stage Files
  type: git_stage
  handler: git.stage_all
  required: true
  dependencies:
    - step-1
    - step-2
    - step-3
    - step-4
    - step-5
  config:
    paths:
      - "*"
```

**Advanced Configuration:**
```yaml
- id: step-6
  name: Stage Files
  handler: git.stage_all
  config:
    paths:
      - "*"  # Stage all files
    exclude:
      - "*.pyc"
      - ".env"
      - "venv/"
    force: false  # Force add ignored files (default: false)
```

**Configuration Options:**
- `paths`: List of paths to stage (required, use `["*"]` for all)
- `exclude`: List of paths to exclude (optional)
- `force`: Force add ignored files (optional, default: `false`)

---

### Step 7: Run Validators

**Basic Configuration:**
```yaml
- id: step-7
  name: Run Validators
  type: validation
  handler: vwmp.run_validators
  required: true
  dependencies:
    - step-6
  config:
    validators:
      - scripts/validation/validate_branch_context.py
      - scripts/validation/validate_changelog_format.py
    strict_mode: true
```

**Advanced Configuration:**
```yaml
- id: step-7
  name: Run Validators
  handler: vwmp.run_validators
  config:
    validators:
      - scripts/validation/validate_branch_context.py
      - scripts/validation/validate_changelog_format.py
      - scripts/validation/validate_custom.py  # Add custom validators
    strict_mode: true  # Fail on validation errors
    continue_on_error: false  # Stop on first error (default: false)
```

**Configuration Options:**
- `validators`: List of validator script paths (required)
- `strict_mode`: Fail on validation errors (optional, default: `true`)
- `continue_on_error`: Continue after errors (optional, default: `false`)

---

### Step 8: Commit Changes

**Basic Configuration:**
```yaml
- id: step-8
  name: Commit Changes
  type: git_commit
  handler: git.commit
  required: true
  dependencies:
    - step-7
  config:
    message_template: "{version} - {summary}"
```

**Advanced Configuration:**
```yaml
- id: step-8
  name: Commit Changes
  handler: git.commit
  config:
    message_template: "{version} - {summary}"
    include_changes: true  # Include list of changed files
    sign_commit: false  # Sign commit with GPG (default: false)
```

**Configuration Options:**
- `message_template`: Commit message template (required)
  - `{version}` - Current version
  - `{summary}` - Release summary
  - `{type}` - Change type
- `include_changes`: Include changed files list (optional, default: `true`)
- `sign_commit`: Sign commit with GPG (optional, default: `false`)

---

### Step 9: Create Git Tag

**Basic Configuration:**
```yaml
- id: step-9
  name: Create Git Tag
  type: git_tag
  handler: git.create_tag
  required: false
  enabled: true
  dependencies:
    - step-8
  config:
    tag_template: v{version}
    message_template: "Release {tag}: {summary}"
    annotated: true
```

**Advanced Configuration:**
```yaml
- id: step-9
  name: Create Git Tag
  handler: git.create_tag
  config:
    tag_template: v{version}  # v0.21.0.41
    message_template: "Release {tag}: {summary}"
    annotated: true  # Create annotated tag
    sign_tag: false  # Sign tag with GPG (default: false)
    force: false  # Overwrite existing tag (default: false)
```

**Configuration Options:**
- `tag_template`: Tag name template (required)
  - `{version}` - Current version
- `message_template`: Tag message template (required)
  - `{tag}` - Tag name
  - `{summary}` - Release summary
- `annotated`: Create annotated tag (optional, default: `true`)
- `sign_tag`: Sign tag with GPG (optional, default: `false`)
- `force`: Overwrite existing tag (optional, default: `false`)

---

### Step 10: Push to Remote

**Basic Configuration:**
```yaml
- id: step-10
  name: Push to Remote
  type: git_push
  handler: git.push
  required: false
  enabled: true
  dependencies:
    - step-8
    - step-9
  config:
    push_tags: true
    remote: origin
```

**Advanced Configuration:**
```yaml
- id: step-10
  name: Push to Remote
  handler: git.push
  config:
    remote: origin  # Remote name
    push_tags: true  # Push tags along with commits
    branch: null  # Current branch (default: current branch)
    force: false  # Force push (default: false)
    upstream: true  # Set upstream if not set (default: true)
```

**Configuration Options:**
- `remote`: Remote repository name (required, default: `origin`)
- `push_tags`: Push tags along with commits (optional, default: `true`)
- `branch`: Branch to push (optional, default: current branch)
- `force`: Force push (optional, default: `false`)
- `upstream`: Set upstream if not set (optional, default: `true`)

---

## 📊 Kanban Review Workflow Step Examples

### Step 1: Review Kanban State

**Basic Configuration:**
```yaml
- id: step-1
  name: Review Kanban State
  type: kanban_review_state
  handler: kanban.review_state
  required: true
  dependencies: []
  config:
    kanban_root: ${config.kanban_root}
```

**Advanced Configuration:**
```yaml
- id: step-1
  name: Review Kanban State
  handler: kanban.review_state
  config:
    kanban_root: KB/PM_and_Portfolio
    include_metrics: true  # Include document metrics (default: true)
    include_stories: true  # Include story documents (default: true)
```

**Configuration Options:**
- `kanban_root`: Path to Kanban documentation root (required)
- `include_metrics`: Include document metrics (optional, default: `true`)
- `include_stories`: Include story documents (optional, default: `true`)

---

### Step 2: Check Template Divergence

**Basic Configuration:**
```yaml
- id: step-2
  name: Check Template Divergence
  type: kanban_check_template_divergence
  handler: kanban.check_template_divergence
  required: true
  dependencies:
    - step-1
  config:
    kanban_root: ${config.kanban_root}
    epic_template: ${config.epic_template}
    story_template: ${config.story_template}
```

**Advanced Configuration:**
```yaml
- id: step-2
  name: Check Template Divergence
  handler: kanban.check_template_divergence
  config:
    kanban_root: KB/PM_and_Portfolio
    epic_template: KB/Documentation/Document_Templates/Kanban/EPIC_TEMPLATE.md
    story_template: KB/Documentation/Document_Templates/Kanban/STORY_TEMPLATE.md
    strict_mode: true  # Require exact template match (default: false)
    ignore_optional: false  # Ignore optional sections (default: false)
```

**Configuration Options:**
- `kanban_root`: Path to Kanban documentation root (required)
- `epic_template`: Path to Epic template (required)
- `story_template`: Path to Story template (required)
- `strict_mode`: Require exact template match (optional, default: `false`)
- `ignore_optional`: Ignore optional sections (optional, default: `false`)

---

### Step 3: Check Documentation Bloat

**Basic Configuration:**
```yaml
- id: step-3
  name: Check Documentation Bloat
  type: kanban_check_bloat
  handler: kanban.check_bloat
  required: true
  dependencies:
    - step-1
  config:
    kanban_root: ${config.kanban_root}
    bloat_threshold: ${config.bloat_threshold}
```

**Advanced Configuration:**
```yaml
- id: step-3
  name: Check Documentation Bloat
  handler: kanban.check_bloat
  config:
    kanban_root: KB/PM_and_Portfolio
    bloat_threshold: 500  # Maximum recommended lines
    check_word_count: true  # Also check word count (default: true)
    warn_threshold: 400  # Warning threshold (default: 80% of bloat_threshold)
```

**Configuration Options:**
- `kanban_root`: Path to Kanban documentation root (required)
- `bloat_threshold`: Maximum recommended lines (required, default: `500`)
- `check_word_count`: Also check word count (optional, default: `true`)
- `warn_threshold`: Warning threshold (optional, default: `80%` of `bloat_threshold`)

---

### Step 4: Check Repetition/Contradictions

**Basic Configuration:**
```yaml
- id: step-4
  name: Check Repetition/Contradictions
  type: kanban_check_repetition
  handler: kanban.check_repetition
  required: true
  dependencies:
    - step-1
  config:
    kanban_root: ${config.kanban_root}
    min_text_length: ${config.min_text_length}
```

**Advanced Configuration:**
```yaml
- id: step-4
  name: Check Repetition/Contradictions
  handler: kanban.check_repetition
  config:
    kanban_root: KB/PM_and_Portfolio
    min_text_length: 20  # Minimum text length to consider
    check_contradictions: true  # Check for contradictions (default: true)
    similarity_threshold: 0.9  # Similarity threshold for repetitions (default: 0.9)
```

**Configuration Options:**
- `kanban_root`: Path to Kanban documentation root (required)
- `min_text_length`: Minimum text length to consider (required, default: `20`)
- `check_contradictions`: Check for contradictions (optional, default: `true`)
- `similarity_threshold`: Similarity threshold for repetitions (optional, default: `0.9`)

---

### Step 5: Check Coverage Gaps

**Basic Configuration:**
```yaml
- id: step-5
  name: Check Coverage Gaps
  type: kanban_check_coverage_gaps
  handler: kanban.check_coverage_gaps
  required: true
  dependencies:
    - step-1
    - step-2
  config:
    kanban_root: ${config.kanban_root}
    epic_template: ${config.epic_template}
    story_template: ${config.story_template}
```

**Advanced Configuration:**
```yaml
- id: step-5
  name: Check Coverage Gaps
  handler: kanban.check_coverage_gaps
  config:
    kanban_root: KB/PM_and_Portfolio
    epic_template: KB/Documentation/Document_Templates/Kanban/EPIC_TEMPLATE.md
    story_template: KB/Documentation/Document_Templates/Kanban/STORY_TEMPLATE.md
    check_vague_content: true  # Check for vague content (default: true)
    min_section_length: 50  # Minimum characters for section (default: 50)
```

**Configuration Options:**
- `kanban_root`: Path to Kanban documentation root (required)
- `epic_template`: Path to Epic template (required)
- `story_template`: Path to Story template (required)
- `check_vague_content`: Check for vague content (optional, default: `true`)
- `min_section_length`: Minimum characters for section (optional, default: `50`)

---

### Step 6: Apply Documented Updates

**Basic Configuration:**
```yaml
- id: step-6
  name: Apply Documented Updates
  type: kanban_apply_updates
  handler: kanban.apply_updates
  required: false
  enabled: true
  dependencies:
    - step-2
    - step-3
    - step-4
    - step-5
  config:
    auto_apply: false
```

**Advanced Configuration:**
```yaml
- id: step-6
  name: Apply Documented Updates
  handler: kanban.apply_updates
  config:
    auto_apply: false  # Automatically apply fixes (use with caution)
    backup_before_apply: true  # Create backup before applying (default: true)
    dry_run: false  # Show what would be changed without applying (default: false)
```

**Configuration Options:**
- `auto_apply`: Automatically apply fixes (required, default: `false`)
  - ⚠️ **Warning:** Use with caution - will modify files
- `backup_before_apply`: Create backup before applying (optional, default: `true`)
- `dry_run`: Show what would be changed without applying (optional, default: `false`)

---

## 🔄 Using Configuration Variables

### Referencing Workflow Config

**In Step Configuration:**
```yaml
config:
  version_file: VERSION

steps:
  - name: Bump Version
    config:
      version_file: ${config.version_file}  # ✅ Reference config
```

**Multiple Config Variables:**
```yaml
config:
  version_file: VERSION
  changelog_dir: KB/Changelog_and_Release_Notes/Changelog_Archive
  main_changelog: CHANGELOG.md

steps:
  - name: Create Changelog
    config:
      changelog_dir: ${config.changelog_dir}
      main_changelog: ${config.main_changelog}
```

### Referencing Parameters

**In Step Configuration:**
```yaml
parameters:
  - name: auto_apply
    type: boolean
    default: false

steps:
  - name: Apply Updates
    config:
      auto_apply: ${parameters.auto_apply}  # ✅ Reference parameter
```

**Multiple Parameters:**
```yaml
parameters:
  - name: summary
    type: string
  - name: change_type
    type: enum

steps:
  - name: Commit Changes
    config:
      message_template: "{version} - {summary}"
      # Summary comes from parameters.summary
```

---

## 📋 Complete Workflow Examples

### Example 1: Simple Release Workflow

```yaml
name: Simple Release
version: 1.0.0
type: release
description: Simplified release workflow

config:
  version_file: VERSION
  changelog_dir: KB/Changelog_and_Release_Notes/Changelog_Archive

steps:
  - id: step-1
    name: Bump Version
    handler: release.version_bump
    required: true
    dependencies: []
    config:
      increment_type: patch
      version_file: ${config.version_file}

  - id: step-2
    name: Create Changelog
    handler: release.changelog_create
    required: true
    dependencies: [step-1]
    config:
      changelog_dir: ${config.changelog_dir}

  - id: step-3
    name: Commit and Push
    handler: git.commit
    required: true
    dependencies: [step-2]
    config:
      message_template: "{version} - Release"

parameters:
  - name: summary
    type: string
    required: true
```

### Example 2: Custom Kanban Review

```yaml
name: Custom Kanban Review
version: 1.0.0
type: kanban_review
description: Customized Kanban review workflow

config:
  kanban_root: KB/PM_and_Portfolio
  epic_template: KB/Documentation/Document_Templates/Kanban/EPIC_TEMPLATE.md
  bloat_threshold: 600  # Higher threshold
  min_text_length: 25   # Longer minimum

steps:
  - id: step-1
    name: Review State
    handler: kanban.review_state
    required: true
    dependencies: []
    config:
      kanban_root: ${config.kanban_root}

  - id: step-2
    name: Check Bloat
    handler: kanban.check_bloat
    required: true
    dependencies: [step-1]
    config:
      kanban_root: ${config.kanban_root}
      bloat_threshold: ${config.bloat_threshold}

parameters:
  - name: auto_apply
    type: boolean
    required: false
    default: false
```

---

## 🎯 Best Practices

### Configuration Tips

1. **Use Config Variables:**
   - ✅ Define shared values in workflow config
   - ✅ Reference with `${config.variable_name}`
   - ✅ Update in one place affects all steps

2. **Use Parameters:**
   - ✅ Define user-provided values as parameters
   - ✅ Reference with `${parameters.param_name}`
   - ✅ Provide sensible defaults

3. **Type Safety:**
   - ✅ Use correct types (integer, boolean, string)
   - ✅ Convert types in handlers if needed
   - ✅ Validate parameters before execution

4. **Documentation:**
   - ✅ Document config variables in workflow description
   - ✅ Document parameters with examples
   - ✅ Add inline comments for complex config

---

## 🔗 Related Documentation

- **[VWMP User Guide](vwmp-user-guide.md)** - How to use VWMP
- **[Release Workflow Reference](release-workflow-reference.md)** - Detailed step reference
- **[Kanban Review Workflow Reference](kanban-review-workflow-reference.md)** - Detailed step reference
- **[Best Practices Guide](best-practices.md)** - Design patterns and recommendations
- **[Troubleshooting Guide](troubleshooting.md)** - Common issues and solutions

---

**Last Updated:** 2025-11-21
**Guide Version:** 1.0.0

