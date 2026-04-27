End-to-End Supply Chain Data Platform on Snowflake

A portfolio project and white-paper-driven case study that showcases how to build an automated, AI-enriched analytics pipeline on Snowflake using native platform capabilities.

Overview

This project demonstrates an end-to-end supply chain analytics platform built around a medallion architecture with three layers:

- **Raw:** exact source replica for auditability
- **Stage:** denormalized transformations and business rules
- **Curated:** star schema, semantic views, AI views, and RAG-powered analytics

The solution is designed to support governed analytics, self-service KPI reporting, and AI-assisted business insights.

Architecture Flow

```text
Amazon S3 CSV files
   -> Snowpipe AUTOINGEST via SQS
   -> DATARAW.SUPPLYCHAIN
   -> Streams + Task DAG
   -> DATASTAGE.SUPPLYCHAIN
   -> Data Quality Framework
   -> DATACURATED.SUPPLYCHAIN
   -> Semantic Views / Cortex Analyst
   -> Cortex AI Views / RAG Pipeline
```

Named Objects

Databases and Schemas
- `DATARAW.SUPPLYCHAIN`
- `DATASTAGE.SUPPLYCHAIN`
- `DATACURATED.SUPPLYCHAIN`

Pipeline Components
- 13 Snowpipe objects for ingestion
- 13 Streams for CDC
- `TASKSTGROOTSCHEDULER`
- 5 child refresh tasks
- 5 stored procedures for stage refresh

AI and Search Components
- `SUPPLYCHAINSEARCH`
- `SPRAGQUERY`
- AI views for executive briefing, customer segmentation, order risk, restock recommendations, and delay analysis

End-to-End Flow

1. Ingestion Layer
Source data arrives as Amazon S3 CSV files across 13 entities. Snowpipe monitors the landing paths and continuously ingests data into the Raw layer.

2. Raw Layer
`DATARAW.SUPPLYCHAIN` stores the exact source replica for auditability and downstream CDC processing.

3. CDC Layer
Streams track inserts, updates, and deletes. A Task DAG, led by `TASKSTGROOTSCHEDULER`, orchestrates downstream refresh logic.

4. Staging Layer
`DATASTAGE.SUPPLYCHAIN` contains 5 denormalized staging tables. This layer applies joins, business rules, and data quality checks.

5. Data Quality Framework
The platform validates null patterns, duplicate records, outliers, data type issues, and formatting consistency before publishing downstream analytical models.

6. Curated Layer
`DATACURATED.SUPPLYCHAIN` contains a star schema with dimension and fact tables used for KPI reporting and analytics consumption.

7. Semantic Analytics
Snowflake Semantic Views provide a governed business layer for self-service analytics and Cortex Analyst natural-language querying.

8. AI Analytics and RAG
Cortex AI views generate business summaries, risk classifications, and recommendations. A RAG pipeline uses `SUPPLYCHAINSEARCH` and `SPRAGQUERY` for conversational retrieval over an AI-enriched knowledge base.

Transformations and KPI Examples

Example Transformations
- Channel resolution and customer-distributor unification
- Shipment delivery status classification
- Inventory value calculations
- Safety stock and reorder flags
- Pricing and order size enrichment
- Product family and business line classification

Example KPIs
- Revenue by business line
- Monthly revenue trend
- On-time delivery rate
- Delay days by shipment
- Inventory health status
- Reorder requirement
- Cost per transit day
- Critical stock counts

Semantic and AI Layer

Semantic Views
- `SUPPLYCHAINANALYTICS`
- `SALESANALYTICS`
- `LOGISTICSANALYTICS`
- `INVENTORYHEALTH`

AI Views
- `VAIEXECUTIVEBRIEFING`
- `VAICUSTOMERSEGMENTS`
- `VAIORDERRISK`
- `VAIRESTOCKRECOMMENDATIONS`
- `VAIDELAYANALYSIS`

Tech Stack

- **Storage / Compute:** Snowflake
- **Landing / Ingestion:** Amazon S3, Snowpipe, SQS
- **CDC / Orchestration:** Streams, Tasks
- **Transformations:** SQL stored procedures
- **Modeling:** Star schema, semantic views
- **AI:** Snowflake Cortex functions (`AICOMPLETE`, `AICLASSIFY`)
- **RAG:** Cortex Search, AI-enriched knowledge base, `SPRAGQUERY`

Why This Project Matters

This project demonstrates the ability to design a production-style analytics platform that goes beyond ingestion and reporting. It combines cloud data engineering, data modeling, governance, KPI enablement, semantic analytics, and AI capabilities within a single Snowflake-native architecture.

Suggested Repo Additions

To strengthen this repository further, consider adding:

- Mermaid architecture diagrams
- Sanitized SQL examples for stage and curated layers
- Sample semantic view YAML or pseudocode
- Screenshots of KPI dashboards
- A short architecture decision record (ADR)
- Known issues and scaling recommendations

Summary

This project can be presented as a complete Snowflake-native supply chain platform that ingests source data from S3, processes changes with Streams and Tasks, applies transformation and data quality logic in Stage, models business-ready facts and dimensions in Curated, and exposes both KPI analytics and AI-powered insights using semantic views, Cortex AI, and RAG.
