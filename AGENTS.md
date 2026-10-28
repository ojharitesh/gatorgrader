# AGENTS.md

This document provides guidelines for AI agents contributing to this
repository.

## Role and Project Knowledge

You are a coding agent contributing to **GatorGrader**, an automated grading
tool that checks student writing and source code (file existence, word and
paragraph counts, comments, fragments and regex, command output, Git commits).
It is often driven by GatorGrade via `gatorgrade.yml`, and can also run as a
standalone CLI.

- **Tech stack:** Python `>=3.10,<4.0` (CI covers 3.10 through 3.14); **uv**
  and **hatchling**; **taskipy** for tasks; **pytest** and **pytest-cov**;
  **ruff**; **bandit**, **radon**, and **xenon**. Runtime libraries include
  Typer, GitPython, pluginbase, requests, commonmark, and num2words.
- **Layout:**
  - `gator/` — main package
  - `gatorgrader.py` — CLI entry (`gatorgrader` script)
  - `gator/checks/check_*.py` — check plugins (`get_parser`, `parse`, `act`)
  - `tests/` and `tests/checks/` — tests mirroring the package
  - `pyproject.toml` — dependencies and taskipy tasks
  - `scripts/tasks.py` — legacy Invoke/pyenv multi-version helpers

Prefer [`README.md`](README.md) for developer setup and contributor
commands. Treat this file and the README as the source of truth for
agents; do not rely on other contributor docs.

## Build, Lint, and Test Commands

Put these commands to use early and often. You have permission to run all
commands in this file.

- **Install dependencies:** `uv sync`
- **Run all tasks (required gate):** `uv run task all`
  (runs `lint` then `test` then `cover`)
- **Run all linters:** `uv run task lint`
- **Format code:** `uv run task format` (fix),
  `uv run task format-check` or `uv run task lint-ruff-format` (check)
- **Lint pieces:** `uv run task lint-ruff`,
  `uv run task lint-bandit`, `uv run task lint-radon-cc`,
  `uv run task lint-radon-mi`, `uv run task lint-xenon`
- **Test all:** `uv run task test` (`pytest -x`)
- **Test with coverage:** `uv run task cover` or
  `uv run task test-coverage` (branch coverage on `gator`, fail-under **99**)
- **Run a single test:**
  `uv run pytest tests/test_file.py::test_function`
- **Multi-Python (preferred):** `uv python pin <version>` then
  `uv run task test` or `uv run task cover`
- **Multi-Python (legacy):** [`scripts/tasks.py`](scripts/tasks.py) wraps the
  same taskipy commands via Invoke and pyenv; prefer `uv python pin` for
  day-to-day work

There is **no** mypy, ty, pyrefly, `ccl`, or markdownlint taskipy task in this
repository. Do not invent those gates.

## Overview of Instructions

- **Always use `uv`:** Use `uv` for dependency management, virtual
  environments, and task running. Do not use `pip` or `venv` directly.
- **Follow these guidelines:** This file is the operating manual for agents in
  this repository. Follow it strictly.
- **Verify your changes:** Before claiming work is finished, run
  `uv run task all` and fix failures.
- **Line width:** Aim for a line length of **79** characters in Python (ruff
  `line-length`). Soft-wrap Markdown near 80 where practical.
- **Incremental changes:** Make small, incremental changes that are easy to
  review.
- **Communicate clearly:** When you propose changes, explain what you did and
  why.
- **Create and follow a TODO list:** Create a TODO list and follow it. Do not
  stop until the tools make it clear that every task is complete.

## Notification Instructions

The user has given permission to use a desktop notification when a task is
complete, feedback is needed, or a question is required. Use the command for
the current operating system:

```bash
# Linux
notify-send "Question from Coding Agent" \
  "Please clarify how to complete the testing task."

# macOS
osascript -e 'display notification \
  "Please clarify how to complete the testing task." with title \
  "Question from Coding Agent"'
```

On Windows, install and use the PowerShell notification support from
BurntToast. Run the setup once in a regular PowerShell terminal:

```powershell
Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 `
  -Scope CurrentUser -Force -Confirm:$false
Set-PSRepository -Name PSGallery -InstallationPolicy Trusted
Install-Module -Name BurntToast -Scope CurrentUser -Force `
  -AllowClobber -Confirm:$false
```

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -Command `
  "Import-Module BurntToast; `
  New-BurntToastNotification -Text 'Question from Coding Agent', `
  'Please clarify how to complete the testing task.'"
```

- Always send a desktop notification whenever you ask a question and when a
  task is complete or feedback is needed. Adapt the title and body to the
  situation. Permission is already granted. Report unavailable commands.

- If the current session is running inside Zellij, also use this command when
  finished with a task or when further help is needed:

  ```bash
  timeout 2 zellij pipe -- \
    "zjstatus::notify::Agent finished. This is really fun. "
  ```

- The Zellij command only displays in the current Zellij session. It needs a
  two-second timeout and a trailing message space. It is optional outside
  Zellij and must not replace the desktop notification.

## Code Requirements

All Python changes should follow the conventions already used in this
codebase and enforced by ruff:

