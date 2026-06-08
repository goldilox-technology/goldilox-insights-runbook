# Security

The **Table Scan Efficiency Report** notebook is designed to run entirely inside your own Snowflake
account. It is a diagnostic, not an agent.

## What it does

- **Read-only.** Every query is a `SELECT` against Snowflake's own telemetry views — it never
  modifies, writes, or deletes any data or objects.
- **No external network calls.** The notebook contacts nothing outside Snowflake — no data is sent
  to Goldilox or any third party.
- **No persistent objects.** It creates no tables, stages, tasks, or other objects; results are
  rendered inline only.

## What it reads

Only Snowflake-native Account Usage views — query and partition **metadata** (counts, timings,
credits), never the contents of your tables:

- `SNOWFLAKE.ACCOUNT_USAGE.TABLE_PRUNING_HISTORY`
- `SNOWFLAKE.ACCOUNT_USAGE.COLUMN_QUERY_PRUNING_HISTORY`
- `SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY`
- `SNOWFLAKE.ACCOUNT_USAGE.QUERY_ATTRIBUTION_HISTORY`

## Verify it yourself

The notebook is source-available — every cell is visible (expand any results-only cell to read the
code). Review exactly which queries will run before you execute it.

## Least-privilege role

You can run it as `ACCOUNTADMIN`, but it only needs **read** access to Account Usage. See the
README's "Recommended role" section for a minimal read-only role plus the exact grant SQL.

## Reporting a concern

Email **support@goldilox.com**.
