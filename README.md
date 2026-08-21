# PaySim Fraud Monitoring on Databricks

An end-to-end Databricks demonstration using the **PaySim synthetic
financial transaction dataset**.\
The project implements a simple **Medallion Architecture (Bronze →
Silver → Gold)** with Unity Catalog, Delta tables, PySpark, Spark SQL, a
Databricks AI/BI dashboard, and Lakeflow Jobs orchestration.

## Project Overview

The goal is to demonstrate a realistic Databricks workflow for
ingesting, transforming, analyzing, and monitoring financial transaction
data.

``` text
PaySim CSV
    |
    v
Unity Catalog Volume
    |
    v
Bronze Delta Table
    |
    | PySpark
    v
Silver Delta Table
    |
    | Spark SQL
    v
Gold Analytics Tables
    |
    +--------------------+
    |                    |
    v                    v
AI/BI Dashboard      Lakeflow Job
                     Bronze -> Silver -> Gold
```

## Dataset

The project uses the **PaySim** dataset, a synthetic mobile-money
transaction dataset generated from a simulation based on patterns
observed in real transaction data.

The dataset used here contains:

-   **6,362,620 transactions**
-   **11 original columns**
-   **8,213 fraudulent transactions**
-   Five transaction types: `CASH_OUT`, `PAYMENT`, `CASH_IN`,
    `TRANSFER`, and `DEBIT`

The source CSV is approximately 494 MB.

### Original columns

  -----------------------------------------------------------------------
  Column                              Description
  ----------------------------------- -----------------------------------
  `step`                              Simulation time step; one step
                                      corresponds to one hour

  `type`                              Transaction type

  `amount`                            Transaction amount

  `nameOrig`                          Origin account/customer identifier

  `oldbalanceOrg`                     Origin balance before the
                                      transaction

  `newbalanceOrig`                    Origin balance after the
                                      transaction

  `nameDest`                          Destination
                                      account/customer/merchant
                                      identifier

  `oldbalanceDest`                    Destination balance before the
                                      transaction

  `newbalanceDest`                    Destination balance after the
                                      transaction

  `isFraud`                           Whether the simulated transaction
                                      is fraudulent

  `isFlaggedFraud`                    Whether the transaction was flagged
                                      by the built-in fraud rule
  -----------------------------------------------------------------------

> The data is synthetic and is used here for educational and
> demonstration purposes. It is not transaction data from RBI or another
> real bank.

## Databricks Architecture

The project uses a Unity Catalog catalog named:

``` text
paysim_fraud
```

The main objects are organized as follows:

``` text
paysim_fraud
|
+-- bronze
|   +-- Volume: raw_files
|   |   +-- paysim.csv
|   |
|   +-- Table: transactions_raw
|
+-- silver
|   +-- Table: transactions
|
+-- gold
    +-- fraud_by_transaction_type
    +-- fraud_metrics_hourly
    +-- fraud_by_amount_band
    +-- rule_performance
    +-- overall_metrics
```

## Bronze Layer

Notebook:

``` text
01_bronze_ingestion
```

The Bronze layer ingests the original CSV from a Unity Catalog Volume.

Source:

``` text
/Volumes/paysim_fraud/bronze/raw_files/paysim.csv
```

Main steps:

1.  Define an explicit Spark schema.
2.  Read the CSV with PySpark.
3.  Preserve the original source columns.
4.  Add ingestion metadata:
    -   `ingestion_timestamp`
    -   `source_file`
5.  Write the result as a managed Delta table.

Output:

``` text
paysim_fraud.bronze.transactions_raw
```

The Bronze layer intentionally remains close to the raw source data.

## Silver Layer

Notebook:

``` text
02_silver_transformations
```

The Silver layer reads the Bronze Delta table and creates a cleaner,
analytics-ready transaction model.

Output:

``` text
paysim_fraud.silver.transactions
```

### Transformations

The Silver pipeline includes:

-   clearer column names
-   Boolean fraud flags
-   simulated day and hour derivation
-   destination classification
-   balance-change features
-   fraud-relevant derived attributes

Examples of renamed columns:

``` text
nameOrig        -> origin_account
nameDest        -> destination_account
oldbalanceOrg   -> origin_balance_before
newbalanceOrig  -> origin_balance_after
isFraud         -> is_fraud
isFlaggedFraud  -> is_flagged_fraud
type            -> transaction_type
```

Additional derived fields include:

``` text
simulation_day
hour_of_day
destination_type
origin_balance_change
destination_balance_change
origin_account_emptied
amount_to_origin_balance_ratio
```

The Silver table retains all **6,362,620 transactions**.

## Gold Layer

Notebook:

``` text
03_gold_analytics
```

The Gold layer uses Spark SQL to create small, business-oriented
analytical tables for dashboard consumption.

