# Project Setup Specification

This document describes the complete project structure and configuration for a local
PySpark development environment matching Microsoft Fabric Runtime 1.3 (Spark 3.5,
Java 11, Python 3.11, Delta Lake 3.2). Ruff is the sole linter and formatter —
it replaces black, flake8, isort, pylint, pyupgrade, and bandit.

The `_template-ref/` directory contains the original microsoft/python-package-template
for reference.

---

## Step 1 — `.devcontainer/devcontainer.json`

The devcontainer should have these extensions and settings:

```jsonc
{
  "name": "Fabric Runtime 1.3 (Local PySpark)",
  "build": {
    "dockerfile": "Dockerfile"
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "ms-python.vscode-pylance",
        "ms-toolsai.jupyter",
        "charliermarsh.ruff"
      ],
      "settings": {
        "python.defaultInterpreterPath": "/usr/bin/python3.11",
        "python.analysis.typeCheckingMode": "basic",
        "editor.formatOnSave": true,
        "editor.defaultFormatter": "charliermarsh.ruff",
        "[python]": {
          "editor.defaultFormatter": "charliermarsh.ruff",
          "editor.codeActionsOnSave": {
            "source.fixAll.ruff": "explicit",
            "source.organizeImports.ruff": "explicit"
          }
        },
        "files.trimTrailingWhitespace": true,
        "python.testing.unittestEnabled": false,
        "python.testing.pytestEnabled": true
      }
    }
  },
  "remoteUser": "vscode"
}
```

## Step 2 — `.devcontainer/Dockerfile`

The Dockerfile pip install block should include these packages:

```
pyspark==3.5.0
delta-spark==3.2.0
jupyter
pandas==2.1.4
numpy==1.26.4
pyarrow==14.0.2
ms-fabric-cli
azure-identity
azure-storage-file-datalake
ruff
pytest
pre-commit
pytest-cov
```

## Step 3 — `.gitignore`

The project root should have a `.gitignore` with standard Python patterns plus
Spark and VS Code entries:

```
# Byte-compiled / optimized / DLL files
__pycache__/
*.py[cod]
*$py.class

# C extensions
*.so

# Distribution / packaging
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg

# PyInstaller
*.manifest
*.spec

# Installer logs
pip-log.txt
pip-delete-this-directory.txt

# Unit test / coverage reports
htmlcov/
.tox/
.nox/
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
*.py,cover
.hypothesis/
.pytest_cache/

# Environments
.env
.venv
env/
venv/
ENV/

# mypy
.mypy_cache/

# pyright
.pyright/

# Jupyter
.ipynb_checkpoints

# Spark
derby.log
metastore_db/
spark-warehouse/

# VS Code
.vscode/
```

## Step 4 — `pyproject.toml`

The project root should have a `pyproject.toml` with ruff, pyright, pytest, and
coverage configuration. No `[build-system]`, `[project]`, `[tool.black]`,
`[tool.flake8]`, `[tool.bandit]`, `[tool.pylint.*]`, or `[tool.tox]` sections.

```toml
[tool.ruff]
target-version = "py311"
line-length = 120
src = ["src"]

[tool.ruff.lint]
select = [
    "F",      # Pyflakes
    "E", "W", # pycodestyle
    "B",      # flake8-bugbear
    "I",      # isort
    "UP",     # pyupgrade
    "S",      # flake8-bandit
    "PL",     # Pylint rules
]
ignore = [
    "E722",   # bare except
    "E203",   # whitespace before ':'
]

[tool.ruff.lint.isort]
known-first-party = ["spark_project"]

[tool.pyright]
include = ["src"]
exclude = ["**/node_modules", "**/__pycache__"]
reportMissingImports = true
reportMissingTypeStubs = false
pythonVersion = "3.11"
pythonPlatform = "Linux"
executionEnvironments = [{ root = "src" }]

[tool.pytest.ini_options]
addopts = "--cov-report xml:coverage.xml --cov src --cov-fail-under 0 --cov-append -m 'not integration'"
pythonpath = ["src"]
testpaths = "tests"
junit_family = "xunit2"
markers = [
    "integration: marks as integration test",
    "spark: marks tests which need Spark",
    "slow: marks tests as slow",
    "unit: fast offline tests",
]

[tool.coverage.run]
branch = true

[tool.coverage.report]
fail_under = 0
```

## Step 5 — `.pre-commit-config.yaml`

