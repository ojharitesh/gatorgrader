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