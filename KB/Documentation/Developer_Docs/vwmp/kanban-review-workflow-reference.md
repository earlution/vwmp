# Kanban Review Workflow Reference

**Version:** 1.0.0
**Last Updated:** 2025-11-21
**Related:** Epic 01, Story 4 - Comprehensive VWMP Documentation

---

## 📖 Overview

This document provides a comprehensive reference for the **Kanban Review Workflow**, explaining each step, its parameters, configuration options, and how they work together. Use this as a detailed reference after completing the [Tutorial: Building Kanban Review Workflow from Scratch](tutorial-kanban-review-workflow.md).

---

## 📋 Workflow Structure

The Kanban Review Workflow consists of **6 steps** organized into phases:

### Phase 1: State Analysis
- **Step 1:** Review Kanban State

### Phase 2: Parallel Analysis
- **Step 2:** Check Template Divergence
- **Step 3:** Check Documentation Bloat
- **Step 4:** Check Repetition/Contradictions

### Phase 3: Coverage Analysis
- **Step 5:** Check Coverage Gaps

### Phase 4: Fix Application (Optional)
- **Step 6:** Apply Documented Updates

---

## 🔧 Workflow-Level Configuration

Before diving into individual steps, let's understand the workflow-level configuration that applies to all steps.

### Configuration Variables

These variables are defined at the workflow level and can be referenced in any step:

```yaml
config:
  kanban_root: KB/PM_and_Portfolio
  epic_template: KB/Documentation/Document_Templates/Kanban/EPIC_TEMPLATE.md
  story_template: KB/Documentation/Document_Templates/Kanban/STORY_TEMPLATE.md
  bloat_threshold: 500
  min_text_length: 20
```

**Usage in steps:**
- Reference these variables using `${config.variable_name}`
- Example: `${config.kanban_root}` resolves to `KB/PM_and_Portfolio`
- This allows easy reconfiguration without changing individual step configs

---

## 📝 Step-by-Step Reference

### Step 1: Review Kanban State

**Handler:** `kanban.review_state`
**Category:** Kanban
**Icon:** 📊
**Required:** ✅ Yes
**Default Dependencies:** None (first step)

#### Purpose

Analyzes the current state of Kanban documentation by scanning all Epic and Story documents, extracting status information, task counts, and document metrics. This step creates a baseline state that subsequent analysis steps build upon.

#### Configuration Parameters

**`kanban_root`** (string, required)
- **Default:** `KB/PM_and_Portfolio`
- **Label:** Kanban Root Directory
- **Description:** Path to Kanban documentation root directory
- **Usage:** Specifies where to find Epic and Story documents

#### Execution Details

1. **Scans Kanban root directory** for Epic and Story documents
2. **Extracts status information** from each document (e.g., "IN PROGRESS", "COMPLETE", "TODO")
3. **Counts stories and tasks** in each Epic document
4. **Calculates document metrics** (line count, word count, non-empty lines)
5. **Creates summary statistics** (total Epics, total Stories, completion status)
6. **Saves findings** to `kanban_review_findings.json` for use by subsequent steps

#### Output

The step outputs a state object containing:
- `kanban_root`: Path to the Kanban root directory
- `epics`: Dictionary of Epic documents with status, story count, task count, and metrics
- `stories`: Dictionary of Story documents with status, task count, and metrics
- `summary`: Summary statistics (total Epics, total Stories, completion counts)

#### Findings File

Findings are saved to `kanban_review_findings.json` with the key `state_review`:
```json
{
  "state_review": {
    "kanban_root": "KB/PM_and_Portfolio",
    "epics": {...},
    "stories": {...},
    "summary": {...}
  }
}
```

---

### Step 2: Check Template Divergence

**Handler:** `kanban.check_template_divergence`
**Category:** Kanban
**Icon:** 🔍
**Required:** ✅ Yes
**Default Dependencies:** `step-1`

#### Purpose

