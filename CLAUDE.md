# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Constitutions (Governance Rules)

All policies live in `constitutions/`. The **CORE_CONSTITUTION.md** is always active; others are loaded on-demand.

### TDD Workflow (mandatory)

Follow strict **RED → GREEN → PAUSE → REFACTOR** cycle:

| Phase | Allowed files | Rule |
|-------|--------------|------|
| RED | Test files only | Write the failing test; no production code |
| GREEN | Production code only | Minimal implementation to pass the test |
| REFACTOR | Production code only | Only with explicit user approval |

Run tests after **every** step. Make the smallest change possible.

### Commit Discipline

- Prefix every commit: `[RED]`, `[GREEN]`, or `[REFACTOR]`
- Atomic, single-purpose commits only — no multi-feature commits

### Refactoring Gate

- **Never auto-refactor.** Always ask the user first.
- Before refactoring, ask: **Local** (current file) or **Global** (cross-file)?
- Present a prioritized list of code smells and wait for confirmation.
- Cross-file changes require rationale + impact/pros/cons + approval.

### Architecture Constraints

- Domain-first, minimal abstraction
- No speculative design, no flexibility without an explicit requirement
- Simplicity over extensibility — delay layering decisions

### Code Quality

- No duplication, no dead code, no hidden dependencies
- Small functions, single responsibility, clear naming

### Restrictions

- Do not read files listed in `.gitignore`
