# GOLDILOX Insights — Companion Notebooks

Operational notebooks for **existing GOLDILOX Insights customers**, run after the Native App is
installed. Run as `ACCOUNTADMIN`.

> Just evaluating Goldilox? You probably want the free
> [Table Scan Efficiency Report](../Table_Scan_Efficiency_Report.ipynb) in the repo root instead —
> no install required.

| Notebook | Purpose |
|---|---|
| `Setup_App_Permissions.ipynb` | **Onboarding & ongoing operations.** Grants warehouse MONITOR and database access required to keep the app running. Re-run when adding warehouses or databases. |
| `Setup_Shared_Views.ipynb` | **Troubleshooting & data sharing.** Creates shared views to share data back to the Goldilox provider for support. |
| `Migration.ipynb` | **Migration.** Prepares your account for app relisting — backup schema + app grants. |

## Install from Git

```sql
-- One-time: workspace + API integration + Git repository object (no credentials needed)
CREATE DATABASE IF NOT EXISTS GOLDILOX_INSIGHTS_CLIENT_WORKSPACE;
CREATE SCHEMA   IF NOT EXISTS GOLDILOX_INSIGHTS_CLIENT_WORKSPACE.REPO;

CREATE OR REPLACE API INTEGRATION goldilox_public_git
  API_PROVIDER = git_https_api
  API_ALLOWED_PREFIXES = ('https://github.com/goldilox-technology')
  ENABLED = TRUE;

CREATE OR REPLACE GIT REPOSITORY GOLDILOX_INSIGHTS_CLIENT_WORKSPACE.REPO.NOTEBOOKS_REPO
  ORIGIN = 'https://github.com/goldilox-technology/goldilox-insights-runbook.git'
  API_INTEGRATION = goldilox_public_git;

-- Create the notebooks
CREATE SCHEMA IF NOT EXISTS GOLDILOX_INSIGHTS_CLIENT_WORKSPACE.NOTEBOOKS;
USE SCHEMA GOLDILOX_INSIGHTS_CLIENT_WORKSPACE.NOTEBOOKS;

CREATE OR REPLACE NOTEBOOK GOLDILOX_SETUP_APP_PERMISSIONS
  FROM '@GOLDILOX_INSIGHTS_CLIENT_WORKSPACE.REPO.NOTEBOOKS_REPO/branches/main'
  MAIN_FILE = 'notebooks/existing-goldilox-customers/Setup_App_Permissions.ipynb'
  IDLE_AUTO_SHUTDOWN_TIME_SECONDS = 60
  COMMENT = 'Goldilox Insights - Warehouse and database permission setup';

CREATE OR REPLACE NOTEBOOK GOLDILOX_SETUP_SHARED_VIEWS
  FROM '@GOLDILOX_INSIGHTS_CLIENT_WORKSPACE.REPO.NOTEBOOKS_REPO/branches/main'
  MAIN_FILE = 'notebooks/existing-goldilox-customers/Setup_Shared_Views.ipynb'
  IDLE_AUTO_SHUTDOWN_TIME_SECONDS = 60
  COMMENT = 'Goldilox Insights - Shared views and data sharing setup';

CREATE OR REPLACE NOTEBOOK GOLDILOX_MIGRATION
  FROM '@GOLDILOX_INSIGHTS_CLIENT_WORKSPACE.REPO.NOTEBOOKS_REPO/branches/main'
  MAIN_FILE = 'notebooks/existing-goldilox-customers/Migration.ipynb'
  IDLE_AUTO_SHUTDOWN_TIME_SECONDS = 60
  COMMENT = 'Goldilox Insights - Migration backup and restore';
```

## Update to the latest version

```sql
ALTER GIT REPOSITORY GOLDILOX_INSIGHTS_CLIENT_WORKSPACE.REPO.NOTEBOOKS_REPO FETCH;
-- then re-run the CREATE OR REPLACE NOTEBOOK statements above.
```

---

*License: [Apache-2.0](../../LICENSE). Questions: [contact@goldilox.com](mailto:contact@goldilox.com).*
