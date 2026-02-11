# WFRIA Migration Validator

## Purpose

This validator verifies that migrated frontend files strictly comply
with WFRIA migration rules and procedure.

It must be executed after migration and before deployment.

------------------------------------------------------------------------

## Validation Scope

Validate all files inside:

    MigratedFrontend/

Do NOT validate original source directory.

------------------------------------------------------------------------

## Validation Checklist

### 1. Dependency Check

Confirm project contains:

    @wf-wfria/pioneer-core

If missing → FAIL

------------------------------------------------------------------------

### 2. Import Validation

For every migrated file:

✔ Only allowed UI import source:

    @wf-wfria/pioneer-core

FAIL if found:

-   material-ui
-   antd
-   bootstrap
-   chakra
-   semantic-ui

------------------------------------------------------------------------

### 3. Component Validation

For each JSX component used:

Rule:

Component must exist in:

    @wfria-components/

If component not documented → FAIL

------------------------------------------------------------------------

### 4. Prop Validation

For each WFRIA component:

✔ Props must match documentation

If invalid prop detected:

    INVALID_PROP: <component>.<prop>

------------------------------------------------------------------------

### 5. ThemeProvider Rule

Each root file must contain:

    <ThemeProvider>

Conditions:

  Case                Result
  ------------------- --------
  Missing             FAIL
  Multiple wrappers   FAIL
  Single wrapper      PASS

------------------------------------------------------------------------

### 6. Logic Integrity Check

Compare migrated file with original file.

Ensure no changes in:

-   variables
-   functions
-   conditions
-   loops
-   hooks
-   API calls

If difference detected:

    LOGIC_MODIFIED: <filename>

------------------------------------------------------------------------

### 7. UI Replacement Validation

For each replaced element:

✔ Mapping must exist in component mapping docs.

If replacement exists without mapping:

    UNAUTHORIZED_REPLACEMENT: <element>

------------------------------------------------------------------------

### 8. File Location Validation

All migrated files must exist inside:

    MigratedFrontend/

If file written elsewhere → FAIL

------------------------------------------------------------------------

### 9. Overwrite Protection

Confirm original files were NOT modified.

If original file timestamp changed → FAIL

------------------------------------------------------------------------

### 10. Syntax Validation

Each migrated file must:

-   compile
-   parse JSX correctly
-   contain no undefined imports

If error:

    SYNTAX_ERROR: <filename>

------------------------------------------------------------------------

## Validation Output Report

Generate file:

    validation-report.md

Format:

  File   Result   Issue
  ------ -------- -------

Results:

-   PASS
-   FAIL

------------------------------------------------------------------------

## Global Failure Rule

If any file FAILS:

Overall validation status = FAILED

Deployment must be blocked.

------------------------------------------------------------------------

## Strict Enforcement Mode

Validator must behave as:

> deterministic compiler

It must NOT:

-   auto-fix code
-   suggest improvements
-   modify files

Only detect violations.

------------------------------------------------------------------------

## Execution Order

Always validate in this sequence:

    Dependencies
    → Imports
    → Components
    → Props
    → ThemeProvider
    → Logic Integrity
    → UI Mapping
    → File Location
    → Overwrite Check
    → Syntax
    → Report

Do NOT skip steps.

------------------------------------------------------------------------

## Final Output Requirement

Validator must output ONLY:

-   validation-report.md
-   overall status line

Example:

    VALIDATION_STATUS: FAILED

No explanations.

------------------------------------------------------------------------

## Authority Rule

This validator document overrides any conflicting instruction files.

Compliance is mandatory.
