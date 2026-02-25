# GOLDILOX-keyblade-runbook

This is a companian runbook for GOLDILOX Inisghts clients

### How to setup notebook in Snowsight

#### 1. Setup INTEGRATION

```
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

#### 2. Create NOTEBOOK
```
CREATE SCHEMA IF NOT EXISTS GOLDILOX_INSIGHTS_CLIENT_WORKSPACE.NOTEBOOKS;
USE SCHEMA GOLDILOX_INSIGHTS_CLIENT_WORKSPACE.NOTEBOOKS;

CREATE OR REPLACE NOTEBOOK GOLDILOX_INSIGHTS_RUNBOOK
  FROM '@REPO.NOTEBOOKS_REPO/branches/main'
  MAIN_FILE = 'notebooks/Setup_Shared_Views.ipynb'  
  -- QUERY_WAREHOUSE = CLIENT_WH                   -- used by SQL cells
  -- WAREHOUSE = CLIENT_WH                         -- used by Python runtime
  IDLE_AUTO_SHUTDOWN_TIME_SECONDS = 60         -- optional (15 mins)
  COMMENT = 'Client copy of Goldilox Insights runbook';

```

#### 3. Update NOTEBOOK with latest code
```
-- 1) Fetch latest from the Git repository
ALTER GIT REPOSITORY NOTEBOOKS_REPO FETCH;

-- 2) Refresh the notebook to the latest branch contents
CREATE OR REPLACE NOTEBOOK GOLDILOX_INSIGHTS_RUNBOOK
  FROM '@REPO.NOTEBOOKS_REPO/branches/main'
  MAIN_FILE = 'notebooks/Setup_Shared_Views.ipynb';
```