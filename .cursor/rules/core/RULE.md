---
description: Core project standards and conventions. Always attached to provide foundational context.
globs:
alwaysApply: true
---

# Core Development Standards

## Commit Messages

Use conventional commits format:
```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`

## Error Handling Philosophy

- Errors are values, not exceptions (especially in Go)
- Always provide context when wrapping errors
- Log at the point of handling, not at every level
- Return early on errors, keep happy path unindented

## Logging Standards

- Use structured logging (JSON in production)
- Log levels: DEBUG, INFO, WARN, ERROR
- Include correlation IDs for request tracing
- Never log sensitive data (passwords, tokens, PII)
- Look for existing logging packages and best practices to adapt
- Do not reinvent the wheel if a standard is already established.

## Code Organization

- Keep functions small and focused (single responsibility)
- Prefer composition over inheritance
- Write code for readability first, optimize when measured
- Delete dead code, don't comment it out

## Quality Gates

Before considering work complete:
1. Code compiles/lints without warnings
2. Tests pass
3. No hardcoded secrets or credentials
4. Error cases are handled

## Available Agents

Invoke these agents for specialized tasks:
- `@planner` - Feature planning and architecture design
- `@builder` - TDD implementation
- `@reviewer` - Code review and security analysis
- `@tester` - Test generation and coverage
- `@documenter` - Documentation and comments
- `@releaser` - Release management and changelogs
- `@debugger` - Root cause analysis and debugging
- `@challenger` - First principles thinking and decision documentation

