# Semantic Layer

This document explains how the semantic layer exposes governed business analytics in the **End-to-End Supply Chain Data Platform on Snowflake**.

## Purpose

The semantic layer sits on top of the curated dimensional model and translates technical warehouse structures into business-friendly metrics, synonyms, and example questions. It is the main bridge between curated facts and dimensions and the people who consume analytics.

## Semantic Layer Role

The semantic layer serves three goals:

- Standardize metric definitions
- Provide governed self-service access
- Enable natural-language analytics through Cortex Analyst

## Semantic Views

The platform defines four semantic views built with `SYSTEM$CREATE_SEMANTIC_VIEW_FROM_YAML`.

### `SUPPLY_CHAIN_ANALYTICS`
- **Audience:** Executive and cross-functional users
- **Tables:** All 7 curated tables
- **Coverage:** Full supply chain business view
- **Value:** High-level operational and financial visibility

### `SALES_ANALYTICS`
- **Audience:** Finance and Sales
- **Tables:** `FCT_ORDERS`, `DIM_CUSTOMER`, `DIM_PRODUCT`, `DIM_DATE`
- **Metrics:** Revenue and order-related metrics
- **Value:** Sales performance and customer analysis

### `LOGISTICS_ANALYTICS`
- **Audience:** Operations and Logistics
- **Tables:** `FCT_SHIPMENTS`, `DIM_FACILITY`, `DIM_DATE`
- **Metrics:** Shipment performance and delivery metrics
- **Value:** Transportation and service-level analysis

### `INVENTORY_HEALTH`
- **Audience:** Supply Planning and Warehouse
- **Tables:** `FCT_INVENTORY`, `DIM_PRODUCT`, `DIM_FACILITY`
- **Metrics:** Inventory health and stock-status metrics
- **Value:** Inventory risk and replenishment visibility

## What the Semantic Layer Provides

The semantic views include:

- Business-friendly names
- Metric definitions
- Synonyms for common business terms
- Verified example queries
- Governed access patterns

## Cortex Analyst

The semantic layer is also the foundation for Snowflake Cortex Analyst, which translates natural-language business questions into SQL.

Example questions supported by the model:

- What is the total revenue by business line?
- Show me the monthly revenue trend.
- Who are the top 10 customers by revenue?
- What is the on-time delivery rate?
- Which inventory items are at critical stock levels?

## Metric Consumption Pattern

A typical consumption flow looks like this:

1. Raw data lands in Snowflake.
2. Stage transforms and validates it.
3. Curated builds facts and dimensions.
4. Semantic views expose business metrics.
5. Cortex Analyst converts questions into SQL.
6. Business users get governed answers without needing to understand the warehouse design.

## Why This Layer Matters

The semantic layer reduces ambiguity and duplicate metric logic. It ensures that revenue, delivery, and inventory KPIs are defined once and reused consistently across teams. It also makes the platform friendlier for non-technical users while preserving control for data governance.
