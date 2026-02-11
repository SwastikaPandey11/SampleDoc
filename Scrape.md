You are a senior React architect and documentation generator.

Your task is to analyze the entire repository codebase and generate a structured Markdown documentation file that describes all reusable components in this framework so that another AI model can use this documentation to migrate applications into this framework.

Follow these instructions strictly.

--------------------------------------------------

STEP 1 — Repository Scan
Scan all files recursively and detect:

• React components (.tsx, .jsx, .js)
• Shared UI components
• Layout components
• Utility components
• Hooks
• Context Providers
• Theme / styling system
• Routing wrappers
• Form components
• Data display components
• Config driven components

Ignore:
• test files
• storybook files
• build files
• configs unrelated to UI usage

--------------------------------------------------

STEP 2 — Component Analysis

For EACH component, extract:

1. Component Name
2. File path
3. Purpose (1–2 lines)
4. Props table:
   - prop name
   - type
   - required or optional
   - default value
   - description (infer from code usage)
5. Example usage snippet
6. Dependencies used
7. Styling method (css / styled / tailwind / theme / inline)
8. Variants if any
9. Composition rules (can it wrap children? supports render props?)
10. Accessibility support if visible
11. Common patterns where it is used

If prop types are not explicitly declared, infer them intelligently.

--------------------------------------------------

STEP 3 — Categorization

Group components into sections:

# Layout Components
# Form Components
# Display Components
# Navigation
# Feedback Components
# Utility Components
# Hooks
# Providers
# Theming System
# Internal Only Components

--------------------------------------------------

STEP 4 — Migration Intelligence Notes

For each component add:

**Migration Notes**
Explain when this component should be used instead of standard HTML or third-party libraries.

Example:
Use FrameworkButton instead of <button> when:
• you need loading state
• need built-in permission handling
• want consistent theme styles

--------------------------------------------------

STEP 5 — Framework Rules Section

At top of document generate:

# Framework Usage Rules

Include:
• naming conventions
• styling conventions
• state management approach
• data fetching pattern
• form handling standard
• validation method
• error handling pattern
• folder structure rules
• component composition philosophy

--------------------------------------------------

STEP 6 — AI Migration Guide Section

Add a section titled:

# Instructions for AI Models Migrating Code

Explain:

How the model should map generic React code into this framework:

Example rules:

IF plain HTML button → use FrameworkButton  
IF form input → use FrameworkInput  
IF modal logic → use FrameworkModal  

Provide mapping table.

--------------------------------------------------

STEP 7 — Output Format

Output MUST be a single Markdown file.

Structure exactly like:

# Framework Name Documentation

## Overview
## Usage Rules
## Component Categories
   ### Component Name
      Props Table
      Usage
      Notes
      Migration Notes
## Hooks
## Providers
## Mapping Guide for Migration AI

Use tables wherever possible.
Use clean markdown.
Do not skip any component.
Do not summarize components.
Be exhaustive.

--------------------------------------------------

STEP 8 — Quality Rules

Documentation must be:

• deterministic
• structured
• machine readable
• human readable
• consistent formatting
• no speculation
• no missing props
• no vague descriptions

If something is unclear from code, mark:

> Unable to infer from source

--------------------------------------------------

FINAL OUTPUT REQUIREMENT

Return ONLY the markdown file.
No explanations.
No commentary.
No analysis text.
