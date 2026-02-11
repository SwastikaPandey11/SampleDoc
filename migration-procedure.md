# WFRIA Migration Procedure

## Objective

Define the exact execution flow that must be followed to migrate
existing `.js` / `.jsx` frontend files to WFRIA-based components and
store results safely.

------------------------------------------------------------------------

## Step-by-Step Execution Flow

### Step 1 --- Verify WFRIA Installation

Check whether the dependency exists in:

    package.json

Look for:

    @wf-wfria/pioneer-core

**If not present** → Stop processing\
→ Return error:

    WFRIA_NOT_INSTALLED

Do NOT proceed with migration.

------------------------------------------------------------------------

### Step 2 --- Scan Target Files

Process only frontend files:

Allowed:

    .js
    .jsx

Ignore:

    .test.js
    .spec.js
    node_modules
    dist
    build

------------------------------------------------------------------------

### Step 3 --- Load Mapping Rules

Load all component mapping definitions from:

    @wfria-components/

If mapping folder missing:

    COMPONENT_MAPPING_NOT_FOUND

Stop execution.

------------------------------------------------------------------------

### Step 4 --- Parse Source File

For each `.js/.jsx` file extract:

-   JSX elements
-   imported UI libraries
-   custom UI components
-   HTML elements used as UI

------------------------------------------------------------------------

### Step 5 --- Replace Components

  Condition            Action
  -------------------- ------------------------------
  Mapping exists       Replace with WFRIA component
  Mapping missing      Keep original code
  Invalid WFRIA prop   Do not replace

Never modify: - logic - hooks - state - functions - APIs

Only UI layer may change.

------------------------------------------------------------------------

### Step 6 --- Ensure Imports

If WFRIA component used but import missing:

    import { ComponentName } from "@wf-wfria/pioneer-core";

If duplicate imports exist → merge them.

------------------------------------------------------------------------

### Step 7 --- Add ThemeProvider Wrapper

If root component is not wrapped:

    <ThemeProvider>
       existing JSX
    </ThemeProvider>

Apply only once per file.

------------------------------------------------------------------------

### Step 8 --- Create Output Folder

Create:

    MigratedFrontend/

If already exists → reuse it.\
Never delete contents.

------------------------------------------------------------------------

### Step 9 --- Write Migrated Files

Save to:

    MigratedFrontend/<same-relative-path>

Example:

    src/pages/Home.jsx
    →
    MigratedFrontend/src/pages/Home.jsx

Never overwrite originals.

------------------------------------------------------------------------

### Step 10 --- Validation Pass

Before saving file confirm:

✔ JSX syntax valid\
✔ imports valid\
✔ components exist in mapping docs\
✔ no undefined components\
✔ no removed logic

If validation fails:

    FILE_VALIDATION_FAILED: <filename>

Skip file.

------------------------------------------------------------------------

### Step 11 --- Output Summary Report

Generate:

    migration-report.md

Format:

  File   Status   Notes
  ------ -------- -------

Statuses: - MIGRATED - SKIPPED - FAILED

------------------------------------------------------------------------

## Strict Safety Rules

Never: - refactor code - optimize logic - rename variables - change
formatting style - convert JS ↔ TS

Only replace UI components.

------------------------------------------------------------------------

## Execution Priority Order

    Install Check
    → File Scan
    → Load Mapping
    → Parse JSX
    → Replace Components
    → Fix Imports
    → Wrap ThemeProvider
    → Save File
    → Validate
    → Report

Do NOT skip steps.

------------------------------------------------------------------------

## Failure Policy

If any critical step fails:

Stop processing immediately.\
Do not partially migrate project.

------------------------------------------------------------------------

## Completion Condition

Migration is successful only when:

-   all valid files processed
-   report generated
-   no validation errors

------------------------------------------------------------------------

## Usage

Reference in instructions:

    Follow migration workflow:
    @migration-procedure.md

------------------------------------------------------------------------

## Enforcement Rule

Execution must strictly follow this document in exact order.
