# Transformation Rules

This document summarizes the major transformation rules applied in the **End-to-End Supply Chain Data Platform on Snowflake**.

## Purpose

The transformation layer converts raw operational data into denormalized stage tables and curated dimensional models that support KPI reporting, semantic analytics, and AI-driven use cases.

## Transformation Layers

The project applies business logic across two main layers:

- **Stage layer** for denormalized joins, standardization, and intermediate business rules
- **Curated layer** for dimensional modeling, metric enrichment, and analytical classification

## Stage Layer Rules

### `STG_ORDERS_ENRICHED`

**Source joins**
- `ORDERS`
- `CUSTOMER`
- `DISTRIBUTOR`
- `PRODUCT`

**Key rules**
- Resolve selling channel as **DIRECT** or **DISTRIBUTOR**
- Derive `ORDER_CHANNEL` based on buyer relationship
- Calculate `PRICE_VARIANCE`
- Derive `PRODUCT_CATEGORY` using regex-based classification
- Apply `LPAD` correction to `ZIP_CODE` values to preserve leading zeros
- Maintain dual-customer/distributor interpretation by design

### `STG_SHIPMENT_DETAILS`

**Source joins**
- `SHIPMENT`
- `MFG_PLANT`
- `FAT_FACILITY` (origin/destination resolution)
- `DISTRIBUTOR`
- `CUSTOMER`

**Key rules**
- Resolve origin and destination through `COALESCE`-style matching logic
- Derive `DELIVERY_STATUS` as **EARLY**, **ON_TIME**, or **LATE**
- Calculate shipment transit-day metrics
- Flag unresolved origin mappings for downstream monitoring

### `STG_MFG_INVENTORY_ENRICHED`

**Source joins**
- `MFG_INVENTORY`
- `MFG_PLANT`
- `RAW_MATERIAL`
- `COMPONENT`

**Key rules**
- Derive `IS_BELOW_SAFETY_STOCK`
- Calculate `INVENTORY_VALUE`
- Preserve mutually exclusive `MATERIAL_ID` and `COMPONENT_ID` row patterns
- Support manufacturing inventory analysis with enriched material/component context

### `STG_FAT_INVENTORY_ENRICHED`

**Source joins**
- `FAT_INVENTORY`
- `FAT_FACILITY`
- `PRODUCT`

**Key rules**
- Enrich FAT inventory with facility and product context
- Preserve component/product interpretation where available
- Current limitation: `COMPONENT_ID` type mismatch prevents full component enrichment

### `STG_DISTRIBUTOR_INVENTORY_ENRICHED`

**Source joins**
- `DISTRIBUTOR_INVENTORY`
- `DISTRIBUTOR`
- `PRODUCT`

**Key rules**
- Derive `NEEDS_REORDER`
- Derive `IS_BELOW_SAFETY_STOCK`
- Calculate `INVENTORY_VALUE`
- Support replenishment and channel inventory visibility

## Cross-Cutting Stage Rules

### Standardization
- Normalize identifiers for downstream joins
- Correct formatting issues such as ZIP code padding
- Preserve source auditability while improving analytical usability

### Data Quality Controls
- Validate NULL patterns
- Check duplicate natural keys
- Detect statistical outliers using three-sigma logic
- Identify data type mismatch risks that impact joins

## Curated Layer Rules

### Dimension Rules

#### `DIM_DATE`
- Generated date spine
- Derive `IS_WEEKEND`
- Derive `YEAR_QUARTER`
- Derive `YEAR_MONTH`

#### `DIM_PRODUCT`
- Derive `PRODUCT_FAMILY` using regex
- Derive `PRICE_TIER` as **Low**, **Med**, **High**, or **Premium**
- Standardize `BUSINESS_LINE_NAME`

#### `DIM_CUSTOMER`
- Unify `CUSTOMER` and `DISTRIBUTOR` into a single buyer model
- Derive `BUYER_TYPE` as **DIRECT** or **DISTRIBUTOR**

#### `DIM_FACILITY`
- Unify `MFG_PLANT` and `FAT_FACILITY`
- Derive `FACILITY_TYPE` as **MFG_PLANT** or **FAT_FACILITY**

### Fact Rules

#### `FCT_ORDERS`
- Derive `ORDER_SIZE_TIER` as **SMALL**, **MEDIUM**, **LARGE**, or **ENTERPRISE**
- Derive `PRICING_TYPE` as **DISCOUNTED**, **LIST_PRICE**, or **PREMIUM**
- Set `NET_REVENUE` to zero for cancelled orders; otherwise use order amount
- Derive boolean flags such as `IS_CANCELLED` and `IS_FULFILLED`

#### `FCT_SHIPMENTS`
- Derive `DELAY_DAYS` as actual minus expected transit time
- Derive `SHIPPING_SPEED_TIER` as **EXPRESS**, **STANDARD**, **ECONOMY**, or **SLOW**
- Calculate `COST_PER_TRANSIT_DAY`
- Flag `HAS_UNKNOWN_ORIGIN` for unresolved facility mappings

#### `FCT_INVENTORY`
- Unify manufacturing, FAT, and distributor inventory into a single inventory fact
- Derive `STOCK_STATUS` as **OUT_OF_STOCK**, **CRITICAL**, **LOW**, or **HEALTHY**
- Derive `ITEM_TYPE` as **RAW_MATERIAL**, **COMPONENT**, or **FINISHED_GOOD**
- Include planning fields such as `LEAD_TIME_DAYS` and `DAYS_FORWARD_COVERAGE`

## Design Notes

- Stage transformations are optimized for readability, enrichment, and data quality validation.
- Curated transformations are optimized for dimensional consumption, KPI reporting, and semantic exposure.
- Full-refresh stage procedures simplify consistency today, while incremental processing is a future improvement path.
