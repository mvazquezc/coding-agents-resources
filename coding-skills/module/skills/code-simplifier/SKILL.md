---
name: code-simplifier
description: Simplify and refine code for clarity, consistency, and maintainability while preserving all functionality.
---

Simplify and refine the following code. Apply the guidelines below.

Target: $ARGUMENTS

---

Guidelines adapted from [claude-plugins-official/code-simplifier](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/code-simplifier).

## 1. Preserve Functionality

**Never change what the code does -- only how it does it.**

- All original features, outputs, and behaviors must remain intact.
- Run existing tests (if available) before and after changes to confirm nothing broke.
- If you are unsure whether a change alters behavior, err on the side of not making it.

## 2. Apply Project Standards

**Follow the established coding standards of the project.**

Before making changes, check for project-level guidelines (e.g., CLAUDE.md, .editorconfig, linter configs, CONTRIBUTING.md). Apply what you find. When no project standards exist, follow language-idiomatic conventions:

- Use the project's preferred module system and import style.
- Match existing naming conventions (casing, prefixes, suffixes).
- Follow the project's error handling patterns.
- Respect existing formatting choices (indentation, line length, brace style).

## 3. Enhance Clarity

**Simplify code structure without sacrificing readability.**

- Reduce unnecessary complexity and nesting.
- Eliminate redundant code and abstractions.
- Improve readability through clear variable and function names.
- Consolidate related logic.
- Remove comments that describe obvious code.
- Avoid nested ternary operators -- prefer switch statements or if/else chains for multiple conditions.
- Choose clarity over brevity -- explicit code is often better than overly compact code.

## 4. Maintain Balance

**Avoid over-simplification that harms the code.**

Do not:
- Reduce code clarity or maintainability in the name of "simplification."
- Create overly clever solutions that are hard to understand.
- Combine too many concerns into single functions or components.
- Remove helpful abstractions that improve code organization.
- Prioritize "fewer lines" over readability (e.g., nested ternaries, dense one-liners).
- Make the code harder to debug or extend.

## 5. Focus Scope

**Only refine code that was explicitly requested.**

- If the user points to specific files or functions, limit changes to those.
- If no specific scope is given, focus on recently modified code in the current session.
- Do not "improve" adjacent code, comments, or formatting that wasn't asked about.
- If you notice unrelated issues, mention them -- don't fix them silently.

## Refinement Process

1. Identify the target code sections (from user input or recent modifications).
2. Read and understand the code's current behavior.
3. Analyze for opportunities to improve clarity and consistency.
4. Apply project-specific coding standards.
5. Make changes, ensuring all functionality remains unchanged.
6. Verify the refined code is simpler and more maintainable.
7. Summarize what was changed and why.
