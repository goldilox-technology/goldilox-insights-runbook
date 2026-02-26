# GOLDILOX Insights Runbook

Companion runbook for GOLDILOX Insights clients.

### Available Notebooks

| Notebook | Purpose | Run As |
|---|---|---|
| `Setup_App_Permissions.ipynb` | **Onboarding & ongoing operations.** Grants warehouse MONITOR and database access permissions required to keep the app running uninterrupted. Re-run when adding new warehouses or databases. | ACCOUNTADMIN |
| `Setup_Shared_Views.ipynb` | **Troubleshooting & data sharing.** Creates shared views and tables to share data back to the Goldilox provider for support and analysis. | ACCOUNTADMIN |

### How to setup notebooks in Snowsight

#### 1. Setup Integration

```sql
-- 1) Create Database for runbook
CREATE DATABASE GOLDILOX_INSIGHTS_CLIENT_WORKSPACE;
USE GOLDILOX_INSIGHTS_CLIENT_WORKSPACE;
CREATE SCHEMA REPO;

-- 2) Create an API integration that allows GitHub access to public repo (no creds)
CREATE OR REPLACE API INTEGRATION goldilox_public_git
  API_PROVIDER = git_https_api
  API_ALLOWED_PREFIXES = ('https://github.com/goldilox-technology')
  ENABLED = TRUE;

-- 3) Register the public repo as a Git repository object
CREATE OR REPLACE GIT REPOSITORY NOTEBOOKS_REPO
  ORIGIN = 'https://github.com/goldilox-technology/goldilox-insights-runbook.git'
  API_INTEGRATION = goldilox_public_git;
```

#### 2. Create Notebooks

```sql
CREATE SCHEMA IF NOT EXISTS GOLDILOX_INSIGHTS_CLIENT_WORKSPACE.NOTEBOOKS;
USE SCHEMA GOLDILOX_INSIGHTS_CLIENT_WORKSPACE.NOTEBOOKS;

-- Notebook 1: Setup App Permissions (required for onboarding and ongoing operations)
CREATE OR REPLACE NOTEBOOK GOLDILOX_SETUP_APP_PERMISSIONS
  FROM '@REPO.NOTEBOOKS_REPO/branches/main'
  MAIN_FILE = 'notebooks/Setup_App_Permissions.ipynb'
  -- QUERY_WAREHOUSE = CLIENT_WH                   -- used by SQL cells
  -- WAREHOUSE = CLIENT_WH                         -- used by Python runtime
  IDLE_AUTO_SHUTDOWN_TIME_SECONDS = 60
  COMMENT = 'Goldilox Insights - Warehouse and database permission setup';

-- Notebook 2: Setup Shared Views (for troubleshooting and data sharing with provider)
CREATE OR REPLACE NOTEBOOK GOLDILOX_SETUP_SHARED_VIEWS
  FROM '@REPO.NOTEBOOKS_REPO/branches/main'
  MAIN_FILE = 'notebooks/Setup_Shared_Views.ipynb'
  -- QUERY_WAREHOUSE = CLIENT_WH                   -- used by SQL cells
  -- WAREHOUSE = CLIENT_WH                         -- used by Python runtime
  IDLE_AUTO_SHUTDOWN_TIME_SECONDS = 60
  COMMENT = 'Goldilox Insights - Shared views and data sharing setup';
```

#### 3. Update Notebooks with latest code

```sql
-- 1) Fetch latest from the Git repository
ALTER GIT REPOSITORY NOTEBOOKS_REPO FETCH;

-- 2) Refresh the notebooks to the latest branch contents
CREATE OR REPLACE NOTEBOOK GOLDILOX_SETUP_APP_PERMISSIONS
  FROM '@REPO.NOTEBOOKS_REPO/branches/main'
  MAIN_FILE = 'notebooks/Setup_App_Permissions.ipynb';

CREATE OR REPLACE NOTEBOOK GOLDILOX_SETUP_SHARED_VIEWS
  FROM '@REPO.NOTEBOOKS_REPO/branches/main'
  MAIN_FILE = 'notebooks/Setup_Shared_Views.ipynb';
```