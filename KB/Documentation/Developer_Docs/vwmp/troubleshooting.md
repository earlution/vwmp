# VWMP Troubleshooting Guide

**Version:** 1.0.0
**Last Updated:** 2025-11-21
**Related:** Epic 01 - Visual Workflow Management Platform

---

## 📖 Overview

This guide helps you troubleshoot common issues when using the Visual Workflow Management Platform (VWMP). It covers errors, warnings, and common problems you may encounter, along with their solutions.

**Who This Guide Is For:**
- Users experiencing issues with VWMP
- Developers debugging workflow problems
- Administrators troubleshooting system issues

---

## 🔍 Quick Troubleshooting Checklist

Before diving into specific issues, try these quick checks:

1. ✅ **Is VWMP running?** - Check `http://localhost:8000/vwmp/`
2. ✅ **Are you authenticated?** - Check if you're logged in
3. ✅ **Is the workflow saved?** - Ensure workflow is saved before executing
4. ✅ **Are all steps configured?** - Check that required parameters are set
5. ✅ **Are dependencies correct?** - Verify step dependencies are valid
6. ✅ **Check execution logs** - Review logs for error messages

---

## 🚨 Common Errors

### Error: "Workflow not found"

**Symptoms:**
- Workflow doesn't load in designer
- "404 Not Found" error
- Workflow ID not recognized

**Causes:**
- Workflow was deleted
- Incorrect workflow ID in URL
- Workflow file missing from storage

**Solutions:**
1. **Check workflow ID** in URL: `/vwmp/designer/<workflow-id>/`
2. **Verify workflow exists** in dashboard workflow list
3. **Recreate workflow** if it was accidentally deleted
4. **Check workflow storage** directory: `workflows/`

---

### Error: "Handler not found"

**Symptoms:**
- Step shows error badge
- "Handler not found" in logs
- Step configuration panel shows error

**Causes:**
- Step handler path is incorrect
- Handler plugin not registered
- Handler class doesn't exist

**Solutions:**
1. **Check handler path** in step configuration:
   - Format: `plugin.handler_name` (e.g., `release.version_bump`)
   - Must match registered handler path
2. **Verify handler exists** in plugin registry:
   - Check `src/vwmp/plugins/registry.py`
   - Ensure handler is registered
3. **Check handler import** - ensure handler class is imported
4. **Review step configuration** - verify handler path matches plugin structure

---

### Error: "Circular dependency detected"

**Symptoms:**
- Workflow validation fails
- "Circular dependency" error message
- Cannot execute workflow

**Causes:**
- Step A depends on Step B, and Step B depends on Step A
- Complex dependency chain creates a cycle

**Solutions:**
1. **Review step dependencies** - identify the cycle
2. **Break the cycle** - remove unnecessary dependencies
3. **Reorganize steps** - restructure workflow to avoid cycles
4. **Use workflow config** - share data via config instead of dependencies

**Example Fix:**
```yaml
# Before (circular):
step-1:
  dependencies: [step-2]
step-2:
  dependencies: [step-1]  # ❌ Circular!

# After (fixed):
step-1:
  dependencies: []
step-2:
  dependencies: [step-1]  # ✅ Linear
```

---

### Error: "Required parameter missing"

**Symptoms:**
- Step shows validation error
- "Required parameter missing" message
- Step configuration panel shows error

**Causes:**
- Required step parameter not provided
- Parameter name typo
- Parameter value is empty

**Solutions:**
1. **Check step configuration** - open configuration panel
2. **Review required fields** - marked with asterisk (*)
3. **Fill all required parameters** - provide values for all required fields
4. **Verify parameter names** - ensure they match handler schema
5. **Check parameter types** - ensure values match expected types

**Example:**
```yaml
# Step configuration showing required parameter
config:
  kanban_root: KB/PM_and_Portfolio  # ✅ Required - provided
  # epic_template: ...  # ❌ Required - missing!
```

---

### Error: "File not found"