Compares each Epic and Story document against their respective templates to identify structural differences. This step detects missing sections, extra sections, and structural deviations from the expected template format.

#### Configuration Parameters

**`kanban_root`** (string, required)
- **Default:** `KB/PM_and_Portfolio`
- **Label:** Kanban Root Directory
- **Description:** Path to Kanban documentation root directory

**`epic_template`** (string, required)
- **Default:** `KB/Documentation/Document_Templates/Kanban/EPIC_TEMPLATE.md`
- **Label:** Epic Template Path
- **Description:** Path to the Epic document template for comparison

**`story_template`** (string, required)
- **Default:** `KB/Documentation/Document_Templates/Kanban/STORY_TEMPLATE.md`
- **Label:** Story Template Path
- **Description:** Path to the Story document template for comparison

#### Execution Details

1. **Parses Epic and Story templates** to extract expected section names
2. **Scans all Epic documents** and extracts their section names
3. **Compares Epic sections** against template sections to find missing/extra sections
4. **Scans all Story documents** and extracts their section names
5. **Compares Story sections** against template sections to find missing/extra sections
6. **Documents divergences** with file path, document type, and section differences
7. **Saves findings** to `kanban_review_findings.json`

#### Output

The step outputs:
- `divergences`: List of documents with template divergences
- `count`: Total number of diverged documents

Each divergence entry contains:
- `file`: Path to the diverged document
- `type`: Document type ("epic" or "story")
- `missing_sections`: List of sections present in template but missing in document
- `extra_sections`: List of sections present in document but not in template

#### Findings File

Findings are saved under the key `template_divergences`:
```json
{
  "template_divergences": [
    {
      "file": "KB/PM_and_Portfolio/epics/overview/Epic 01/Epic-01.md",
      "type": "epic",
      "missing_sections": ["summary", "naming_convention"],
      "extra_sections": []
    }
  ]
}
```

#### Dependencies

**Requires:** Step 1 (Review Kanban State)
- Uses the document list from Step 1
- Builds on the state review to perform template comparison

---

### Step 3: Check Documentation Bloat

**Handler:** `kanban.check_bloat`
**Category:** Kanban
**Icon:** 📦
**Required:** ✅ Yes
**Default Dependencies:** `step-1`

#### Purpose

Identifies documents that exceed the recommended size threshold, indicating potential bloat, verbosity, or duplicative content. This step helps maintain concise, focused documentation.

#### Configuration Parameters

**`kanban_root`** (string, required)
- **Default:** `KB/PM_and_Portfolio`
- **Label:** Kanban Root Directory
- **Description:** Path to Kanban documentation root directory

**`bloat_threshold`** (integer, required)
- **Default:** `500`
- **Label:** Bloat Threshold
- **Description:** Maximum recommended lines per document
- **Usage:** Documents exceeding this threshold are flagged as bloated

#### Execution Details

1. **Scans all Epic documents** and calculates document metrics
2. **Compares Epic document line count** against bloat threshold
3. **Flags Epic documents** exceeding the threshold
4. **Scans all Story documents** and calculates document metrics
5. **Compares Story document line count** against bloat threshold
6. **Flags Story documents** exceeding the threshold
7. **Documents bloated files** with line count, word count, and excess lines
8. **Saves findings** to `kanban_review_findings.json`

#### Output

The step outputs:
- `bloated_docs`: List of bloated documents
- `count`: Total number of bloated documents

Each bloated document entry contains:
- `file`: Path to the bloated document
- `type`: Document type ("epic" or "story")
- `lines`: Total non-empty lines in the document
- `threshold`: Bloat threshold used for comparison
- `excess_lines`: Number of lines exceeding the threshold
- `word_count`: Total word count in the document

#### Findings File

