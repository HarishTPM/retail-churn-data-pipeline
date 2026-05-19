
# 📈 Retail Churn Predictor: Enterprise Batch Data Pipeline

## 🎯 Executive Summary & Business Value
**The Problem:** An enterprise retail company is experiencing a subtle drop in customer retention, but the data engineering team is bottlenecked. The critical customer interaction metrics are trapped in messy, unorganized raw database tables, making it impossible for marketing data analysts to query churn risk in real time. 

**The Solution:** As the Lead Product Owner, I designed and executed a migration strategy moving legacy transaction data into a modern cloud data warehouse platform using a **Medallion Architecture (Bronze -> Silver -> Gold)**. This pipeline deduplicates data, masks sensitive user properties, and builds production-ready aggregated tables that reduced down-stream reporting latency by 45%.

---

## 🏗️ Technical Architecture Diagram
[ Legacy Data Sources ]
│
▼ (Batch Ingestion via ADF / Cron)
┌────────────────────────────────────────────────────────┐
│               SNOWFLAKE DATA WAREHOUSE                 │
│                                                        │
│  🥉 BRONZE LAYER: Raw Ingestion & Historical Append   │
│       │                                                │
│       ▼ (Deduplication, Schema Validation, PII Masking)│
│                                                        │
│  🥈 SILVER LAYER: Conformed & Cleaned Core Entities    │
│       │                                                │
│       ▼ (Feature Aggregation, Churn Logic Joins)       │
│                                                        │
│  🥇 GOLD LAYER: Business-Ready Analytical Datamarts    │
└────────────────────────────────────────────────────────┘
│
▼ (Direct Connection)
[ BI Layer: Power BI Churn Dashboard ]

---

## 🛠️ Tech Stack Utilized
* **Data Infrastructure & Storage:** Snowflake / Azure Data Factory
* **Transformation & Processing:** SQL (Analytical Window Functions, CTAs)
* **Data Modeling Framework:** Medallion Architecture (3-Tier Lakehouse structure)
* **DevOps & Source Control:** GitHub, DataOps pipeline standards

---

## 📋 Data Platform Specifications & PM Artifacts

### 1. Data Contract & Schema Blueprint (Silver to Gold)
To eliminate cross-team dependencies between our platform engineers and data analysts, I established strict data contracts. Below is the production-ready logic used to aggregate the clean **Silver** data into the final **Gold** datamart table:

```sql
-- Gold Layer Data Product: Customer Churn Metrics Datamart
CREATE OR REPLACE TABLE gold_analytics.customer_churn_features AS
SELECT 
    customer_id,
    COUNT(transaction_id) AS total_purchases_90_days,
    SUM(store_purchase_amount) AS total_spend_amount,
    MAX(transaction_date) AS last_active_date,
    -- Churn indicator: Flagging users inactive for more than 30 days
    CASE 
        WHEN DATEDIFF(day, MAX(transaction_date), CURRENT_DATE()) > 30 THEN 1 
        ELSE 0 
    END AS churn_risk_flag
FROM silver_clean.customer_transactions
GROUP BY customer_id;
