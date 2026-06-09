# Free Snowflake Table Scan Efficiency Report

**Find which Snowflake tables are wasting compute on avoidable partition scans — and what it's worth.**

A free, fully source-available Snowflake notebook you run **in your own account**. It reads
Snowflake's own telemetry and shows you how much of your scanning is avoidable, a conservative
dollar estimate of the opportunity, the tables where it concentrates, and the columns your queries
filter on.

> **Read-only — runs entirely in your account.** Every query is a `SELECT`; it **never modifies,
> writes, or deletes any data or objects.** No app install. **No external network calls.** No data
> leaves your account. No persistent objects created. It only reads Snowflake's Account Usage
> telemetry and renders the results inline. (See [SECURITY.md](SECURITY.md).)

It sizes the opportunity; it does **not** tell you which clustering key to build — that's the part
[Goldilox Insights](https://goldilox.com) handles. Diagnosis is free and open; the prescription is
the product.

---

## Run it in 5 minutes

1. **Download** [`notebooks/Table_Scan_Efficiency_Report.ipynb`](notebooks/Table_Scan_Efficiency_Report.ipynb)
   from this repo (open the file → **Download raw file**).
2. In **Snowsight**, go to **Projects → Notebooks**, click the **▾** next to **+ Notebook**, and
   choose **Import .ipynb file**. Pick the file.
3. Choose any database/schema to hold it and attach a warehouse (an **X-Small** is plenty).
4. Set the notebook's role to a read-only role (see below) or `ACCOUNTADMIN`, then click **Run all**.

That's it — the results render inline. Nothing is installed or sent anywhere.

---

## Recommended role (least privilege)

You *can* run it as `ACCOUNTADMIN`, but it only needs **read** access to Account Usage. Have an
admin create a minimal read-only role once:

```sql
-- Run once as ACCOUNTADMIN (or a role with MANAGE GRANTS):
CREATE ROLE IF NOT EXISTS SCAN_REPORT_READER;

-- Read-only access to Snowflake's Account Usage telemetry (metadata only — not your table data):
GRANT IMPORTED PRIVILEGES ON DATABASE SNOWFLAKE TO ROLE SCAN_REPORT_READER;

-- A small warehouse to run the notebook's light queries:
GRANT USAGE ON WAREHOUSE <YOUR_WAREHOUSE> TO ROLE SCAN_REPORT_READER;

-- Let whoever runs the report use the role:
GRANT ROLE SCAN_REPORT_READER TO USER <YOUR_USER>;
```

Then select `SCAN_REPORT_READER` as the notebook's role. (`ACCOUNTADMIN` works too if you'd rather
skip this step.)

---

## What it reads — and what it doesn't

**Reads** (Snowflake-native Account Usage views — query/partition *metadata*, never table contents):

- `SNOWFLAKE.ACCOUNT_USAGE.TABLE_PRUNING_HISTORY`
- `SNOWFLAKE.ACCOUNT_USAGE.COLUMN_QUERY_PRUNING_HISTORY`
- `SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY` + `QUERY_ATTRIBUTION_HISTORY`

**Does not**: make external calls, send data anywhere, create objects, or recommend clustering keys.

**Requirements**: Enterprise Edition or above (the pruning views), and a few hours of Account Usage
latency before recent activity appears.

---

## How to read the results

1. **Scan efficiency** — what share of micro-partitions your large tables read vs. skip, plus a daily trend.
2. **Opportunity ($)** — a directional, upper-bound estimate of the compute tied to scanning.
3. **Top tables** — where avoidable scanning concentrates, ranked, with apportioned dollars.
4. **Filter/join columns** — which columns your queries actually filter on (context, not a recommendation).

---

## What Goldilox does next

This report tells you *whether* there's a problem and *how big*. The next questions — *which*
clustering key to apply, in what column order, and the projected post-change savings — require
modeling selectivity, cardinality, and predicate interaction. That's
[**Goldilox Insights**](https://goldilox.com): a Snowflake Native App that makes the recommendation
and projects the ROI, running entirely in your account.

> 📈 In our TPC-DS benchmark, Goldilox's clustering recommendations cut the workload's compute **~50%**.
> Snowflake bills by compute-time, and scanning **~76% fewer partitions** (**~81% fewer bytes**) roughly
> halved total query time — the avoidable scanning this report measures, turned into savings.

If your estimated opportunity looks meaningful, start at **[goldilox.com](https://goldilox.com)** or
email **[contact@goldilox.com](mailto:contact@goldilox.com)**.

---

## Advanced: install from Git (auto-updating)

Instead of importing the file, you can register this repo as a Snowflake Git repository so the
notebook stays in sync:

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

-- Create the notebook from the repo
CREATE SCHEMA IF NOT EXISTS GOLDILOX_INSIGHTS_CLIENT_WORKSPACE.NOTEBOOKS;
CREATE OR REPLACE NOTEBOOK GOLDILOX_INSIGHTS_CLIENT_WORKSPACE.NOTEBOOKS.TABLE_SCAN_EFFICIENCY_REPORT
  FROM '@GOLDILOX_INSIGHTS_CLIENT_WORKSPACE.REPO.NOTEBOOKS_REPO/branches/main'
  MAIN_FILE = 'notebooks/Table_Scan_Efficiency_Report.ipynb'
  IDLE_AUTO_SHUTDOWN_TIME_SECONDS = 60
  COMMENT = 'Goldilox - Table scan efficiency report';

-- To update later: ALTER GIT REPOSITORY ... FETCH; then CREATE OR REPLACE NOTEBOOK ... again.
```

---

## Already a GOLDILOX Insights customer?

Companion notebooks for the installed app (permissions, shared views, migration) live in
[`notebooks/existing-goldilox-customers/`](notebooks/existing-goldilox-customers/) — see that
folder's README for setup instructions.

---

*License: [Apache-2.0](LICENSE). Questions & inquiries: [contact@goldilox.com](mailto:contact@goldilox.com).*
