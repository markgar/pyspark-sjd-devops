# Plan: Package Scaffolding Refactor

**Goal:** Eliminate the manual "rename `src/spark_project/`" step. The sjd-builder
agent reads the package name from CONSTITUTION.md and scaffolds automatically.

## Current State (commit fc40908)

```
src/spark_project/__init__.py    ← placeholder name, must be renamed manually
pyproject.toml                   ← known-first-party = ["spark_project"]
.github/copilot-instructions.md  ← references src/spark_project/
tests/conftest.py                ← no package name references (clean)
```

## Target State

```
src/.gitkeep                     ← empty, agent creates package dir from CONSTITUTION.md
pyproject.toml                   ← [project] name = "_PACKAGE_NAME_", known-first-party = ["_PACKAGE_NAME_"]
.github/copilot-instructions.md  ← generic description, no placeholder (always truthful)
tests/conftest.py                ← unchanged
```

## Changes

### 1. Remove `src/spark_project/`

- Delete `src/spark_project/__init__.py`
- Delete `src/spark_project/` directory
- Add `src/.gitkeep` so git tracks the empty `src/` directory

### 2. Add `[project]` table and replace package name with `_PACKAGE_NAME_` placeholder

**pyproject.toml** — add `[project]` table and update isort:
```toml
# add (does not exist yet — required for pip install -e .)
[project]
name = "_PACKAGE_NAME_"
version = "0.1.0"
requires-python = ">=3.11"

# update existing
known-first-party = ["_PACKAGE_NAME_"]
```

**.github/copilot-instructions.md** — use generic description (no placeholder):
```markdown
# before
- Source code lives in `src/spark_project/`
# after
- Source code lives in `src/<package>/` (created by sjd-builder from CONSTITUTION.md)
```

This avoids the chicken-and-egg problem: copilot-instructions.md is loaded into
every agent's context. A literal `_PACKAGE_NAME_` placeholder would confuse
non-builder agents before scaffolding runs. The generic description is always
truthful.

### 3. Update sjd-builder agent — add scaffolding step 0

Add a new step before "Read the spec" in `.github/agents/sjd-builder.agent.md`:

**Step 0: Scaffold the package (if needed)**
1. Read CONSTITUTION.md → extract `Package name`
2. Check if `src/<package_name>/` exists
3. If not:
   - `mkdir -p src/<package_name>`
   - Create `src/<package_name>/__init__.py` with version and docstring
   - Replace `_PACKAGE_NAME_` → `<package_name>` in `pyproject.toml` (`[project] name` and `known-first-party`)
   - Replace generic source path → `src/<package_name>/` in `.github/copilot-instructions.md`
4. Run `pip install -e .` to install the package in editable mode

### 4. Update sample_spec_set CONSTITUTION.md

Remove the "IMPORTANT: rename" callout — the scaffolding step handles it now.
The CONSTITUTION just needs to declare the package name; the agent does the rest.

## What Doesn't Change

- `pyproject.toml` tool config (ruff, pytest, pyright, coverage) — all name-agnostic
- `tests/conftest.py` — no package references
- `.pre-commit-config.yaml` — name-agnostic
- `.devcontainer/` — name-agnostic
- `.github/skills/` — name-agnostic
- `devops_helpers/fabric_ops.py` — name-agnostic
- `.gitignore`, `.gitattributes` — name-agnostic

## User Experience After

```
1. Clone the template
2. Write specs (CONSTITUTION.md declares package_name)
3. @sjd-builder implement the spec in my_specs/
   → agent reads package_name from CONSTITUTION.md
   → agent scaffolds src/<package_name>/ automatically
   → agent implements the spec
4. Done. No renaming. No manual steps.
```

## Execution Order

1. ~~Commit and push current state~~ ✅
2. Remove `src/spark_project/`, add `src/.gitkeep`
3. Add `[project]` table + update `pyproject.toml` placeholder
4. Update `.github/copilot-instructions.md` (generic description, no placeholder)
5. Update `sample_spec_set/CONSTITUTION.md` (remove rename callout)
6. Update `.github/agents/sjd-builder.agent.md` (add step 0)
7. Test: verify `ruff check` and `pytest` still pass with placeholder
8. Commit and push
