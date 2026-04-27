# Known Issues and Recommendations

This document captures the major data quality and architecture issues identified in the project, along with recommended next steps.

## Data Quality Issues

### Component ID Type Mismatch
- **Issue:** `FAT_INVENTORY` stores `COMPONENT_ID` as TEXT/UUID while `COMPONENT` stores it as NUMBER
- **Impact:** Prevents successful joins and causes 100% NULL values on `COMPONENT_NAME` in FAT inventory enrichment
- **Recommendation:** Standardize source key types or introduce a mapping/crosswalk table

### Unresolved Origin Facilities
- **Issue:** 49% of shipment origin UUIDs do not map to `MFG_PLANT` or `FAT_FACILITY`
- **Impact:** Weakens shipment lineage and origin analysis
- **Recommendation:** Investigate whether an upstream supplier or warehouse entity is missing from the model

### Unresolved Destination IDs
- **Issue:** 25.5% of shipment destination values cannot be resolved
- **Impact:** Reduces completeness of destination analytics
- **Recommendation:** Audit destination identifiers across all possible source entity types

### ZIP Code Stored as Number
- **Issue:** Source stores ZIP codes as NUMBER
- **Impact:** Leading zeros are lost for some addresses
- **Recommendation:** Fix at source; keep LPAD logic in staging as interim protection

## Architecture Limitations

### Full-Refresh Stage Processing
- **Issue:** Current child task procedures use full refresh logic
- **Impact:** Simpler for consistency, but not ideal for higher data volume or scale
- **Recommendation:** Evolve toward incremental `MERGE`-based processing

### Curated Layer Automation Gap
- **Issue:** Current focus is stronger from Raw to Stage than from Stage to Curated orchestration
- **Impact:** Curated refresh automation may need to mature as the platform scales
- **Recommendation:** Add second-tier orchestration for curated refresh and downstream publication

### Data Quality Monitoring Maturity
- **Issue:** Data quality checks are defined, but continuous monitoring can be improved
- **Impact:** Issues may be identified later than desired in production
- **Recommendation:** Implement Snowflake Data Metric Functions and alerting

### Credential Handling Risk
- **Issue:** Plaintext credentials or exposed secrets represent operational risk
- **Impact:** Security and compliance exposure
- **Recommendation:** Use Snowflake secrets management and rotate any exposed credentials immediately
