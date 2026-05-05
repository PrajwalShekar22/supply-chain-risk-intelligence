# Supply Chain Late Delivery Risk Intelligence Platform

![Python](https://img.shields.io/badge/Python-3.12-blue)
![dbt](https://img.shields.io/badge/dbt-Cloud-orange)
![Snowflake](https://img.shields.io/badge/Snowflake-Data%20Warehouse-29B5E8)
![Airflow](https://img.shields.io/badge/Apache%20Airflow-2.8.4-017CEE)
![Tableau](https://img.shields.io/badge/Tableau-Dashboard-E97627)

## Project Overview

An end-to-end supply chain analytics platform that identifies orders at risk of late delivery before they are delivered. The platform ingests raw order data, transforms it through a multi-layer dbt pipeline, validates data quality with 86 automated tests, and serves insights through interactive Tableau dashboards.

**Business Question:** How can we identify which orders are at risk of late delivery before they are delivered?

**Key Finding:** First Class shipping has a 95.6% late delivery rate — the highest risk shipping mode — compared to Standard Class at 38.09%.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Data Ingestion | Python (pandas, snowflake-connector-python) |
| Data Warehouse | Snowflake (X-Small warehouse, $400 trial credits) |
| Data Transformation | dbt Cloud |
| Orchestration | Apache Airflow 2.8.4 |
| CI/CD | GitHub Actions |
| Visualization | Tableau Desktop |
| Version Control | Git + GitHub |

---

## Dataset

**Source:** DataCo Smart Supply Chain Dataset (Kaggle)

| File | Rows | Columns | Description |
|---|---|---|---|
| DataCoSupplyChainDataset.csv | 180,519 | 47 | Order-level supply chain data |
| tokenized_access_logs.csv | 469,977 | 8 | Web traffic clickstream data |

**Date Range:** January 2015 — January 2018 (3 years)

**Target Variable:** `LATE_DELIVERY_RISK` — binary (1 = late, 0 = not late)
- Late deliveries: 98,977 (54.8%)
- On time: 81,542 (45.2%)

**Key Dropped Columns (PII):** Customer Email, Customer Password, Customer Street, Product Description, Product Image, Order Zipcode

---

## Architecture

CSV Files (Mac)
↓
Python Ingestion Script
↓
Snowflake RAW Schema
(RAW_SUPPLY_CHAIN: 180,519 rows | RAW_WEB_TRAFFIC: 469,977 rows)
↓
dbt Cloud Transformation
↓
Snowflake STAGING Schema (Views)
stg_orders | stg_customers | stg_products | stg_web_traffic
↓
Snowflake MARTS Schema (Tables)
fct_orders | dim_customers | dim_products | dim_geography
↓
Snowflake REPORTING Schema (Tables)
rpt_delivery_kpis | rpt_revenue_analysis | rpt_risk_prediction | rpt_web_traffic
↓
Tableau Dashboard

---

## Project Structure

supply-chain-risk-intelligence/
├── .github/
│   └── workflows/
│       └── dbt_ci.yml              # GitHub Actions CI/CD
├── airflow/
│   └── dags/
│       └── supply_chain_pipeline.py # Airflow DAG
├── ingestion/
│   └── load_to_snowflake.py        # Python ingestion script
├── models/
│   ├── staging/
│   │   ├── sources.yml             # RAW source definitions
│   │   ├── staging_tests.yml       # Staging data quality tests
│   │   ├── stg_orders.sql          # Cleaned order data
│   │   ├── stg_customers.sql       # Deduplicated customer data
│   │   ├── stg_products.sql        # Deduplicated product data
│   │   └── stg_web_traffic.sql     # Cleaned web traffic data
│   ├── marts/
│   │   ├── marts_tests.yml         # Mart data quality tests
│   │   ├── fct_orders.sql          # Fact table - order items
│   │   ├── dim_customers.sql       # Customer dimension
│   │   ├── dim_products.sql        # Product dimension
│   │   └── dim_geography.sql       # Geography dimension
│   └── reporting/
│       ├── reporting_tests.yml     # Reporting data quality tests
│       ├── rpt_delivery_kpis.sql   # Delivery KPI aggregations
│       ├── rpt_revenue_analysis.sql # Revenue analysis
│       ├── rpt_risk_prediction.sql  # Risk prediction features
│       └── rpt_web_traffic.sql     # Web traffic analysis
├── tests/
│   ├── assert_shipping_days_range.sql      # Business logic test
│   └── assert_late_risk_matches_status.sql # Business logic test
└── dbt_project.yml                 # dbt project configuration

---

## dbt Models

### Staging Layer (Views)
| Model | Source | Description |
|---|---|---|
| stg_orders | RAW_SUPPLY_CHAIN | Cleaned order data, cast dates, derived metrics |
| stg_customers | RAW_SUPPLY_CHAIN | Deduplicated customer records |
| stg_products | RAW_SUPPLY_CHAIN | Deduplicated product catalog |
| stg_web_traffic | RAW_WEB_TRAFFIC | Cleaned web clickstream data |

### Marts Layer (Tables)
| Model | Description |
|---|---|
| fct_orders | Fact table - 180,519 rows - core analytics table |
| dim_customers | Customer dimension with order metrics and value tiers |
| dim_products | Product dimension with sales metrics and price tiers |
| dim_geography | Geography dimension with late delivery rates by region |

### Reporting Layer (Tables)
| Model | Description |
|---|---|
| rpt_delivery_kpis | Late delivery KPIs aggregated by time, shipping mode, market |
| rpt_revenue_analysis | Revenue and profit analysis by segment and department |
| rpt_risk_prediction | Risk prediction features for ML model inputs |
| rpt_web_traffic | Web traffic patterns by department, category, time of day |

---

## Data Quality

**86 automated dbt tests across all layers:**

| Layer | Tests | Types |
|---|---|---|
| Staging | 29 | unique, not_null, accepted_values |
| Marts | 34 | unique, not_null, accepted_values, relationships |
| Reporting | 21 | not_null, accepted_values |
| Custom SQL | 2 | Business logic tests |
| **Total** | **86** | **All passing** |

**Custom Business Logic Tests:**
- `assert_shipping_days_range` — validates shipping days between 0 and 6
- `assert_late_risk_matches_status` — validates late_delivery_risk=1 always maps to 'Late delivery' status

---

## Key Business Insights

| Insight | Finding |
|---|---|
| Highest risk shipping mode | First Class — 95.6% late delivery rate |
| Lowest risk shipping mode | Standard Class — 38.09% late delivery rate |
| Orders with negative profit | 33,784 orders (18.7%) |
| Date range | Jan 2015 — Jan 2018 (3 years) |
| Total unique customers | 20,652 |
| Total unique products | 118 |
| Total unique orders | 65,752 |

---

## Pipeline Orchestration (Airflow)

The pipeline runs daily at 6AM via Apache Airflow:
Task 1: load_raw_data_to_snowflake
↓
Task 2: dbt_build
↓
Task 3: dbt_test
↓
Task 4: verify_row_counts

**Airflow UI:** `http://localhost:8080`
**DAG:** `supply_chain_risk_pipeline`
**Schedule:** `0 6 * * *` (daily at 6AM)

---

## CI/CD (GitHub Actions)

Every push to `main` branch automatically triggers:
1. Install dbt-snowflake
2. Create profiles.yml from GitHub Secrets
3. `dbt deps`
4. `dbt build`
5. `dbt test`

---

## Snowflake Setup

```sql
-- Database
SUPPLY_CHAIN_DB

-- Schemas
RAW      -- Raw ingested data
STAGING  -- dbt transformed views and tables

-- Warehouse
SUPPLY_CHAIN_WH (X-Small, AUTO_SUSPEND=60)
```

---

## How to Run

### 1. Install dependencies
```bash
pip install snowflake-connector-python pandas
```

### 2. Run Python ingestion
```bash
cd ingestion
python3 load_to_snowflake.py
```

### 3. Run dbt models
```bash
dbt build
```

### 4. Run dbt tests
```bash
dbt test
```

### 5. Start Airflow
```bash
conda activate airflow_env
export AIRFLOW_HOME=~/supply-chain-risk-intelligence/airflow
airflow webserver --port 8080
airflow scheduler
```

---

## Author

**Prajwal Gorkhar Chandrashekar**
- MS Business Analytics, Supply Chain Track — ASU W.P. Carey School of Business (GPA 3.75)
- LinkedIn: [linkedin.com/in/prajwalshekar](https://linkedin.com/in/prajwalshekar)
- GitHub: [github.com/PrajwalShekar22](https://github.com/PrajwalShekar22)
- Portfolio: [datascienceportfol.io/pgorkhar](https://datascienceportfol.io/pgorkhar)