# Tutorial: Building the Kanban Review Workflow from Scratch

**Version:** 1.0.0
**Last Updated:** 2025-11-21
**Related:** Epic 01, Story 4 - Comprehensive VWMP Documentation

---

## 🎯 Overview

This tutorial will guide you through building the **Kanban Review Workflow** from scratch in the Visual Workflow Management Platform (VWMP). The Kanban Review Workflow automates documentation quality assurance by analyzing Epic and Story documents for template compliance, bloat, repetition, contradictions, and coverage gaps.

**By the end of this tutorial, you will:**
- Understand how to create analysis workflows in VWMP
- Know how to configure multi-step analysis workflows with dependencies
- Understand how to chain analysis steps that build on each other's findings
- Be able to build a complete documentation quality assurance workflow

**Prerequisites:**
- VWMP is installed and running
- Access to the visual designer at `http://localhost:8000/vwmp/designer/`
- Basic understanding of YAML syntax
- Familiarity with Kanban documentation structure

**Estimated Time:** 30-45 minutes

---

## 📋 What We're Building

The Kanban Review Workflow consists of **6 steps** that analyze documentation quality:

1. **Review Kanban State** - Analyze current Kanban documentation state
2. **Check Template Divergence** - Compare documents against templates
3. **Check Documentation Bloat** - Identify oversized documents
4. **Check Repetition/Contradictions** - Find repeated or conflicting information
5. **Check Coverage Gaps** - Identify missing sections and vague content
6. **Apply Documented Updates** - Optionally apply automated fixes

---

## 🚀 Step-by-Step Tutorial

### Step 1: Open the Visual Designer

1. **Navigate to the designer:**
   ```
   http://localhost:8000/vwmp/designer/
   ```

2. **Click "New Workflow"** to create a new workflow from scratch.

3. **Enter workflow metadata:**
   - **Name:** `Kanban Review Workflow`
   - **Version:** `1.0.0`
   - **Type:** `kanban_review`
   - **Description:** `Automated Kanban documentation review workflow for quality assurance`

---

### Step 2: Configure Workflow Parameters

Before adding steps, configure the workflow-level parameters that will be used by multiple steps:

1. **Click "Workflow Config"** in the workflow editor
2. **Add the following configuration:**
   ```yaml
   kanban_root: KB/PM_and_Portfolio
   epic_template: KB/Documentation/Document_Templates/Kanban/EPIC_TEMPLATE.md
   story_template: KB/Documentation/Document_Templates/Kanban/STORY_TEMPLATE.md
   bloat_threshold: 500
   min_text_length: 20
   ```

**Configuration Explanation:**
- `kanban_root`: Path to the Kanban documentation root directory
- `epic_template`: Path to the Epic document template for comparison
- `story_template`: Path to the Story document template for comparison
- `bloat_threshold`: Maximum recommended lines per document (default: 500)
- `min_text_length`: Minimum text length to consider for repetition detection (default: 20)

---

### Step 3: Add Step 1 - Review Kanban State

1. **From the step palette**, find **"Review Kanban State"** in the Kanban category
2. **Drag it onto the canvas** - This will be the first step (no dependencies)
3. **Click the step** to configure it:
   - **Step ID:** `step-1`
   - **Name:** `Review Kanban State`
   - **Handler:** `kanban.review_state`
   - **Required:** ✅ Yes
   - **Dependencies:** (leave empty - this is the first step)

4. **Configure step parameters:**
   ```yaml
   kanban_root: ${config.kanban_root}
   ```

**What this step does:**
- Scans the Kanban documentation root for all Epic and Story documents
- Extracts status, task counts, and metrics for each document
- Creates a summary of the overall Kanban state
- Saves findings to `kanban_review_findings.json` for use by subsequent steps

---

### Step 4: Add Step 2 - Check Template Divergence

1. **Drag "Check Template Divergence"** from the Kanban category onto the canvas
2. **Position it after Step 1** and connect Step 1's output to Step 2's input
3. **Configure the step:**
   - **Step ID:** `step-2`
   - **Name:** `Check Template Divergence`
   - **Handler:** `kanban.check_template_divergence`
   - **Required:** ✅ Yes
   - **Dependencies:** `step-1`

4. **Configure step parameters:**
   ```yaml
   kanban_root: ${config.kanban_root}
   epic_template: ${config.epic_template}
   story_template: ${config.story_template}
   ```

**What this step does:**
- Compares each Epic/Story document against the respective template
- Identifies missing sections, extra sections, and structural differences
- Documents findings for each diverged document

**Why it depends on Step 1:**
- Uses the document list and state information from Step 1
- Builds on the initial state review to perform template comparison

---

### Step 5: Add Step 3 - Check Documentation Bloat

1. **Drag "Check Documentation Bloat"** from the Kanban category onto the canvas
2. **Position it parallel to Step 2** (both depend on Step 1)
3. **Connect Step 1's output to Step 3's input**
4. **Configure the step:**
   - **Step ID:** `step-3`
   - **Name:** `Check Documentation Bloat`
   - **Handler:** `kanban.check_bloat`
   - **Required:** ✅ Yes
   - **Dependencies:** `step-1`

5. **Configure step parameters:**
   ```yaml
   kanban_root: ${config.kanban_root}
   bloat_threshold: ${config.bloat_threshold}
   ```

**What this step does:**
- Analyzes document sizes (line count, word count)
- Identifies documents exceeding the bloat threshold
- Flags verbose or duplicative content

**Note:** Steps 2, 3, and 4 can all run in parallel since they only depend on Step 1.

---

### Step 6: Add Step 4 - Check Repetition/Contradictions

