# WFRIA Dry‑Run Validator

## Purpose

This dry‑run validator checks whether a project is ready for WFRIA
migration BEFORE any files are modified.

It prevents migration failures by detecting issues early.

------------------------------------------------------------------------

## Scan Scope

Scan project directories:

-   src/
-   components/
-   pages/
-   layouts/

Ignore:

-   node_modules
-   dist
-   build
-   coverage

------------------------------------------------------------------------

## Pre‑Migration Checks

### 1. Dependency Readiness

Verify:

    package.json

Must contain:

    @wf-wfria/pioneer-core

If missing:

DRYRUN_FAIL: WFRIA_NOT_INSTALLED

------------------------------------------------------------------------

### 2. File Eligibility

Eligible files:

    .js
    .jsx

Count:

-   total eligible files
-   invalid extension files

If zero eligible files:

DRYRUN_FAIL: NO_VALID_FILES

------------------------------------------------------------------------

### 3. JSX Detection

Each target file must contain JSX syntax.

If file contains no JSX:

SKIP_FILE: NO_JSX

------------------------------------------------------------------------

### 4. Unsupported Libraries Scan

Detect usage of disallowed UI libraries:

-   @mui
-   antd
-   bootstrap
-   chakra-ui
-   semantic-ui

If detected:

FLAG_LIBRARY_USAGE: `<library>`{=html}

(This does NOT fail dry run --- only warns)

------------------------------------------------------------------------

### 5. Mapping Coverage Check

Load:

    @wfria-components/

For each detected UI element:

If mapping exists → OK\
If mapping missing → FLAG:

    MISSING_MAPPING: <element>

------------------------------------------------------------------------

### 6. Root Component Detection

Detect entry components:

Possible roots:

-   App.jsx
-   index.jsx
-   main.jsx

If none found:

DRYRUN_FAIL: ROOT_NOT_FOUND

------------------------------------------------------------------------

### 7. Syntax Health Check

Ensure each file:

-   parses successfully
-   contains no syntax errors

If error:

    SYNTAX_ISSUE: <filename>

------------------------------------------------------------------------

### 8. Write Permission Check

Confirm system can create folder:

    MigratedFrontend/

If not writable:

    DRYRUN_FAIL: NO_WRITE_PERMISSION

------------------------------------------------------------------------

### 9. File Size Safety

Detect files exceeding safe size threshold:

Threshold: 500 KB

If exceeded:

    LARGE_FILE_WARNING: <filename>

------------------------------------------------------------------------

### 10. Circular Import Detection

If circular dependencies detected:

    CIRCULAR_DEPENDENCY: <fileA> ↔ <fileB>

------------------------------------------------------------------------

## Dry‑Run Report

Generate:

    dryrun-report.md

Format:

  Check   Status   Details
  ------- -------- ---------

Status values:

-   PASS
-   WARNING
-   FAIL

------------------------------------------------------------------------

## Overall Status Rule

If any FAIL occurs:

    DRYRUN_STATUS: FAILED

If only warnings:

    DRYRUN_STATUS: PASSED_WITH_WARNINGS

If no issues:

    DRYRUN_STATUS: READY_FOR_MIGRATION

------------------------------------------------------------------------

## Execution Order

Run checks strictly in this order:

    Dependencies
    → File Eligibility
    → JSX Detection
    → Library Scan
    → Mapping Coverage
    → Root Detection
    → Syntax
    → Permissions
    → File Size
    → Circular Imports
    → Report

------------------------------------------------------------------------

## Behavior Rules

Validator must:

-   not modify files
-   not attempt fixes
-   not suggest code

It is a detection-only system.

------------------------------------------------------------------------

## Authority Rule

This document governs pre‑migration validation and overrides conflicting
preparation instructions.