**Symptoms:**
- Step fails during execution
- "File not found" or "FileNotFoundError" in logs
- Workflow execution stops

**Causes:**
- File path is incorrect
- File doesn't exist at specified location
- Relative path resolves incorrectly

**Solutions:**
1. **Verify file path** - check if file exists at specified location
2. **Use absolute paths** - or verify relative paths from workspace root
3. **Check path format** - ensure forward slashes for cross-platform compatibility
4. **Verify permissions** - ensure file is readable/writable

**Example Fix:**
```yaml
# Before (incorrect):
config:
  version_file: version.py  # ❌ Relative path may not resolve correctly

# After (fixed):
config:
  version_file: VERSION  # ✅ Path from workspace root
```

---

### Error: "Permission denied"

**Symptoms:**
- Step fails during execution
- "Permission denied" error
- Cannot write to files/directories

**Causes:**
- Insufficient file permissions
- Directory not writable
- Running with wrong user permissions

**Solutions:**
1. **Check file permissions** - ensure files/directories are writable
2. **Verify directory exists** - create directories if needed
3. **Check user permissions** - ensure running user has required access
4. **Review Git permissions** - ensure Git operations are allowed

**Example Fix:**
```bash
# Check permissions
ls -l VERSION

# Fix permissions if needed
chmod 644 VERSION
chmod 755 workflows/
```

---

### Error: "Step execution failed"

**Symptoms:**
- Step status shows "Failed" (red)
- Error message in execution log
- Workflow execution stops or continues with warnings

**Causes:**
- Step handler encountered an error
- Invalid configuration provided
- External dependency failed

**Solutions:**
1. **Check execution logs** - review error message details
2. **Review step output** - check if step produced any output
3. **Validate configuration** - ensure all parameters are correct
4. **Check handler documentation** - verify parameter usage
5. **Test step individually** - isolate the problem

**Example:**
```
Error in execution log:
"TypeError: '>' not supported between instances of 'int' and 'str'"

Fix:
Convert string to integer:
bloat_threshold: 500  # Instead of "500"
```

---

### Error: "JSON serialization error"

**Symptoms:**
- Error when loading/saving workflow
- "Object is not JSON serializable" error
- Workflow data cannot be saved

**Causes:**
- Workflow contains non-JSON-serializable objects
- Dataclass objects not converted to dict
- Circular references in data

**Solutions:**
1. **Check workflow data** - ensure all data is JSON-serializable
2. **Convert dataclasses** - use `.to_dict()` method if available
3. **Remove circular references** - simplify data structure
4. **Review step output** - ensure outputs are serializable

---

## ⚠️ Common Warnings

### Warning: "Step skipped"

**Symptoms:**
- Step shows "Skipped" status (orange)
- Step not executed
- Workflow continues without step

**Causes:**
- Step marked as `required: false`
- Step marked as `enabled: false`
- Dependency failed and step not required

**Solutions:**
1. **Check if skipping is expected** - review step configuration
2. **Enable step** if it should run:
   ```yaml
   enabled: true  # Change from false
   ```
3. **Mark as required** if it must run:
   ```yaml
   required: true  # Change from false
   ```
4. **Fix dependencies** if dependency failure caused skip

---

### Warning: "Validation errors found"

**Symptoms:**
- Red error badges on steps
- Validation panel shows errors
- Cannot execute workflow

**Causes:**
- Missing required parameters
- Invalid parameter values
- Dependency issues

**Solutions:**
1. **Open validation panel** - review all errors
2. **Fix each error** - address validation issues
3. **Re-validate** - ensure all errors are resolved
4. **Review step configuration** - check all required fields

---

### Warning: "Workflow execution took longer than expected"

**Symptoms:**
- Long execution time
- Steps taking too long to complete
- Timeout warnings

**Causes:**
- Large files being processed
- Network operations
- Complex operations in handlers
- Too many sequential steps

**Solutions:**
1. **Check execution logs** - identify slow steps
2. **Optimize handlers** - reduce operation complexity
3. **Enable parallel execution** - minimize sequential dependencies
4. **Add timeout handling** - configure timeouts for long-running steps
5. **Review file sizes** - optimize file processing

