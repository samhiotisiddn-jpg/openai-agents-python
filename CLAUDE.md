# Contributor Guide

This guide helps new contributors get started with the OpenAI Agents Python repository. It covers repo structure, how to test your work, available utilities, and guidelines for commits and PRs.

**Location:** `AGENTS.md` at the repository root.

## Table of Contents

1. [Policies & Mandatory Rules](#policies--mandatory-rules)
2. [Project Structure Guide](#project-structure-guide)
3. [Operation Guide](#operation-guide)

## Policies & Mandatory Rules

### Mandatory Skill Usage

#### `$code-change-verification`

Run `$code-change-verification` before marking work complete when changes affect runtime code, tests, or build/test behavior.

Run it when you change:
- `src/agents/` (library code) or shared utilities.
- `tests/` or add or modify snapshot tests.
- `examples/`.
- Build or test configuration such as `pyproject.toml`, `Makefile`, `mkdocs.yml`, `docs/scripts/`, or CI workflows.

You can skip `$code-change-verification` for docs-only or repo-meta changes (for example, `docs/`, `.agents/`, `README.md`, `AGENTS.md`, `.github/`), unless a user explicitly asks to run the full verification stack.

#### `$openai-knowledge`

When working on OpenAI API or OpenAI platform integrations in this repo (Responses API, tools, streaming, Realtime API, auth, models, rate limits, MCP, Agents SDK or ChatGPT Apps SDK), use `$openai-knowledge` to pull authoritative docs via the OpenAI Developer Docs MCP server (and guide setup if it is not configured).

#### `$implementation-strategy`

Before changing runtime code, exported APIs, external configuration, persisted schemas, wire protocols, or other user-facing behavior, use `$implementation-strategy` to decide the compatibility boundary and implementation shape. Judge breaking changes against the latest release tag, not unreleased branch-local churn.

#### `$pr-draft-summary`

When a task in this repo finishes with moderate-or-larger code changes, invoke `$pr-draft-summary` in the final handoff to generate the required PR summary block, branch suggestion, title, and draft description.

### Public API Positional Compatibility

Treat the parameter and dataclass field order of exported runtime APIs as a compatibility contract.

- For public constructors (e.g., `RunConfig`, `FunctionTool`, `AgentHookContext`), preserve existing positional argument meaning.
- When adding a new optional public field/parameter, append it to the end.
- If reordering is unavoidable, add an explicit compatibility layer and regression tests.

## Project Structure Guide

### Overview

The OpenAI Agents Python repository provides the Python Agents SDK, examples, and documentation built with MkDocs. Use `uv run python ...` for Python commands.

### Repo Structure & Important Files

- `src/agents/` — Core library implementation.
- `tests/` — Test suite; see `tests/README.md` for snapshot guidance.
- `examples/` — Sample projects showing SDK usage.
- `docs/` — MkDocs documentation source; do not edit translated docs under `docs/ja`, `docs/ko`, or `docs/zh`.
- `docs/scripts/` — Documentation utilities, including translation and reference generation.
- `mkdocs.yml` — Documentation site configuration.
- `Makefile` — Common developer commands.
- `pyproject.toml`, `uv.lock` — Python dependencies and tool configuration.
- `.github/PULL_REQUEST_TEMPLATE/pull_request_template.md` — PR template.
- `site/` — Built documentation output.

### Agents Core Runtime Guidelines

- `src/agents/run.py` is the runtime entrypoint (`Runner`, `AgentRunner`). Keep it focused on orchestration. Put new runtime logic under `src/agents/run_internal/`.
- Keep streaming and non-streaming paths behaviorally aligned.
- Input guardrails run only on the first turn and only for the starting agent.
- If the serialized RunState shape changes, update `CURRENT_SCHEMA_VERSION` in `src/agents/run_state.py`.

## Operation Guide

### Prerequisites

- Python 3.10+.
- `uv` installed for dependency management and `uv run` for Python commands.
- `make` available to run repository tasks.

### Development Workflow

1. Sync with `main` and create a feature branch:
   ```bash
   git checkout -b feat/<short-description>
   ```
2. If dependencies changed or you are setting up the repo, run `make sync`.
3. Implement changes and add or update tests alongside code updates.
4. Build docs when you touch documentation:
   ```bash
   make build-docs
   ```
5. When `$code-change-verification` applies, run it before marking work complete.
6. Commit with concise, imperative messages; keep commits small and focused.

### Testing & Automated Checks

#### Unit tests and type checking

```bash
make tests          # full test suite
uv run pytest -s -k <pattern>   # focused test
make typecheck      # type checking
```

#### Snapshot tests

```bash
make snapshots-fix     # fix existing snapshots
make snapshots-create  # create new snapshots
```

#### Coverage

```bash
make coverage   # fails if coverage drops below threshold
```

#### Formatting, linting, and type checking

```bash
make format   # apply fixes
make lint     # check only
make typecheck
```

#### Mandatory local run order

```bash
make format
make lint
make typecheck
make tests
```

### Utilities & Tips

```bash
make sync            # install/refresh dev dependencies
make build-docs      # build docs
make serve-docs      # preview docs locally
make build-full-docs # run translations and build
```

### Pull Request & Commit Guidelines

- Use the template at `.github/PULL_REQUEST_TEMPLATE/pull_request_template.md`.
- Add tests for new behavior when feasible.
- Run `make format`, `make lint`, `make typecheck`, and `make tests` before marking work ready.
- Commit messages: concise, imperative mood, small and focused commits.

### Review Process

- ✅ Checks pass (`make format`, `make lint`, `make typecheck`, `make tests`).
- ✅ Tests cover new behavior and edge cases.
- ✅ Code is readable and consistent with existing style.
- ✅ Public APIs and user-facing behavior changes are documented.
- ✅ Examples are updated if behavior changes.
- ✅ History is clean with a clear PR description.