The project root should have a `.pre-commit-config.yaml` with two repos:
pre-commit-hooks for general checks and ruff-pre-commit for linting/formatting.

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v5.0.0
    hooks:
      - id: check-added-large-files
      - id: check-case-conflict
      - id: check-merge-conflict
      - id: check-yaml
      - id: debug-statements
      - id: end-of-file-fixer
      - id: trailing-whitespace

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.15.5
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format
```

## Step 6 — `src/spark_project/__init__.py`

The source package is `spark_project` under `src/`:

```python
"""Spark Project."""

from __future__ import annotations

__version__ = "0.0.1"
```

## Step 7 — `tests/conftest.py`

The test directory should have a `conftest.py` with auto-marker logic that tags
tests containing "spark" in the name with `pytest.mark.spark` and tests containing
"_int_" with `pytest.mark.integration`:

```python
"""Pytest configuration and shared fixtures."""

from __future__ import annotations

import pytest
from _pytest.nodes import Item


def pytest_collection_modifyitems(items: list[Item]):
    for item in items:
        if "spark" in item.nodeid:
            item.add_marker(pytest.mark.spark)
        elif "_int_" in item.nodeid:
            item.add_marker(pytest.mark.integration)


@pytest.fixture
def unit_test_mocks(monkeypatch):
    """Include mocks here to execute all commands offline and fast."""
    pass
```

## Step 8 — Initialize git and install pre-commit hooks

The project must be a git repository. Install pre-commit hooks:

```bash
git init
pre-commit install
```

## Step 9 — `.github/copilot-instructions.md`

The project should have a `.github/copilot-instructions.md` with project context
for GitHub Copilot:

```markdown
# Copilot Instructions

## Project

Local PySpark development environment matching Microsoft Fabric Runtime 1.3
(Spark 3.5, Java 11, Python 3.11, Delta Lake 3.2).

## Stack

- **Python 3.11** — use modern syntax (type hints, `match`, `|` unions)
- **PySpark 3.5 + Delta Lake 3.2** — primary data processing framework
- **ruff** — sole linter and formatter (no black, flake8, pylint, isort)
- **pytest + pytest-cov** — testing framework
- **pre-commit** — git hook runner

## Code conventions

- Line length: 120 characters
- Source code lives in `src/spark_project/`
- Tests live in `tests/`
- Use `from __future__ import annotations` in all modules
- Imports sorted by ruff (isort-compatible)
- Follow existing `pyproject.toml` ruff rules: F, E, W, B, I, UP, S, PL

## Testing

- Mark Spark-dependent tests so they can be filtered: `pytest -m spark`
- Tests with `_int_` in the name are auto-marked as integration tests
- Target `pytest -m "not integration"` for fast local runs
- Place fixtures in `tests/conftest.py`

## PySpark patterns

- Prefer DataFrame API over SQL strings
- Use `spark.createDataFrame()` with explicit schemas in tests
- Stop SparkSessions in test fixtures (session-scoped)
```

---

## What is NOT included (and why)

| Template item | Why skipped |
|---|---|
| `black`, `flake8`, `isort`, `pylint`, `bandit`, `pyupgrade` | Ruff replaces all of these |
| `ms-python.pylint`, `ms-python.isort`, `ms-python.flake8`, `ms-python.black-formatter` extensions | Ruff extension replaces all of these |
| `flit` / `[build-system]` / `[tool.flit.module]` | Not publishing to PyPI |
| `tox` | Running pytest directly |
| `check-manifest` | Not publishing to PyPI |
| `shellcheck-py` | No shell scripts |
| `codespell` pre-commit hook | Nice-to-have, add later if wanted |
| `python.formatting.provider: black` VS Code setting | Deprecated, does nothing |
| Tool `.path` settings (black, pylint, flake8, isort) | Those paths are for a different container image |
| `docs/` (Sphinx setup) | Add when needed |
| `CODE_OF_CONDUCT.md`, `SECURITY.md`, `SUPPORT.md`, `LICENSE` | Boilerplate for public open-source repos |
| `.github/workflows/` | Add CI workflows when needed |
| `coverage fail_under = 100` | Starts at 0, increase as test coverage grows |
| 150+ lines of `[tool.pylint.*]` config | Ruff PL rules cover the important checks without configuration |
| RST pre-commit hooks | Not writing RST docs |
| `pytest-github-actions-annotate-failures`, `pylint_junit`, `flake8-formatter_junit_xml` | CI-specific, add when using GitHub Actions |

## Notes

- `dbutils` — Pyright will flag as undefined. Suppress with
  `# type: ignore[reportUndefinedVariable]` or add to pyright `extraPaths`.
- PySpark type stubs — Pylance handles these. No config needed with
  `ms-python.vscode-pylance`.