Findings are saved under the key `bloated_docs`:
```json
{
  "bloated_docs": [
    {
      "file": "KB/PM_and_Portfolio/epics/overview/Epic 4/Epic-4.md",
      "type": "epic",
      "lines": 650,
      "threshold": 500,
      "excess_lines": 150,
      "word_count": 12000
    }
  ]
}
```

#### Dependencies

**Requires:** Step 1 (Review Kanban State)
- Uses the document list from Step 1
- Leverages document metrics from the state review

---

### Step 4: Check Repetition/Contradictions

**Handler:** `kanban.check_repetition`
**Category:** Kanban
**Icon:** 🔄
**Required:** ✅ Yes
**Default Dependencies:** `step-1`

#### Purpose

Identifies repeated text blocks across documents and detects contradictory statements (e.g., different statuses, conflicting dates, inconsistent information). This step helps maintain consistency and eliminate redundancy.

#### Configuration Parameters

**`kanban_root`** (string, required)
- **Default:** `KB/PM_and_Portfolio`
- **Label:** Kanban Root Directory
- **Description:** Path to Kanban documentation root directory

**`min_text_length`** (integer, required)
- **Default:** `20`
- **Label:** Minimum Text Length
- **Description:** Minimum text length to consider for repetition detection
- **Usage:** Excludes short summaries from repetition analysis to avoid false positives

#### Execution Details

1. **Scans all Epic and Story documents** for text content
2. **Extracts text blocks** of minimum length from each document
3. **Compares text blocks** across documents to find repetitions
4. **Analyzes documents** for contradictory statements:
   - Different statuses for the same Epic/Story
   - Conflicting dates or version numbers
   - Inconsistent information
5. **Documents repetitions and contradictions** with file paths and content
6. **Saves findings** to `kanban_review_findings.json`

#### Output

The step outputs:
- `repetitions`: List of repeated text blocks
- `contradictions`: List of contradictory statements
- `repetition_count`: Total number of repetitions found
- `contradiction_count`: Total number of contradictions found

Each repetition entry contains:
- `text`: The repeated text block
- `files`: List of files containing this text
- `count`: Number of occurrences

Each contradiction entry contains:
- `type`: Type of contradiction (e.g., "status_mismatch", "date_conflict")
- `files`: Files with conflicting information
- `details`: Description of the contradiction

#### Findings File

Findings are saved under the keys `repetitions` and `contradictions`:
```json
{
  "repetitions": [
    {
      "text": "This epic establishes the foundational system design...",
      "files": ["Epic-1.md", "Epic-2.md"],
      "count": 2
    }
  ],
  "contradictions": [
    {
      "type": "status_mismatch",
      "files": ["Epic-01.md"],
      "details": "Epic 01 status is 'IN PROGRESS' but all stories are 'COMPLETE'"
    }
  ]
}
```

#### Dependencies

**Requires:** Step 1 (Review Kanban State)
- Uses the document list from Step 1
- Analyzes all documents found in the state review

---

### Step 5: Check Coverage Gaps

**Handler:** `kanban.check_coverage_gaps`
**Category:** Kanban
**Icon:** 🔎
**Required:** ✅ Yes
**Default Dependencies:** `step-1`, `step-2`

#### Purpose

Identifies missing sections, vague content, and undocumented dependencies or risks in Epic and Story documents. This step builds on template divergence findings to identify coverage gaps.

#### Configuration Parameters

**`kanban_root`** (string, required)
- **Default:** `KB/PM_and_Portfolio`
- **Label:** Kanban Root Directory
- **Description:** Path to Kanban documentation root directory

**`epic_template`** (string, required)
- **Default:** `KB/Documentation/Document_Templates/Kanban/EPIC_TEMPLATE.md`
- **Label:** Epic Template Path
- **Description:** Path to the Epic document template for comparison

**`story_template`** (string, required)
- **Default:** `KB/Documentation/Document_Templates/Kanban/STORY_TEMPLATE.md`
- **Label:** Story Template Path
- **Description:** Path to the Story document template for comparison

#### Execution Details