1. **Drag "Check Repetition/Contradictions"** from the Kanban category onto the canvas
2. **Position it parallel to Steps 2 and 3** (all depend on Step 1)
3. **Connect Step 1's output to Step 4's input**
4. **Configure the step:**
   - **Step ID:** `step-4`
   - **Name:** `Check Repetition/Contradictions`
   - **Handler:** `kanban.check_repetition`
   - **Required:** ✅ Yes
   - **Dependencies:** `step-1`

5. **Configure step parameters:**
   ```yaml
   kanban_root: ${config.kanban_root}
   min_text_length: ${config.min_text_length}
   ```

**What this step does:**
- Searches for repeated information across documents
- Identifies contradictory statements (e.g., different statuses, dates)
- Excludes succinct summaries as per configuration

---

### Step 7: Add Step 5 - Check Coverage Gaps

1. **Drag "Check Coverage Gaps"** from the Kanban category onto the canvas
2. **Position it after Steps 1 and 2** (depends on both)
3. **Connect Step 1's and Step 2's outputs to Step 5's inputs**
4. **Configure the step:**
   - **Step ID:** `step-5`
   - **Name:** `Check Coverage Gaps`
   - **Handler:** `kanban.check_coverage_gaps`
   - **Required:** ✅ Yes
   - **Dependencies:** `step-1`, `step-2`

5. **Configure step parameters:**
   ```yaml
   kanban_root: ${config.kanban_root}
   epic_template: ${config.epic_template}
   story_template: ${config.story_template}
   ```

**What this step does:**
- Identifies missing sections in documents
- Finds vague statements needing more detail
- Identifies undocumented dependencies or risks

**Why it depends on Step 2:**
- Uses template divergence findings to identify missing sections
- Builds on template comparison to find coverage gaps

---

### Step 8: Add Step 6 - Apply Documented Updates

1. **Drag "Apply Documented Updates"** from the Kanban category onto the canvas
2. **Position it at the end** (depends on all analysis steps)
3. **Connect Steps 2, 3, 4, and 5's outputs to Step 6's inputs**
4. **Configure the step:**
   - **Step ID:** `step-6`
   - **Name:** `Apply Documented Updates`
   - **Handler:** `kanban.apply_updates`
   - **Required:** ❌ No (optional step)
   - **Enabled:** ✅ Yes
   - **Dependencies:** `step-2`, `step-3`, `step-4`, `step-5`

5. **Configure step parameters:**
   ```yaml
   auto_apply: false
   ```

**What this step does:**
- Applies fixes from documented findings
- Updates diverged docs to match templates
- Trims bloated content
- Resolves contradictions
- Fills coverage gaps

**⚠️ Important:** This step is optional and disabled by default (`auto_apply: false`). Only enable auto-apply after carefully reviewing all findings.

---

### Step 9: Configure Workflow Parameters

1. **Click "Workflow Parameters"** in the workflow editor
2. **Add the following parameters:**

   **Parameter 1: Auto-Apply**
   - **Name:** `auto_apply`
   - **Type:** `boolean`
   - **Required:** ❌ No
   - **Default:** `false`
   - **Label:** `Auto-Apply Fixes`
   - **Description:** `Automatically apply documented fixes (use with caution)`

   **Parameter 2: Bloat Threshold**
   - **Name:** `bloat_threshold`
   - **Type:** `integer`
   - **Required:** ❌ No
   - **Default:** `500`
   - **Label:** `Bloat Threshold`
   - **Description:** `Maximum recommended lines per document`

   **Parameter 3: Minimum Text Length**
   - **Name:** `min_text_length`
   - **Type:** `integer`
   - **Required:** ❌ No
   - **Default:** `20`
   - **Label:** `Minimum Text Length`
   - **Description:** `Minimum text length to consider for repetition detection`

---

### Step 10: Save and Test the Workflow

1. **Click "Save Workflow"** to save your workflow
2. **Review the workflow structure:**
   - Verify all dependencies are correctly connected
   - Ensure Step 1 has no dependencies (it's the starting point)
   - Ensure Step 6 depends on all analysis steps (2-5)
   - Check that Steps 2, 3, and 4 can run in parallel

3. **Test the workflow:**
   - Click "Execute Workflow"
   - Monitor execution in real-time
   - Review the generated `kanban_review_findings.json` file
   - Check logs for each step

---

## 📊 Workflow Execution Flow

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
- Initial scan of all Kanban documentation
- Creates baseline state for subsequent analysis

**Phase 2: Parallel Analysis** (Steps 2, 3, 4)
- All run in parallel after Step 1 completes
- Each performs independent analysis

**Phase 3: Coverage Analysis** (Step 5)
- Runs after Steps 1 and 2 complete
- Uses template divergence findings

**Phase 4: Fix Application** (Step 6, optional)
- Runs after all analysis steps complete
- Applies fixes based on all findings

---

## 🎓 Key Concepts Learned

1. **Chained Analysis:** Steps can build on each other's findings using the `kanban_review_findings.json` file
2. **Parallel Execution:** Independent analysis steps can run in parallel for efficiency
3. **Optional Steps:** Some steps can be marked as optional and enabled/disabled as needed
4. **Configuration Reuse:** Workflow-level config can be referenced in steps using `${config.variable_name}`
5. **Parameter Management:** Workflow parameters allow runtime configuration without editing the workflow

---

## 📚 Next Steps

- **[Kanban Review Workflow Reference](kanban-review-workflow-reference.md)** - Detailed reference for each step
- **[Release Workflow Tutorial](tutorial-release-workflow.md)** - Learn another workflow example
- **[Visual Designer Guide](visual-designer-guide.md)** - Master the visual designer interface

---

**Last Updated:** 2025-11-21
**Tutorial Version:** 1.0.0