---

## 🐛 Debugging Workflows

### Step-by-Step Debugging

**1. Isolate the Problem:**
- Disable all steps except the problematic one
- Run workflow with minimal configuration
- Add steps back one at a time

**2. Check Step Output:**
- Review step output in execution logs
- Verify output format matches expectations
- Check if output is used by dependent steps

**3. Test Step Individually:**
- Create minimal workflow with just the problem step
- Test with known good configuration
- Verify handler works independently

**4. Review Dependencies:**
- Check if all dependencies completed successfully
- Verify dependency outputs are available
- Ensure dependencies provide required data

### Debugging Tips

**Enable Debug Logging:**
- Check execution logs for detailed information
- Look for stack traces in error logs
- Review step-by-step execution flow

**Use Test Data:**
- Test with sample data before production
- Use small test files to isolate issues
- Verify outputs match expectations

**Check Configuration:**
- Validate all parameter values
- Verify file paths exist
- Ensure permissions are correct

**Review Findings Files:**
- Some workflows save findings (e.g., `kanban_review_findings.json`)
- Check findings for intermediate results
- Verify data is saved correctly

---

## 🔧 Configuration Issues

### Issue: "Config variable not found"

**Symptoms:**
- `${config.variable_name}` not resolving
- Step fails to find config value
- Variable undefined error

**Solutions:**
1. **Check workflow config** - ensure variable is defined:
   ```yaml
   config:
     variable_name: value  # ✅ Must be defined
   ```
2. **Verify syntax** - use `${config.variable_name}` format
3. **Check variable name** - ensure exact match (case-sensitive)
4. **Review scope** - config variables are workflow-scoped

---

### Issue: "Parameter not accessible in step"

**Symptoms:**
- Step cannot access workflow parameter
- Parameter value not available
- `${parameters.param_name}` not resolving

**Solutions:**
1. **Check parameter definition** - ensure parameter exists in workflow
2. **Provide parameter value** when executing workflow
3. **Verify syntax** - use `${parameters.param_name}` format
4. **Check parameter name** - ensure exact match

**Example:**
```yaml
# Workflow parameters
parameters:
  - name: auto_apply
    type: boolean
    default: false

# Step config
config:
  auto_apply: ${parameters.auto_apply}  # ✅ References parameter
```

---

## 🌐 UI Issues

### Issue: "Canvas is empty"

**Symptoms:**
- Visual designer shows blank canvas
- No workflow loaded
- Steps not visible

**Solutions:**
1. **Check workflow ID** - verify correct workflow is loaded
2. **Load workflow** - click "Load Workflow" or navigate to workflow URL
3. **Check browser console** - look for JavaScript errors
4. **Refresh page** - reload designer interface
5. **Check React Flow** - ensure React Flow library loaded correctly

---

### Issue: "Steps not draggable"

**Symptoms:**
- Cannot drag steps from palette
- Steps don't drop on canvas
- Drag-and-drop not working

**Solutions:**
1. **Check browser compatibility** - ensure modern browser
2. **Disable browser extensions** - extensions may interfere
3. **Clear browser cache** - refresh page with cache cleared
4. **Check JavaScript errors** - review browser console
5. **Try different browser** - test in Chrome, Firefox, Safari

---

### Issue: "Configuration panel not opening"

**Symptoms:**
- Clicking step doesn't open config panel
- Panel appears but is empty
- Cannot configure step

**Solutions:**
1. **Click step node** - ensure step is selected
2. **Check panel visibility** - panel may be off-screen
3. **Refresh page** - reload designer
4. **Check browser console** - look for JavaScript errors
5. **Verify step handler** - ensure handler provides config schema

---

## 🔐 Authentication Issues

### Issue: "Authentication required"

**Symptoms:**
- "Please log in" error
- API requests fail with 401
- Cannot execute workflows

