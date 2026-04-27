# Data Flow

This document explains the end-to-end data movement through the **End-to-End Supply Chain Data Platform on Snowflake**.

## Overview

The platform processes supply chain data from raw source files through ingestion, change capture, denormalized transformation, dimensional modeling, semantic analytics, and AI-powered retrieval. The entire flow is implemented using Snowflake-native capabilities and a three-layer medallion architecture.

## End-to-End Flow Summary

```text
Amazon S3 CSV files (13 entities)
  -> Snowpipe AUTO_INGEST via SQS
  -> DATA_RAW.SUPPLY_CHAIN (13 raw tables)
  -> 13 Streams (CDC)
  -> Task DAG + 5 stored procedures
  -> DATA_STAGE.SUPPLY_CHAIN (5 denormalized tables)
  -> Data quality checks + business rules
  -> DATA_CURATED.SUPPLY_CHAIN (4 dimensions + 3 facts)
  -> Semantic Views / Cortex Analyst
  -> AI Views / RAG Pipeline
```

## Step-by-Step Flow

### 1. Source Ingestion

The pipeline starts with 13 source entities delivered as CSV files into Amazon S3. Each source entity maps to a specific Snowpipe object and corresponding raw table.

Examples of source domains include:
- Orders
- Customer
- Distributor
- Product
- Shipment
- Manufacturing inventory
- Facility inventory
- Distributor inventory
- Raw material
- Component

## 2. Event-Driven Loading

Snowpipe continuously monitors S3 paths and automatically loads new files into the Raw layer.

Key ingestion settings include:
- `AUTO_INGEST = TRUE`
- `PARSE_HEADER = TRUE`
- `MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE`

This makes ingestion event-driven and reduces dependency on manual or brittle file-processing logic.

## 3. Raw Layer

**Target:** `DATA_RAW.SUPPLY_CHAIN`

The Raw layer stores an exact replica of source data for auditability and downstream CDC processing.

Key characteristics:
- 13 raw tables
- 13 Snowpipe objects
- 13 Streams
- Minimal business transformation
- Strong traceability to source

## 4. Change Data Capture

Streams are created on each Raw table to capture inserts, updates, and deletes. These Streams act as the trigger mechanism for the next step in the pipeline.

A Task DAG orchestrates downstream processing:
- `TASK_STG_ROOT_SCHEDULER` runs every 5 minutes
- Child tasks execute only when relevant Streams contain data
- `SYSTEM$STREAM_HAS_DATA` controls conditional execution

This design avoids unnecessary refresh work when no source changes are present.

## 5. Stage Refresh

The Stage layer is refreshed through stored procedures triggered by child tasks in the DAG.

Examples include:
- `TASK_REFRESH_STG_ORDERS`
- `TASK_REFRESH_STG_SHIPMENTS`
- `TASK_REFRESH_STG_MFG_INVENTORY`
- `TASK_REFRESH_STG_FAT_INVENTORY`
- `TASK_REFRESH_STG_DISTRIBUTOR_INVENTORY`

Each procedure performs a full refresh of its target staging table and consumes stream offsets after completion.

## 6. Stage Layer Transformations

**Target:** `DATA_STAGE.SUPPLY_CHAIN`

The Stage layer joins and enriches the 13 source tables into 5 denormalized tables optimized for analytics.

Core transformation patterns include:
- Channel resolution
- Origin/destination resolution
- Delivery status derivation
- Safety-stock evaluation
- Reorder logic
- Inventory valuation
- Product and pricing enrichment
- ZIP code formatting correction

This is also the main layer for applying business rules and preparing curated modeling inputs.

## 7. Data Quality Validation

Before data moves further downstream, the platform performs data quality checks across the staging tables.

Validation categories include:
- NULL pattern audits
- Duplicate checks
- Outlier detection
- Data type mismatch analysis
- Transformation formatting validation

These checks improve trust in the curated analytics layer and also surface structural source-system issues early.

## 8. Curated Modeling

**Target:** `DATA_CURATED.SUPPLY_CHAIN`

The Curated layer organizes data into a dimensional model designed for KPI reporting and governed analytics.

Published structures include:
- 4 dimension tables
- 3 fact tables

Dimensions:
- `DIM_DATE`
- `DIM_PRODUCT`
- `DIM_CUSTOMER`
- `DIM_FACILITY`

Facts:
- `FCT_ORDERS`
- `FCT_SHIPMENTS`
- `FCT_INVENTORY`

This layer becomes the single analytical foundation for dashboards, semantic views, and AI use cases.

## 9. Semantic Consumption

On top of the Curated layer, the platform publishes business-facing semantic views.

Semantic views include:
- `SUPPLY_CHAIN_ANALYTICS`
- `SALES_ANALYTICS`
- `LOGISTICS_ANALYTICS`
- `INVENTORY_HEALTH`

These views expose verified metrics, synonyms, and business-friendly structures for governed self-service analytics.

## 10. Natural Language Analytics

The semantic views enable **Cortex Analyst**, which translates business questions into SQL.

Example business questions include:
- What is the total revenue by business line?
- Show me the monthly revenue trend
- Who are the top 10 customers by revenue?
- What is the on-time delivery rate?
- Which inventory items are at critical stock levels?

## 11. AI View Consumption

The platform also publishes AI-enriched analytical views on top of the curated data model.

These views support:
- Executive KPI summaries
- Customer segmentation
- Order-risk classification
- Restock recommendations
- Delay analysis

This extends the pipeline beyond descriptive analytics into intelligent narrative and recommendation use cases.

## 12. RAG Pipeline

The final branch of the data flow is a Retrieval-Augmented Generation pipeline for conversational access.

Flow:
1. Build a 150-entry AI-enriched knowledge base
2. Index it with `SUPPLY_CHAIN_SEARCH`
3. Use `SP_RAG_QUERY` to retrieve relevant entries and generate a short answer

This allows users to ask natural-language supply chain questions and receive contextual responses backed by semantic retrieval.

## Design Characteristics

The data flow is designed around a few key principles:
- Event-driven ingestion
- CDC-aware downstream refresh
- Separation of concerns across medallion layers
- Governed analytics through semantic views
- AI enrichment without leaving Snowflake
- Strong traceability from source to insight

## Future Improvements

Potential next improvements include:
- Replace full-refresh stage processing with incremental `MERGE` logic
- Add automated curated-layer orchestration
- Expand continuous data quality monitoring with Snowflake DMFs
- Harden secrets and credential handling
