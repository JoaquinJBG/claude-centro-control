---
name: conventional-commits
description: Use when writing a git commit message, running git commit, or amending/wording a commit - enforces the Conventional Commits 1.0.0 format (type, optional scope, description, body, footer, breaking changes)
---

# Conventional Commits

## Overview

Every commit message follows a structured format that conveys intent to humans and tooling (changelog generation, SemVer bumps). Source: https://www.conventionalcommits.org/

**Structure:**

```
<type>[optional scope][!]: <description>

[optional body]

[optional footer(s)]
```

- Type + colon + space, then a short imperative description, all on the first line.
- Body: one blank line after the description, free-form, explains *what* and *why* (not *how*).
- Footer(s): one blank line after the body. `Token: value` pairs (`Refs: #123`, `Reviewed-by: ...`) or `BREAKING CHANGE: ...`.

## Types

| Type | Use for |
|------|---------|
| `feat` | A new feature (correlates with MINOR in SemVer) |
| `fix` | A bug fix (correlates with PATCH in SemVer) |
| `docs` | Documentation-only changes |
| `style` | Formatting, whitespace, semicolons — no code-meaning change |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `perf` | Performance improvement |
| `test` | Adding or correcting tests |
| `build` | Build system or external dependency changes (npm, pip, webpack, ...) |
| `ci` | CI configuration / scripts (GitHub Actions, GitLab CI, ...) |
| `chore` | Other changes that don't touch src or tests |
| `revert` | Reverts a previous commit |

## Scope

Optional noun in parentheses describing the section of the codebase: `feat(parser):`, `fix(api): ...`. Keep it short and consistent with scopes already used in the repo.

## Breaking changes

Signal a breaking change **either or both** ways:

1. `!` after the type/scope: `feat(api)!: drop support for Node 14`
2. A footer: `BREAKING CHANGE: <description of the API change>`

A `BREAKING CHANGE` footer (or `!`) correlates with a MAJOR SemVer bump and can appear on **any** type, not just `feat`.

## Rules

- Description: imperative mood, lowercase start, no trailing period. ("add", not "added"/"adds".)
- Use the body for context that doesn't fit in the description — wrap at ~72 chars.
- One logical change per commit; if you need "and" in the description, consider splitting.
- Footers reference issues/links/metadata: `Closes #13`, `Refs: ABC-123`.

## Examples

```
feat: allow provided config object to extend other configs
```

```
fix(parser): handle empty input without throwing

Previously an empty string reached the tokenizer and produced an
unhelpful stack trace. Return an empty AST instead.

Closes #221
```

```
refactor!: replace positional args with an options object

BREAKING CHANGE: createClient(host, port) is now createClient({ host, port }).
```

```
chore(deps): bump django from 4.2.11 to 4.2.13
```

## Project-specific conventions

This skill defines the *format*. Repo-specific rules (language of the message, trailers to include or omit, allowed scopes) live in each project's CLAUDE.md / memory and take precedence. If the project says nothing, write the message in the language the repo's history already uses.