**Solutions:**
1. **Log in to FastAPI** - ensure you're authenticated
2. **Check session** - verify session is valid
3. **Refresh login** - log out and log back in
4. **Check authentication middleware** - ensure middleware is configured
5. **Review permissions** - verify user has required permissions

---

## 💾 Storage Issues

### Issue: "Cannot save workflow"

**Symptoms:**
- Save button doesn't work
- "Save failed" error
- Workflow not persisted

**Solutions:**
1. **Check workflow name** - ensure name is provided
2. **Verify storage directory** - ensure `workflows/` directory exists
3. **Check write permissions** - ensure directory is writable
4. **Review validation errors** - fix errors before saving
5. **Check network** - ensure API connection works

---

### Issue: "Workflow version conflicts"

**Symptoms:**
- "Version mismatch" error
- Cannot update workflow
- Concurrent modification error

**Solutions:**
1. **Reload workflow** - get latest version
2. **Re-apply changes** - redo your modifications
3. **Check version number** - ensure version is updated
4. **Review workflow metadata** - verify version format

---

## 📚 Getting More Help

### Review Documentation

1. **[VWMP User Guide](vwmp-user-guide.md)** - General usage guide
2. **[Best Practices Guide](best-practices.md)** - Design patterns and tips
3. **[Release Workflow Reference](release-workflow-reference.md)** - Step-by-step reference
4. **[Kanban Review Workflow Reference](kanban-review-workflow-reference.md)** - Step-by-step reference

### Check Execution Logs

**Execution logs contain:**
- Step-by-step execution details
- Error messages and stack traces
- Step outputs and intermediate results
- Timing information

**How to access:**
- Execution log panel opens during execution
- Review logs for each step
- Export logs for analysis
- Check findings files for results

### Debug Mode

**Enable debug logging:**
- Check FastAPI logs: `logs/fastapi.log`
- Enable DEBUG mode in FastAPI settings
- Review step handler logs
- Check browser console for frontend errors

---

## 🎯 Common Patterns

### Pattern: Fixing Dependency Issues

**Problem:** Step depends on non-existent step

**Solution:**
```yaml
# Check step dependencies
dependencies:
  - step-1  # ✅ Must exist
  - step-nonexistent  # ❌ Doesn't exist - remove this

# Fix: Remove invalid dependency or create missing step
```

---

### Pattern: Fixing Configuration Issues

**Problem:** Config variable not resolving

**Solution:**
```yaml
# Workflow config (must define variable)
config:
  my_variable: value

# Step config (reference with ${config.variable_name})
config:
  my_param: ${config.my_variable}  # ✅ Correct syntax
```

---

### Pattern: Fixing Type Errors

**Problem:** Type mismatch in parameters

**Solution:**
```yaml
# Wrong (string instead of integer)
config:
  threshold: "500"  # ❌ String

# Correct (integer)
config:
  threshold: 500  # ✅ Integer

# Or convert in handler:
threshold: int(config.get("threshold", 500))
```

---

## 📝 Reporting Issues

### What to Include

When reporting issues, include:

1. **Error message** - exact error text
2. **Steps to reproduce** - how to trigger the issue
3. **Workflow configuration** - workflow YAML (sanitized)
4. **Execution logs** - relevant log excerpts
5. **Environment** - Python version, FastAPI version, OS
6. **Browser** - if UI issue (browser, version)

### Where to Report

- **Create issue** in project repository
- **Include workflow YAML** (sanitized if sensitive)
- **Attach logs** and error messages
- **Describe expected behavior** vs actual behavior

---

## 🔗 Related Documentation

- **[VWMP User Guide](vwmp-user-guide.md)** - How to use VWMP
- **[Best Practices Guide](best-practices.md)** - Design patterns and recommendations
- **[Tutorial: Building Release Workflow](tutorial-release-workflow.md)** - Step-by-step tutorial
- **[Tutorial: Building Kanban Review Workflow](tutorial-kanban-review-workflow.md)** - Step-by-step tutorial

---

**Last Updated:** 2025-11-21
**Guide Version:** 1.0.0