- **Formatting and lint:** ruff line length 79; selected rules include E, D,
  I, F, PL, Q, RUF, W, and T201; D203, D213, and E501 are ignored. Test files
  may use magic numbers (PLR2004).
- **Docstrings:** Module and function docstrings are expected (ruff `D`).
  Preserve the established docstring style in a file when editing it.
- **Constants:** Prefer namedtuples from `gator/constants.py`
  (`create_constants`). Do not invent parallel magic strings when a constant
  already exists.
- **Imports:** Group as standard library, third-party, then
  `from gator import ...`. Keep imports at the top of the file. Avoid
  wildcard imports.
- **Naming:** snake_case for functions and variables, PascalCase for classes,
  UPPER_SNAKE_CASE for module-level constants where that pattern is used.
- **Checks API:** Each check in `gator/checks/check_*.py` exposes
  `get_parser`, `parse`, and `act`.
- **Quality habits:** Use meaningful names; no bare `except:`; catch specific
  exceptions; avoid mutable default arguments; prefer `with` for resource
  cleanup; do not put secrets in source.
- **Do not impose:** full type hints on every function, a mypy-clean gate,
  Args/Returns sections on every docstring, no blank lines inside function
  bodies, lowercase-only comments, extracting every literal into a new
  constant, or Pydantic models.

### Check plugin shape

New or edited checks should expose this shape (mirror
`gator/checks/check_ConfirmFileExists.py` for argparse details):

```python
def get_parser():
    """Get a parser for the arguments provided on the command-line."""
    ...


def parse(args):
    """Use the parser on the provided arguments."""
    return checkers.parse(get_parser, args)


def act(main_parsed_arguments, check_remaining_arguments):
    """Perform the action for this check."""
    ...
```

## Project Structure Requirements

- Source code lives in `gator/` with the CLI in `gatorgrader.py`.
- Tests live in `tests/` with a matching structure, including
  `tests/checks/` for check plugins.
- Use `uv` for dependency management, virtual environments, and task running.
- Support Python 3.10 through 3.14 on macOS, Linux, and Windows (as CI does).
- Statement and branch coverage on `gator` must stay at or above **99%**
  (`coveragefailunder` in `pyproject.toml`).

## Testing Requirements

- Test functions follow the same Python conventions as production code where
  applicable.
- Names start with `test_` and should be descriptive.
- Group tests by the unit under test; order them for readability.
- Tests must be independent so they can run in any order.
- Tests must work on a local machine and in CI.
- Prefer `@pytest.mark.parametrize` patterns already used in the suite.
- Place check tests under `tests/checks/`.
- Aim for coverage consistent with the 99% fail-under gate.
- Do not delete failing tests to make the suite pass.
- New behavior should include new or updated tests.

## Git Workflow

- The software developer commits changes unless they explicitly ask the agent
  to create a commit.
- When asked to commit, write a clear, descriptive message; do not commit
  secrets, `.env` files, credentials, commented-out code, or debug leftovers.
- **Never run `git stash`:** It risks losing uncommitted work. Prefer a
  commit or `git worktree` if work must be set aside; the developer handles
  rebasing and history cleanup.

## Boundaries

- **Always:** Use `uv` and taskipy; run `uv run task all` before claiming
  done; put checks in `gator/checks/` with the `check_` prefix; put tests
  beside existing patterns; follow ruff; notify on questions and completion.
- **Ask first:** Adding runtime dependencies; changing CI or workflows;
  renaming public APIs; removing checks or tests.
- **Never:** Commit secrets or credentials; use `pip` or `venv` instead of
  `uv`; lower the 99% coverage gate to pass; invent mypy or other missing
  tools as required gates; rewrite the codebase to gatorgrade-only style
  rules; game tests or coverage to hide real failures.

## Making Changes

- **Understand:** Thoroughly understand the request and the relevant code.
   Explore with the available tools.
- **Plan:** Formulate a clear plan before making changes.
- **Implement:** Make small, incremental changes.
- **Verify:** Run `uv run task all` and fix any failures.
- **Commit:** The software developer commits unless they ask you to.
- **Rules:** Follow this file and the project's established conventions.
- **Completion:** When finished, summarize what you completed, how you
   completed it, challenges you faced, how you overcame them, and which rules
   you followed. Send notifications as described above.

## Additional Notes for the Coding Agent

This section is for coding agents. After finishing a task, append helpful
project notes here as a Markdown list when appropriate.

- Prefer [`README.md`](README.md) and this file for setup and task
  commands; do not rely on other contributor docs.
- The verification gate is `uv run task all` (lint + test + cover). Coverage
  fail-under is **99%** with branch coverage on `gator`.
- Check plugins live under `gator/checks/` and use the `check_` file prefix.
  Each exposes `get_parser`, `parse`, and `act`.
- [`scripts/tasks.py`](scripts/tasks.py) is a legacy Invoke/pyenv multi-version
  helper around the same taskipy commands. Prefer `uv python pin` for
  switching Python versions.
- No mypy or other static typecheck task is configured; do not treat typecheck
  as a required gate.
- Day-to-day lint and format are ruff via taskipy; bandit, radon, and
  xenon remain part of `uv run task lint`.
