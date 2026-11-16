---
name: coding-guideline
description: Use when writing, modifying, or reviewing code, including production code, tests, fixtures, mocks, stubs, test helpers, and embedded SQL, GraphQL, or HTML.
---

# Coding Guideline

Apply the same formatting standard to production code and test code.
Test fixtures, mocks, stubs, helpers, and assertions receive the same care as implementation code.

Follow applicable `AGENTS.md` instructions and project formatter or linter configuration.
The references explain formatting, size, and reuse requirements.
Examples demonstrate these rules and the project's language conventions.

## Select references before writing

Always read [common rules](references/common.md).
Read every additional reference that applies to the code being changed.

| Code being changed | Additional reference |
| --- | --- |
| PHP | [PHP](references/php.md) |
| JavaScript or TypeScript | [JavaScript and TypeScript](references/javascript-typescript.md) |
| Go | [Go](references/go.md) |
| SQL, including queries inside another language | [SQL](references/sql.md) |
| HTML, including generated or embedded markup | [HTML](references/html.md) |
| CSS, including embedded styles | [CSS](references/css.md) |
| Shopify Liquid templates | [Shopify Liquid](references/shopify-liquid.md), plus HTML for markup changes |
| Dart or Flutter | [Dart and Flutter](references/dart-flutter.md) |
| WordPress PHP | [PHP](references/php.md) and [WordPress PHP](references/wordpress-php.md) |

Selection is cumulative. A PHP test containing SQL requires common, PHP, and SQL guidance.
Common rules include test formatting and size exemptions.
Read platform-specific sections only when the code uses that platform.
For example, WordPress PHP requires common, PHP, and WordPress guidance.
Shopify markup requires common, Liquid, and HTML guidance, plus any embedded language being changed.
For other languages, apply common rules and the project's language conventions.
Add HTML or CSS guidance when changing markup or styles embedded in another language.
Common rules also cover embedded GraphQL.

## Review before completion

- Check changed production and test code against the applicable references.
- Check logical groups, blank lines, long lines, fixtures, assertion chains, and embedded code.
- Check test setup, action, and assertion separation.
- Review helper bodies with the same formatting care.
- Review line lengths against the 80-character target and 120-character upper limit.
- Review production files against 300/500 lines and production functions against 60/120 lines.
- Apply test size exemptions while checking repeated setup, teardown, fixtures, and assertions for helper extraction.
- Reuse existing utilities and platform APIs, and keep custom general helpers in focused shared modules.
- Check formatter requirements and whitespace-sensitive behavior before changing layout.

Passing tests does not establish formatting compliance.
