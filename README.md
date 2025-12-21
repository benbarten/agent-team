# Cursor Agents

A collection of specialized AI agents for Cursor IDE. Invoke them by name to get focused assistance for different development tasks.

## Agents

| Agent | Invoke | Purpose |
|-------|--------|---------|
| **Planner** | `@planner` | Feature planning, architecture design, implementation plans |
| **Builder** | `@builder` | TDD implementation, production-ready code |
| **Reviewer** | `@reviewer` | Code review, security analysis, bug detection |
| **Tester** | `@tester` | Test generation, coverage improvement |
| **Documenter** | `@documenter` | READMEs, API docs, code comments |
| **Releaser** | `@releaser` | Changelogs, version management, release prep |
| **Debugger** | `@debugger` | Root cause analysis, error diagnosis, fixes |

## Usage

Copy the `.cursor/rules/` directory into your project. Then mention any agent in Cursor chat:

```
@planner I need to add user authentication to this app
```

Agents can hand off to each other—a planner might suggest invoking `@builder` when the design is ready.

