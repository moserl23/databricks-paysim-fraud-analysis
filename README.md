# PaySim Fraud Analysis with Databricks

A small end-to-end Databricks project using the **PaySim synthetic
financial transaction dataset**.

The project demonstrates a **Medallion Architecture**, Delta Lake, Unity
Catalog, PySpark, Spark SQL, an AI/BI dashboard, and Lakeflow Jobs.

## Architecture

``` text
PaySim CSV
    ↓
Unity Catalog Volume
    ↓
Bronze
transactions_raw
    ↓
Silver
transactions
    ↓
Gold
analytics tables
    ↓
AI/BI Dashboard
```

The Bronze → Silver → Gold workflow is orchestrated with a **Databricks
Lakeflow Job**.

## Dataset

PaySim is a **synthetic mobile-money transaction dataset** generated
from a simulation based on patterns from real transaction data.

This dataset contains:

-   6,362,620 transactions
-   11 source columns
-   8,213 fraudulent transactions
-   5 transaction types: `CASH_OUT`, `PAYMENT`, `CASH_IN`, `TRANSFER`,
    `DEBIT`

The data is synthetic and does not contain real bank transactions.

## Medallion Pipeline

### Bronze --- `01_bronze_ingestion`

Reads `paysim.csv` from a Unity Catalog Volume using an explicit Spark
schema, adds ingestion metadata, and writes:

``` text
paysim_fraud.bronze.transactions_raw
```

### Silver --- `02_silver_transformations`

Cleans and enriches the Bronze data using PySpark.

Examples:

-   clearer column names
-   simulation day/hour
-   destination type
-   balance changes
-   fraud flags as booleans

Output:

``` text
paysim_fraud.silver.transactions
```

### Gold --- `03_gold_analytics`

Creates dashboard-ready aggregations using Spark SQL:

``` text
fraud_by_transaction_type
fraud_metrics_hourly
fraud_by_amount_band
rule_performance
overall_metrics
```

## Dashboard

The **PaySim Fraud Monitoring Dashboard** shows:

-   total transactions
-   fraud transactions
-   transaction count by type
-   fraud rate by transaction type
-   transaction volume over time
-   fraud transactions over time

## Lakeflow Job

The pipeline is orchestrated as:

``` text
bronze_ingestion
       ↓
silver_transformations
       ↓
gold_analytics
```

Each task reads and writes persisted Delta tables, so the notebooks do
not depend on shared notebook state.

## Technologies

-   Databricks
-   Apache Spark / PySpark
-   Spark SQL
-   Delta Lake
-   Unity Catalog & Volumes
-   Medallion Architecture
-   AI/BI Dashboards
-   Lakeflow Jobs
-   Git / GitHub

## Repository

``` text
databricks-paysim-fraud-analysis/
├── 01_bronze_ingestion
├── 02_silver_transformations
├── 03_gold_analytics
├── README.md
├── LICENSE
└── .gitignore
```

The raw CSV and generated Delta tables are not stored in Git.
