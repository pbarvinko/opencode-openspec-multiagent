# OpenCode OpenSpec Multi-Agent

A multi-agent orchestrated development workflow that uses [OpenSpec](https://openspec.dev) as the source of truth for product and architecture requirements. Agents collaborate in a structured loop — from specification through implementation to review — keeping every change grounded in a written spec before the implementation starts.

## Agents

| Agent | Mode | Role |
|---|---|---|
| [@architect](opencode/agents/architect.md) | primary | Drives the full delivery loop: clarifies requirements, creates OpenSpec specifications, delegates implementation, and evaluates review output. Never writes code itself. |
| [@developer](opencode/agents/developer.md) | subagent | Implements the spec assigned by `@architect` using the `opsx-apply` skill. Runs all checks before reporting completion. |
| [@code-reviewer](opencode/agents/code-reviewer.md) | subagent | Reviews every change against the OpenSpec change definition. Issues actionable change requests or approves; cannot modify code itself. |
| [@repo-scout](opencode/agents/repo-scout.md) | subagent | Scans the repository and maintains `ARCHITECTURE.md` with detected stack, conventions, linting/test commands, and structural hotspots. |

## Workflow

```
User ──► @architect
             │
             ├─ calls @repo-scout (first run or on repo change)
             │
             ├─ guides user through opsx-explore / opsx-propose
             │   to build the OpenSpec specification
             │
             │  [on explicit user approval]
             │
             ├─► @developer  (opsx-apply)
             │       │
             │       └─► reports completion
             │
             ├─► @code-reviewer
             │       │
             │       ├─ change requests ──► @developer (loop)
             │       │
             │       └─ approved ──► @architect
             │
             └─► summary to user + opsx-archive suggestion
```

1. **Discovery** — `@architect` inspects the repo (via `@repo-scout` if needed) and collaborates with the user to define an OpenSpec specification using the `opsx-explore` and `opsx-propose` skills.
2. **Approval gate** — implementation only starts after the user explicitly approves.
3. **Implementation** — `@developer` applies the spec (`opsx-apply`), follows existing repo conventions, adds tests, and validates with linters/type-checkers before reporting back.
4. **Review** — `@code-reviewer` reads the OpenSpec change, runs `git diff`, checks correctness, security, and test coverage, and either requests changes or approves.
5. **Loop** — steps 3–4 repeat until `@code-reviewer` approves and `@architect` is satisfied.
6. **Archive** — `@architect` suggests archiving the change with `opsx-archive` and returns control to the user.

## How to Use

### Prerequisites

- [OpenCode](https://opencode.ai) installed
- [OpenSpec](https://openspec.dev) installed

### Setup

Copy the agent definitions to the OpenCode agents directory and restart OpenCode.

**macOS / Linux**
```bash
cp opencode/agents/* ~/.config/opencode/agents/
```

**Windows**
```powershell
Copy-Item opencode\agents\* "$env:APPDATA\opencode\agents\"
```

### Start

If it is a new project, where you have not used openspec yet, run `openspec init` to set it up before launching opencode.

Open a new OpenCode session in your project directory and describe the change you want to make to `@architect`.

## Credits

This agent setup is based on the workflow described in [How I Write Software With LLMs](https://www.stavros.io/posts/how-i-write-software-with-llms/) by Stavros Korokithakis.
