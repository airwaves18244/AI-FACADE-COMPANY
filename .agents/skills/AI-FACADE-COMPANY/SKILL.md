```markdown
# AI-FACADE-COMPANY Development Patterns

> Auto-generated skill from repository analysis

## Overview
This skill provides guidance on the development patterns and conventions used in the AI-FACADE-COMPANY TypeScript codebase. It covers file organization, code style, import/export practices, and testing patterns to ensure consistency and maintainability for contributors. While no specific frameworks or automated workflows were detected, this skill will help you align with the repository's established practices.

## Coding Conventions

### File Naming
- Use **snake_case** for all file names.
  - **Example:**  
    ```
    user_profile.ts
    data_processor.test.ts
    ```

### Import Style
- Use **relative imports** for referencing modules within the project.
  - **Example:**  
    ```typescript
    import { processData } from './data_processor';
    import { UserProfile } from '../models/user_profile';
    ```

### Export Style
- Use **named exports** rather than default exports.
  - **Example:**  
    ```typescript
    // In data_processor.ts
    export function processData(input: string): string {
      // ...
    }

    // In another file
    import { processData } from './data_processor';
    ```

### Commit Patterns
- Commit messages are freeform, with no enforced prefix.
- Average commit message length is 81 characters.

## Workflows

_No automated workflows were detected in this repository. All processes are manual._

## Testing Patterns

- **Test Framework:** Not explicitly detected; ensure to follow existing patterns or consult maintainers.
- **Test File Naming:** Test files use the `*.test.*` pattern and follow snake_case.
  - **Example:**  
    ```
    user_profile.test.ts
    ```
- **Test Placement:** Tests are typically placed alongside the files they test or in a dedicated test directory.

## Commands

| Command | Purpose |
|---------|---------|
| /new-file | Create a new TypeScript file using snake_case naming and named exports |
| /add-test | Add a test file using the *.test.* pattern for a given module |
| /import-module | Import a module using relative import syntax |
| /export-named | Export functions or variables using named exports |

```