### `fraud_by_transaction_type`

Aggregates transaction volume, transaction amount, fraud count, fraud
amount, flagged transactions, and fraud rate by transaction type.

This makes it easy to compare transaction frequency with fraud risk.

### `fraud_metrics_hourly`

Aggregates activity by simulation time step.

Includes:

-   transaction count
-   total amount
-   average transaction amount
-   fraud count
-   fraud amount
-   fraud rate
-   flagged transaction count

This table powers the time-series dashboard visualizations.

### `fraud_by_amount_band`

Groups transactions into amount ranges and calculates fraud statistics
for each range.

### `rule_performance`

Summarizes the performance of the dataset's existing fraud flag using:

-   true positives
-   false positives
-   false negatives
-   true negatives

### `overall_metrics`

Contains high-level dashboard metrics such as:

-   total transactions
-   fraudulent transactions
-   fraud rate
-   total transaction amount
-   fraudulent transaction amount

## AI/BI Dashboard

Dashboard:

``` text
PaySim Fraud Monitoring Dashboard
```

The dashboard provides a compact overview of transaction activity and
fraud patterns.

Current visualizations include:

-   **Total Transactions** --- approximately 6.36 million
-   **Fraud Transactions** --- 8,213
-   **Transaction Count by Transaction Type**
-   **Fraud Rate by Transaction Type**
-   **Transaction Volume Over Time**
-   **Fraud Transactions Over Time**

One immediately visible pattern is that `CASH_OUT` and `PAYMENT` account
for large transaction volumes, while fraud in PaySim is concentrated in
`TRANSFER` and `CASH_OUT` transactions.

## Lakeflow Job

The notebooks are orchestrated as a Databricks Lakeflow Job.

Job:

``` text
PaySim Medallion Pipeline
```

Task dependency graph:

``` text
bronze_ingestion
        |
        v
silver_transformations
        |
        v
gold_analytics
```

Each task communicates through persisted Unity Catalog Delta tables
rather than notebook-local variables.

This makes the workflow reproducible and demonstrates a basic
production-style data pipeline.

For this demonstration, Bronze and Silver use overwrite semantics so
that the complete pipeline can be rerun deterministically.

## Repository Structure

``` text
databricks-paysim-fraud-analysis/
|
+-- 01_bronze_ingestion
+-- 02_silver_transformations
+-- 03_gold_analytics
+-- README.md
+-- LICENSE
+-- .gitignore
```

The repository contains the transformation logic and project
documentation.

The raw PaySim CSV and generated Delta tables are **not committed to
Git**. Data is stored and governed within Databricks.

## Technologies Demonstrated

-   Databricks
-   Apache Spark
-   PySpark
-   Spark SQL
-   Delta Lake
-   Unity Catalog
-   Unity Catalog Volumes
-   Medallion Architecture
-   Databricks AI/BI Dashboards
-   Databricks Lakeflow Jobs
-   Git / GitHub

## Design Decisions

### Why Bronze, Silver, and Gold?

The Medallion Architecture separates responsibilities:

**Bronze** preserves source data and ingestion metadata.

**Silver** provides cleaned, standardized, reusable transaction data.

**Gold** contains purpose-specific aggregates optimized for analytics
and dashboard consumption.

### Why PySpark for Bronze and Silver?

PySpark provides a clear programmatic workflow for schema definition,
ingestion, transformations, and feature derivation over millions of
rows.

### Why SQL for Gold?

Gold tables represent business-facing aggregations. SQL makes these
transformations concise, readable, and easy for analysts and data
engineers to understand.

### Why Unity Catalog?

Unity Catalog provides a structured namespace and governance layer for
volumes, schemas, and Delta tables.

### Why keep the CSV out of Git?

Git is used for **code and configuration**, not large datasets.

The raw data lives in a Databricks Volume, while transformed datasets
live as managed Delta tables.

## Running the Project

A typical execution flow is:

1.  Upload `paysim.csv` to:

``` text
/Volumes/paysim_fraud/bronze/raw_files/paysim.csv
```

2.  Run `01_bronze_ingestion`.
3.  Run `02_silver_transformations`.
4.  Run `03_gold_analytics`.
5.  Refresh the AI/BI dashboard.

Alternatively, execute the complete Bronze → Silver → Gold workflow
through the **PaySim Medallion Pipeline** Lakeflow Job.

## Purpose

This repository is an educational Databricks project designed to
demonstrate an end-to-end data engineering workflow on a realistic
financial fraud scenario.

It focuses on platform and data-engineering concepts rather than
building a machine-learning model:

``` text
ingestion -> governance -> transformation -> analytics -> orchestration
```

The result is a small but complete Databricks project that can be used
to demonstrate practical familiarity with the core components of a
modern lakehouse workflow.