1. **Parses Epic and Story templates** to identify expected sections
2. **Scans Epic documents** for missing sections and vague content
3. **Analyzes Epic content** for insufficient detail in required sections
4. **Scans Story documents** for missing sections and vague content
5. **Analyzes Story content** for insufficient detail in required sections
6. **Identifies undocumented dependencies** or risks
7. **Documents coverage gaps** with file path, issue type, and recommendations
8. **Saves findings** to `kanban_review_findings.json`

#### Output

The step outputs:
- `gaps`: List of coverage gaps identified
- `count`: Total number of coverage gaps found

Each gap entry contains:
- `file`: Path to the document with the gap
- `type`: Type of gap (e.g., "missing_section", "insufficient_detail", "vague_content")
- `section`: Section name where the gap exists
- `description`: Description of the gap
- `recommendation`: Recommended fix

#### Findings File

Findings are saved under the key `coverage_gaps`:
```json
{
  "coverage_gaps": [
    {
      "file": "KB/PM_and_Portfolio/epics/overview/Epic 01/Epic-01.md",
      "type": "missing_section",
      "section": "Summary",
      "description": "Missing required Summary section",
      "recommendation": "Add Summary section describing the epic's purpose"
    },
    {
      "file": "KB/PM_and_Portfolio/stories/overview/Epic 01/Story-1.md",
      "type": "insufficient_detail",
      "section": "Overview",
      "description": "Overview section is too brief",
      "recommendation": "Expand Overview section with more detail"
    }
  ]
}
```

#### Dependencies

**Requires:** Step 1 (Review Kanban State) and Step 2 (Check Template Divergence)
- Uses the document list from Step 1
- Builds on template divergence findings from Step 2 to identify missing sections
- Leverages template structure to identify required sections

---

### Step 6: Apply Documented Updates

**Handler:** `kanban.apply_updates`
**Category:** Kanban
**Icon:** ✅
**Required:** ❌ No (optional)
**Default Dependencies:** `step-2`, `step-3`, `step-4`, `step-5`

#### Purpose

Applies fixes from documented findings to update diverged documents, trim bloated content, resolve contradictions, and fill coverage gaps. **⚠️ Warning:** This step is optional and disabled by default. Only enable auto-apply after carefully reviewing all findings.

#### Configuration Parameters

**`auto_apply`** (boolean, required)
- **Default:** `false`
- **Label:** Auto-Apply Fixes
- **Description:** Automatically apply documented fixes (use with caution)
- **⚠️ Warning:** Setting this to `true` will automatically modify documentation files. Always review findings first.

#### Execution Details

When `auto_apply` is `false` (default):
1. **Loads findings** from `kanban_review_findings.json`
2. **Summarizes findings** without applying changes:
   - Number of template divergences
   - Number of bloated documents
   - Number of repetitions and contradictions
   - Number of coverage gaps
3. **Documents findings** for manual review
4. **Outputs summary** without modifying files

When `auto_apply` is `true`:
1. **Loads all findings** from previous analysis steps
2. **Applies template fixes** to diverged documents:
   - Adds missing sections from templates
   - Removes or flags extra sections
3. **Trims bloated content** (if possible):
   - Removes verbose or duplicative content
   - Condenses overly long sections
4. **Resolves contradictions** (if possible):
   - Updates conflicting information based on priority rules
   - Flags unresolved contradictions for manual review
5. **Fills coverage gaps**:
   - Adds missing sections with placeholder content
   - Expands vague sections with more detail
6. **Saves applied fixes** with file paths and changes made

#### Output

The step outputs:
- `applied_fixes`: List of fixes applied (if auto_apply is true)
- `count`: Number of fixes applied
- `summary`: Summary of findings (if auto_apply is false)

#### Findings File

