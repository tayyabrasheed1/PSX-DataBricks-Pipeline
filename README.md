# PSX Stock Data Pipeline : Databricks + PySpark + Delta Lake

An end-to-end ETL pipeline that ingests daily price data for Pakistan Stock Exchange (PSX) tickers, cleans and transforms it using PySpark, and produces analysis-ready summary tables — built on the Databricks Lakehouse Platform following the Medallion (Bronze/Silver/Gold) architecture.

## Why I built this

I'm a product designer transitioning into data engineering, and I wanted a project that reflected something I actually use — I follow PSX as an investor, so instead of building against a generic sample dataset, I built the pipeline I'd want for tracking my own portfolio. It doubles as hands-on practice with PySpark, Delta Lake, and Databricks Workflows ahead of the Databricks Data Engineer Associate certification.

## Architecture

Raw PSX price data → Bronze (raw, untouched) → Silver (cleaned, enriched) → Gold (business-ready summary)

```
psxdata library
      │
      ▼
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│   BRONZE    │ ──▶ │    SILVER    │ ──▶ │    GOLD     │
│ raw OHLCV   │     │ deduplicated │     │ per-symbol  │
│ per ticker  │     │ + returns    │     │ summary:    │
│             │     │ + 20-day MA  │     │ avg return, │
│             │     │              │     │ volatility  │
└─────────────┘     └──────────────┘     └─────────────┘
```

- **Bronze**: Raw daily OHLCV (open/high/low/close/volume) data pulled for each ticker, written untouched to a Delta table.
- **Silver**: Deduplicated by symbol + date, with calculated fields added — daily return %, 20-day moving average — using PySpark window functions.
- **Gold**: Aggregated per-symbol summary — latest close, average daily return, and volatility (standard deviation of returns) — the table the chart is built from.

## Tech stack

- **Databricks Free Edition** (serverless workspace)
- **PySpark** — DataFrame transformations, window functions
- **Delta Lake** — table storage format for all three layers
- **[psxdata](https://github.com/mtauha/psxdata)** — open-source Python library for free PSX market data
- **Databricks Jobs / Lakeflow Jobs** — daily scheduled orchestration

## Pipeline steps

1. Install `psxdata` and pull historical OHLCV data for selected tickers
2. Write raw data to the `psx_bronze` Delta table
3. Clean, deduplicate, and calculate returns/moving averages → `psx_silver`
4. Aggregate into per-symbol summary stats → `psx_gold`
5. Visualize `psx_gold` (bar chart: volatility and return by symbol)
6. Schedule the notebook as a daily Databricks Job, running after PSX market close

## Sample output

<img width="1178" height="753" alt="image" src="https://github.com/user-attachments/assets/af5dab8a-8af7-4c71-b24f-56aa5c566922" />


*(Bar chart comparing average daily return and volatility across tracked tickers)*

## How to run this yourself

1. Sign up for [Databricks Free Edition](https://www.databricks.com/learn/free-edition) (no cloud account or credit card needed)
2. Import this notebook into your workspace
3. Run the cells top to bottom — the first cell installs all dependencies
4. Optionally, set up a daily schedule via the Schedule button or Jobs & Pipelines

## What I'd improve next

- **More tickers, more sectors** — currently tracking a handful of stocks; expanding to 15–20 across banking, cement, energy, and tech would make the Gold-layer comparisons more meaningful
- **Data quality checks** — add validation for missing dates, duplicate rows, and OHLC constraint violations (e.g. low > high) before promoting Bronze → Silver
- **Error handling around scraping** — `psxdata` is an alpha-stage library that scrapes PSX's site; if PSX changes their page structure, pulls can silently fail. Wrapping ingestion in try/except with logging would make this production-safer
- **Incremental loads instead of full overwrite** — right now each run overwrites the tables; switching to `MERGE INTO` (upsert) would be more realistic for a daily production pipeline and match what the Databricks certification covers
- **Unity Catalog governance** — add proper catalog/schema structure and access controls instead of default tables, to reflect how this would look in an enterprise environment
- **Alerting** — add a notification (email/Slack) on job failure so a broken pipeline doesn't go unnoticed for days
