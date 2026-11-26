# Common Formatting Rules

These rules apply to code files, test files, snippets, and examples.

## Language and project conventions

- Follow the language's formatting best practices.
- Follow the project's formatter or linter configuration when one exists.
- Use the shared limits below when the project has no explicit configuration for those limits.
- Treat language examples as applications of these rules and the project's conventions.

## Logical groups

Group lines by the task they perform.
Separate each group with one empty line.

## Length limits

| Measure | Review threshold | Upper limit | Applies to |
| --- | --- | --- | --- |
| Line length | 80 characters | 120 characters | Production and test code |
| File length | 300 lines | 500 lines | Production code |
| Function length | 60 lines | 120 lines | Production code |

Crossing a review threshold requires checking readability and cohesion.
Modest growth above the threshold is acceptable when the code retains a clear responsibility.
For example, a cohesive production file can grow from 300 to 320 lines.
Split production files and functions by responsibility before exceeding their upper limits.
Preserve error handling, validation, and cleanup during extraction.

If a production file exceeds 500 lines before the task starts, get explicit user permission before refactoring it to meet the limit.
If a production function exceeds 120 lines before the task starts, get explicit user permission before refactoring it to meet the limit.
Without that permission, keep edits focused on the requested task.

Count physical lines after formatting, including comments and empty lines.
Measure a function from its declaration through its closing delimiter.
Function limits apply to executable functions and methods.
Review HTML elements, CSS rules, and Liquid blocks through logical grouping and file responsibility.
Use the project's formatter or linter to measure line width when available.
Include indentation when checking displayed line width manually.
Preserve logical-group spacing when reducing size.

### Line wrapping

Wrap long lines in changed code as needed without asking for permission.
Keep readable calls, parameter lists, and expressions on one line when they fit the 80-character target.
Review longer lines and wrap at clear syntax boundaries where this improves readability.
Indent each continuation level and keep ordinary code within 120 characters.

Preserve a line when the project formatter requires that layout.
Preserve a line when breaking it would change meaning, such as a significant string literal, URL, or import path.

## Reusable helpers

Search for existing project helpers and platform APIs before writing another implementation.
Use the existing function when its behavior matches the task, including standard trimming functions.
Place custom general-purpose normalization, required-field checks, and password operations in focused shared modules.
Follow the project's existing utility organization and dependency direction.
Keep behavior specific to signup, such as `validateSignup`, with signup code.
Extract shared behavior into a helper with a clear purpose and contract.

## Test code

Apply these rules equally to production and test code.
Helper functions, mocks, and stubs remain code and receive the same formatting care.
Test files, test functions, and test-only helpers are exempt from file and function length limits.
Line-length limits and logical-group spacing still apply to test code.

- Separate setup, action, and assertions with one empty line.
- Group lines within each phase by their task.
- Break long fixtures, test data, and assertion chains across correctly indented lines.
- Keep style consistent across all tests in the file.
- Apply the same care to helper bodies and embedded SQL, GraphQL, or HTML.
- Extract repeated setup, teardown, fixture creation, and assertion logic into named helpers when the behavior is shared.
- Keep each test's inputs, action, and expected outcomes clear at the call site.
- Preserve resource cleanup when consolidating setup or teardown.

Separate the phases present in a test. An assertion can also perform the action.
The existing language and formatter exceptions still apply.

## Embedded code

Format SQL, GraphQL, HTML, and CSS inside strings with the same care as standalone code.
Put each clause, field, or element on its own line and indent each level.
Use the language's multiline string form.
Preserve behavior when whitespace is significant.

Read [SQL formatting](sql.md) when the code contains SQL queries.
Read [HTML formatting](html.md) or [CSS formatting](css.md) when changing embedded markup or styles.
Preserve significant text whitespace, selector relationships, and declaration order during layout changes.