When `auto_apply` is `true`, applied fixes are saved under the key `applied_fixes`:
```json
{
  "applied_fixes": [
    {
      "file": "KB/PM_and_Portfolio/epics/overview/Epic 01/Epic-01.md",
      "type": "add_section",
      "section": "Summary",
      "status": "applied"
    },
    {
      "file": "KB/PM_and_Portfolio/epics/overview/Epic 4/Epic-4.md",
      "type": "trim_content",
      "section": "Notes",
      "lines_removed": 50,
      "status": "applied"
    }
  ]
}
```

#### Dependencies

**Requires:** Steps 2, 3, 4, and 5 (all analysis steps)
- Uses template divergence findings from Step 2
- Uses bloat findings from Step 3
- Uses repetition/contradiction findings from Step 4
- Uses coverage gap findings from Step 5

**⚠️ Important Notes:**
- Always review findings before enabling auto_apply
- Make a backup of documentation before running with auto_apply=true
- Some fixes may require manual review and adjustment
- Test on a small subset first if unsure

---

## 🔄 Execution Flow

### Dependency Graph

```
Step 1 (Review Kanban State)
  ├─→ Step 2 (Check Template Divergence)
  ├─→ Step 3 (Check Documentation Bloat)
  ├─→ Step 4 (Check Repetition/Contradictions)
  └─→ Step 5 (Check Coverage Gaps) ← depends on 1,2
        ↓
      Step 6 (Apply Documented Updates) ← depends on 2,3,4,5
```

### Execution Phases

**Phase 1: State Analysis** (Step 1)
- Creates baseline state for all subsequent analysis
- Scans and catalogs all Epic and Story documents

**Phase 2: Parallel Analysis** (Steps 2, 3, 4)
- All run in parallel after Step 1 completes
- Each performs independent analysis:
  - Step 2: Template comparison
  - Step 3: Size analysis
  - Step 4: Content analysis

**Phase 3: Coverage Analysis** (Step 5)
- Runs after Steps 1 and 2 complete
- Uses template divergence findings to identify gaps

**Phase 4: Fix Application** (Step 6, optional)
- Runs after all analysis steps complete (2, 3, 4, 5)
- Applies fixes based on all findings
- **Disabled by default** - requires explicit enablement

### Parallel Execution

Steps that can run in parallel (after dependencies are met):
- **Steps 2, 3, 4:** All depend on Step 1, but don't depend on each other
- These steps can execute simultaneously for faster analysis

---

## 📊 Findings File Structure

All steps save their findings to `kanban_review_findings.json` in the workspace root:

```json
{
  "state_review": {
    "kanban_root": "KB/PM_and_Portfolio",
    "epics": {...},
    "stories": {...},
    "summary": {...}
  },
  "template_divergences": [...],
  "bloated_docs": [...],
  "repetitions": [...],
  "contradictions": [...],
  "coverage_gaps": [...],
  "applied_fixes": [...]
}
```

---

## 🎯 Workflow Parameters

These parameters can be configured when executing the workflow:

### `auto_apply` (boolean)
- **Default:** `false`
- **Description:** Automatically apply documented fixes
- **⚠️ Warning:** Use with caution - will modify documentation files

### `bloat_threshold` (integer)
- **Default:** `500`
- **Description:** Maximum recommended lines per document
- **Usage:** Adjust based on your documentation standards

### `min_text_length` (integer)
- **Default:** `20`
- **Description:** Minimum text length to consider for repetition detection
- **Usage:** Increase to ignore shorter text blocks, decrease to catch more repetitions

---

## 📚 Related Documentation

- **[Tutorial: Building Kanban Review Workflow from Scratch](tutorial-kanban-review-workflow.md)** - Step-by-step tutorial
- **[Visual Designer Guide](visual-designer-guide.md)** - Using the visual designer
- **[Workflow Execution Guide](execution-guide.md)** - Running and monitoring workflows
- **[Release Workflow Reference](release-workflow-reference.md)** - Reference for another workflow example

---

**Last Updated:** 2025-11-21
**Reference Version:** 1.0.0
