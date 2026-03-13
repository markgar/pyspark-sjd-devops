# pyspark-sjd-devops

Local-first PySpark development environment for Microsoft Fabric Spark Job Definitions (SJDs).

Same code runs locally and on Fabric — only 3–4 narrow branch points separate the two environments.

## What's in the box

| Component | Purpose |
|---|---|
| **Dev container** | Fabric Runtime 1.3 parity: Spark 3.5, Java 11, Python 3.11, Delta Lake 3.2 |
| **`src/`** | Your PySpark package (source layout, `pip install -e .`) |
| **`tests/`** | pytest + pytest-cov, Spark-aware markers (`spark`, `integration`) |
| **`main.py`** | SJD entry point — created by sjd-builder when you implement a spec |
| **`devops_helpers/`** | CLI tools for Fabric deployment, monitoring, and log retrieval |
| **`sample_spec_set/`** | Example spec set demonstrating the sjd-builder workflow |
| **`.github/`** | Copilot agent, skills, and workspace instructions |

## Quick start

1. **Open in the dev container** — VS Code will build the container with Spark, Delta, JDBC drivers, Azure CLI, and all Python dependencies pre-installed.

2. **Write a spec** — describe your ETL pipeline in markdown (see `sample_spec_set/` for the format).

3. **Build with sjd-builder** — point the agent at your spec:
   ```
   @sjd-builder implement the spec in sample_spec_set/
   ```
   It reads the spec, writes the code and tests, runs locally, and deploys to Fabric.

4. **Or build manually:**
   ```bash
   # Install your package
   pip install -e .

   # Run tests
   pytest -m "not integration"

   # Run locally
   python main.py

   # Deploy and run on Fabric
   pip wheel . -w dist/ --no-deps
   # upload .whl to Fabric Environment, publish, deploy SJD...
   export WS_ID=<workspace-id> SJD_ID=<sjd-id>
   python devops_helpers/fabric_ops.py run
   python devops_helpers/fabric_ops.py logs
   ```

## How local/Fabric branching works

| Concern | Local | Fabric | Bridge |
|---|---|---|---|
| Detection | `LOCAL_DEV=1` | not set | `is_local_dev()` |
| File paths | `lakehouse/Files/…` | `Files/…` | `LAKEHOUSE_ROOT` env var |
| Spark + Delta | `local[*]` + `configure_spark_with_delta_pip()` | Managed cluster | Conditional setup in `main.py` |
| Auth | `az login` → `DefaultAzureCredential` | Managed Identity | Same code path |
| SQL | JDBC + access token | `.mssql()` + workspace identity | `is_local_dev()` branch |
| Libraries | `pip install -e .` | `.whl` via Fabric Environments | `environmentArtifactId` |

## `devops_helpers/fabric_ops.py`

CLI for Fabric Spark job operations. Requires `WS_ID` and `SJD_ID` environment variables.

```
fabric_ops.py run        # Submit job, wait for completion, show failure details
fabric_ops.py status     # Latest run status
fabric_ops.py runs       # List recent runs
fabric_ops.py livy       # Livy session details
fabric_ops.py logs       # Driver stdout/stderr
```

## Copilot customization

| File | What it does |
|---|---|
| `.github/copilot-instructions.md` | Global rules: code style, testing, Fabric constraints, devops_helpers check-first rule |
| `.github/agents/sjd-builder.agent.md` | SJD builder agent — full lifecycle from spec to deployed Fabric job |
| `.github/skills/fabric-ops/SKILL.md` | Deploy, run, and debug knowledge (API patterns, gotchas, LRO polling) |
| `.github/skills/local-spark/SKILL.md` | Dual-environment patterns (sessions, paths, auth, SQL connectivity) |

## Stack

- Python 3.11
- PySpark 3.5 + Delta Lake 3.2
- ruff (lint + format)
- pytest + pytest-cov
- pre-commit
- Azure CLI + GitHub CLI
- Microsoft Fabric Runtime 1.3 compatible
