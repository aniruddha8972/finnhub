# FinPulse – Metadata Design

**Project Name:** FinPulse – Enterprise Financial Market Data Platform  
**Document:** Metadata Design  
**Version:** 1.0  
**Status:** Draft  
**Author:** Aniruddha Giri  
**Role:** Data Engineer  
**Platform:** Databricks Free Edition  

---

# 1. Document Purpose

This document defines the metadata-driven framework used by the FinPulse platform.

The metadata framework provides centralized configuration for:

- Source systems
- API endpoints
- Dataset definitions
- Target layers
- Target tables
- Load types
- Processing frequencies
- Watermark configuration
- Active/inactive datasets
- Pipeline parameters
- Data-quality configuration
- Processing dependencies

The objective is to minimize hardcoded pipeline logic and make FinPulse easier to maintain, extend, and operate.

---

# 2. Metadata-Driven Architecture

The FinPulse processing framework follows:

```text
                 Metadata Configuration
                         │
                         ▼
                 Metadata Controller
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
          Source       Target      Processing
        Parameters     Details       Rules
             │           │           │
             └───────────┼───────────┘
                         ▼
                  Dynamic Notebook
                         │
                         ▼
                    Data Pipeline
```

Instead of creating a separate hardcoded implementation for every dataset, the pipeline reads metadata and determines how the dataset should be processed.

---

# 3. Metadata Framework Objectives

The framework is designed to achieve:

### Reusability

A common processing framework can handle multiple datasets.

### Maintainability

Configuration changes can be made through metadata rather than modifying notebook logic.

### Scalability

New datasets can be added with minimal code changes.

### Standardization

All pipelines follow consistent processing and auditing patterns.

### Operational Control

Processing behavior can be controlled through configuration.

---

# 4. Metadata Components

The FinPulse metadata framework contains the following logical components:

```text
Metadata Configuration
        │
        ├── Dataset Metadata
        ├── Source Metadata
        ├── Target Metadata
        ├── Processing Metadata
        ├── Watermark Metadata
        ├── DQ Metadata
        └── Pipeline Metadata
```

---

# 5. Metadata Storage

The project uses configuration files and/or metadata tables to control processing.

The repository contains environment-specific configuration:

```text
control_framework/
│
├── config/
│   ├── dev.json
│   ├── sit.json
│   ├── uat.json
│   └── prod.json
```

The configuration allows environment-specific values to be separated from notebook logic.

---

# 6. Environment Configuration

The framework supports:

```text
DEV
SIT
UAT
PROD
```

Even though the current project runs on Databricks Free Edition, the structure is designed to simulate enterprise deployment practices.

---

# 7. Environment Configuration Responsibilities

Environment configuration may contain:

- Environment name
- Source configuration
- Target database/catalog
- Storage paths
- API configuration references
- Processing parameters
- Logging configuration
- Feature flags

Secrets such as API keys should not be stored as plaintext inside Git-controlled configuration files.

---

# 8. Dataset Metadata

The core metadata entity is the dataset.

Logical dataset:

```text
dataset_metadata
```

Recommended structure:

| Column | Type | Description |
|---|---|---|
| `metadata_id` | STRING | Unique metadata identifier |
| `dataset_name` | STRING | Logical dataset name |
| `source_system` | STRING | Source system |
| `source_endpoint` | STRING | API endpoint |
| `source_object` | STRING | Source object/resource |
| `target_layer` | STRING | Target processing layer |
| `target_table` | STRING | Target table |
| `load_type` | STRING | FULL/INCREMENTAL |
| `watermark_column` | STRING | Incremental column |
| `processing_frequency` | STRING | Processing schedule |
| `active_flag` | BOOLEAN | Dataset status |
| `created_timestamp` | TIMESTAMP | Creation time |
| `updated_timestamp` | TIMESTAMP | Last update |

---

# 9. Example Dataset Metadata

Example configuration:

```json
{
  "metadata_id": "M001",
  "dataset_name": "finnhub_stock_quote",
  "source_system": "Finnhub",
  "source_endpoint": "/quote",
  "target_layer": "SADP",
  "target_table": "stock_quote",
  "load_type": "INCREMENTAL",
  "watermark_column": "event_timestamp",
  "processing_frequency": "DAILY",
  "active_flag": true
}
```

The actual configuration should be adjusted to match the implementation.

---

# 10. Dataset Registration

A new dataset can be registered through metadata.

Example:

```text
New Dataset
     │
     ▼
Create Metadata Record
     │
     ├── Source
     ├── Target
     ├── Load Type
     ├── Watermark
     └── Frequency
     │
     ▼
Activate Dataset
     │
     ▼
Dynamic Pipeline
```

This avoids requiring an entirely new processing framework for every source.

---

# 11. Source Metadata

Source metadata defines where data originates.

Logical structure:

```text
source_metadata
```

Recommended fields:

| Field | Description |
|---|---|
| `source_system` | Source name |
| `source_type` | API / FILE / DATABASE |
| `base_url` | API base URL |
| `endpoint` | Endpoint |
| `authentication_type` | Authentication method |
| `request_method` | GET/POST |
| `active_flag` | Source status |

For FinPulse:

```text
source_system = Finnhub
source_type   = REST_API
```

---

# 12. Finnhub Source Metadata

Example logical configuration:

```json
{
  "source_system": "Finnhub",
  "source_type": "REST_API",
  "base_url": "<configured_base_url>",
  "authentication_type": "API_KEY",
  "request_method": "GET",
  "active_flag": true
}
```

The API key should be retrieved securely rather than stored directly in the repository.

---

# 13. Target Metadata

Target metadata defines where processed data should be written.

Logical structure:

```text
target_metadata
```

Recommended fields:

| Field | Description |
|---|---|
| `target_layer` | NI/SADP/AADP/CADP |
| `target_database` | Target database/schema |
| `target_table` | Target table |
| `storage_format` | Delta |
| `write_mode` | Append/Merge/Overwrite |
| `partition_column` | Partition field |
| `active_flag` | Target status |

---

# 14. FinPulse Target Layers

Metadata should identify the processing layer.

```text
NI
SADP
AADP
CADP
```

Example:

| Dataset | Layer | Purpose |
|---|---|---|
| Raw API data | NI | Native source preservation |
| Standardized quote | SADP | Standardization |
| Analytical quote | AADP | Transformation |
| Daily Stock Summary | CADP | Business consumption |

---

# 15. Load Type Metadata

The framework supports multiple load strategies.

## FULL

Processes the complete source dataset.

```text
Source
 ↓
Full Read
 ↓
Target
```

## INCREMENTAL

Processes only new or changed data.

```text
Watermark
 ↓
Source
 ↓
New Data
 ↓
Target
```

## REPROCESS

Used when previously processed data needs to be processed again.

## BACKFILL

Used to populate historical periods.

---

# 16. Watermark Metadata

Logical dataset:

```text
watermark_control
```

Recommended fields:

| Field | Description |
|---|---|
| `dataset_name` | Dataset |
| `pipeline_name` | Pipeline |
| `watermark_column` | Incremental field |
| `watermark_value` | Last successful value |
| `last_successful_run` | Successful run timestamp |
| `status` | Current state |
| `updated_timestamp` | Last update |

---

# 17. Watermark Processing

The framework follows:

```text
Read Metadata
      │
      ▼
Read Watermark
      │
      ▼
Determine Extraction Boundary
      │
      ▼
Extract Data
      │
      ▼
Process Data
      │
      ▼
Successful?
   /       \
 Yes       No
  │         │
  ▼         ▼
Update     Retain
Watermark  Previous
```

The watermark should only be advanced after successful processing.

---

# 18. Watermark Example

Assume:

```text
Dataset:
finnhub_stock_quote

Watermark Column:
event_timestamp

Previous Watermark:
2026-08-07 00:00:00
```

The next execution identifies data after the previous successful boundary according to the extraction strategy.

After successful processing:

```text
New Watermark
      │
      ▼
2026-08-08 ...
```

---

# 19. Processing Frequency

Metadata controls expected execution frequency.

Supported logical values:

```text
DAILY
MONTHLY
ON_DEMAND
```

Example:

| Dataset | Frequency |
|---|---|
| Stock Quote | DAILY |
| Company Profile | MONTHLY |
| Historical Data | DAILY / ON_DEMAND |
| KPI Generation | DAILY |

The exact frequency should match the implemented Databricks job configuration.

---

# 20. Active Flag

The `active_flag` determines whether a metadata record should participate in processing.

```text
active_flag = TRUE
```

means:

```text
Dataset eligible for processing
```

While:

```text
active_flag = FALSE
```

means:

```text
Dataset skipped
```

This allows datasets to be temporarily disabled without deleting their configuration.

---

# 21. Pipeline Metadata

Logical dataset:

```text
pipeline_metadata
```

Recommended fields:

| Field | Description |
|---|---|
| `pipeline_name` | Pipeline identifier |
| `dataset_name` | Dataset |
| `notebook_path` | Processing notebook |
| `run_type` | Scheduled/manual |
| `environment` | DEV/SIT/UAT/PROD |
| `active_flag` | Status |
| `retry_count` | Retry configuration |
| `timeout_minutes` | Execution timeout |

---

# 22. Notebook Parameterization

The project uses notebook parameters such as:

```text
env
param_name
process_name
run_type
```

Conceptually:

```text
Databricks Job
      │
      ▼
Notebook Parameters
      │
      ▼
Metadata Lookup
      │
      ▼
Dynamic Processing
```

Example:

```text
env = dev
param_name = m101_finnhub
process_name = finnhub_daily_ingestion
run_type = scheduled
```

---

# 23. Metadata Parameter Flow

```text
Job
 │
 ├── env
 ├── param_name
 ├── process_name
 └── run_type
       │
       ▼
Metadata Controller
       │
       ▼
Dataset Configuration
       │
       ▼
Processing Notebook
```

---

# 24. Dynamic Processing Framework

The target architecture is:

```text
              Job Parameters
                     │
                     ▼
             Metadata Controller
                     │
                     ▼
             Dataset Configuration
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
    Source         Load          Target
    Config         Type          Config
       │             │             │
       └─────────────┼─────────────┘
                     ▼
              Generic Processor
                     │
                     ▼
                Delta Target
```

---

# 25. Generic Processor Responsibilities

The generic processor should:

1. Read job parameters.
2. Identify the dataset.
3. Load metadata.
4. Validate metadata.
5. Read source configuration.
6. Determine load type.
7. Read watermark where applicable.
8. Extract source data.
9. Execute DQ validation.
10. Apply transformations.
11. Write target data.
12. Update watermark.
13. Write audit information.
14. Handle failures.

---

# 26. Metadata Validation

Before processing begins, metadata should be validated.

Required checks:

```text
dataset_name exists
source_system exists
target_layer exists
target_table exists
load_type is valid
active_flag is valid
```

For incremental loads:

```text
watermark_column must exist
```

Invalid metadata should stop processing rather than causing unpredictable runtime behavior.

---

# 27. Metadata Validation Flow

```text
Metadata
   │
   ▼
Validate
   │
 ┌─┴───────────┐
 ▼             ▼
Valid        Invalid
 │             │
 ▼             ▼
Process      Fail Fast
```

---

# 28. Data Quality Metadata

DQ rules can also be metadata-driven.

Logical structure:

```text
dq_metadata
```

Example fields:

| Field | Description |
|---|---|
| `rule_id` | Rule identifier |
| `dataset_name` | Dataset |
| `column_name` | Column |
| `rule_type` | NOT_NULL/RANGE/UNIQUE/etc. |
| `rule_expression` | Validation logic |
| `severity` | Critical/High/Medium/Low |
| `active_flag` | Rule status |

---

# 29. Example DQ Metadata

```json
{
  "rule_id": "DQ001",
  "dataset_name": "stock_quote",
  "column_name": "symbol",
  "rule_type": "NOT_NULL",
  "severity": "CRITICAL",
  "active_flag": true
}
```

Another example:

```json
{
  "rule_id": "DQ002",
  "dataset_name": "stock_quote",
  "column_name": "current_price",
  "rule_type": "NUMERIC",
  "severity": "HIGH",
  "active_flag": true
}
```

---

# 30. Transformation Metadata

Transformation rules may be maintained separately from dataset configuration where appropriate.

Examples:

```text
c  → current_price
d  → price_change
dp → price_change_percent
h  → day_high
l  → day_low
o  → open_price
pc → previous_close
t  → event_timestamp
```

This enables the source-specific mapping to remain explicit while the processing framework remains reusable.

---

# 31. Metadata and Medallion Architecture

Metadata determines how data moves through the layers.

```text
Finnhub
   │
   ▼
NI
   │
   │ metadata
   ▼
SADP
   │
   │ metadata
   ▼
AADP
   │
   │ metadata
   ▼
CADP
```

Each layer can have its own target and transformation configuration.

---

# 32. Metadata-Driven Ingestion

The ingestion notebook should not need to hardcode every ticker or dataset definition.

Conceptual process:

```text
Read Dataset Metadata
       │
       ▼
Get Source Configuration
       │
       ▼
Get Request Parameters
       │
       ▼
Call Finnhub API
       │
       ▼
Write NI
       │
       ▼
Audit
```

---

# 33. Metadata-Driven Transformation

```text
Read Transformation Metadata
          │
          ▼
Identify Source Columns
          │
          ▼
Apply Standardization
          │
          ▼
Apply Business Rules
          │
          ▼
Write Target
```

---

# 34. Metadata-Driven KPI Generation

CADP KPI generation can use standardized datasets and configured KPI definitions.

Example:

```text
KPI Metadata
     │
     ├── KPI Name
     ├── Source Dataset
     ├── Metric
     ├── Filter
     ├── Ranking
     └── Output Dataset
```

Example:

```text
KPI:
Top Gainers

Source:
Stock Market Data

Metric:
price_change_percent

Order:
DESC

Limit:
N
```

---

# 35. Environment Configuration Strategy

Environment-specific configuration should follow:

```text
             Common Code
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
      DEV        SIT        UAT
       │          │          │
       └──────────┼──────────┘
                  ▼
                 PROD
```

Application logic should remain consistent while environment-specific values are externalized.

---

# 36. Configuration Files

Repository structure:

```text
control_framework/
│
├── config/
│   ├── dev.json
│   ├── sit.json
│   ├── uat.json
│   └── prod.json
│
└── metadata/
```

The files should contain configuration rather than credentials.

---

# 37. Secret Management

Sensitive values such as:

```text
API keys
Access tokens
Passwords
Service credentials
```

should not be stored in:

```text
GitHub
README
Metadata tables
Plaintext JSON
Notebook source
```

The recommended implementation is to retrieve secrets through the available secret-management mechanism rather than embedding them in code.

---

# 38. Metadata Versioning

Metadata changes should be version-controlled.

Example:

```text
Version 1
   │
   ▼
Add Dataset
   │
   ▼
Version 2
   │
   ▼
Change Endpoint
   │
   ▼
Version 3
```

For Git-managed configuration, changes should be committed and reviewed like source-code changes.

---

# 39. Metadata Change Management

Changes should follow:

```text
Requirement
    │
    ▼
Metadata Change
    │
    ▼
Validation
    │
    ▼
Code Review
    │
    ▼
Testing
    │
    ▼
Deployment
```

---

# 40. Metadata Dependencies

Metadata relationships can be represented as:

```text
Source Metadata
       │
       ▼
Dataset Metadata
       │
       ├──────────────┐
       ▼              ▼
Load Metadata     Target Metadata
       │              │
       └──────┬───────┘
              ▼
       Processing Engine
              │
       ┌──────┼──────┐
       ▼      ▼      ▼
      DQ    Audit  Watermark
```

---

# 41. Metadata Processing Sequence

A typical FinPulse execution follows:

```text
1. Receive job parameters
          │
2. Read environment configuration
          │
3. Identify dataset
          │
4. Read dataset metadata
          │
5. Validate metadata
          │
6. Determine source
          │
7. Determine load strategy
          │
8. Read watermark
          │
9. Extract data
          │
10. Apply transformations
          │
11. Run DQ validation
          │
12. Write target
          │
13. Update watermark
          │
14. Write audit log
          │
15. Complete execution
```

---

# 42. Metadata Failure Handling

If metadata is missing or invalid:

```text
Metadata Lookup
      │
      ▼
Validation
      │
      ▼
Invalid
      │
      ▼
Log Error
      │
      ▼
Stop Processing
```

The system should fail fast for configuration errors because continuing with incomplete metadata could produce incorrect data.

---

# 43. Metadata and Audit

Every metadata-driven execution should be traceable.

Audit records should capture:

```text
run_id
dataset_name
pipeline_name
environment
load_type
source
target
start_timestamp
end_timestamp
status
records_read
records_written
error_message
```

This enables operational troubleshooting.

---

# 44. Metadata and Logging

Metadata provides the context required for meaningful logging.

Example:

```text
run_id:
RUN-20260808-001

dataset:
finnhub_stock_quote

source:
Finnhub

target:
SADP.stock_quote

load_type:
INCREMENTAL

status:
SUCCESS
```

---

# 45. Metadata and Error Recovery

Metadata supports controlled recovery.

Example:

```text
Pipeline Failure
      │
      ▼
Read Previous Watermark
      │
      ▼
Fix Issue
      │
      ▼
Re-run
      │
      ▼
Process From Last Successful Boundary
```

The watermark should not advance when a critical processing step fails.

---

# 46. Adding a New Dataset

A new dataset should follow:

```text
1. Identify source
       ↓
2. Define source metadata
       ↓
3. Define dataset metadata
       ↓
4. Define target metadata
       ↓
5. Define DQ rules
       ↓
6. Define transformations
       ↓
7. Register watermark if incremental
       ↓
8. Test
       ↓
9. Activate
```

The objective is to add datasets primarily through configuration plus minimal dataset-specific transformation logic.

---

# 47. Example: Adding Company Profile

Metadata:

```text
dataset_name:
finnhub_company_profile

source_system:
Finnhub

source_endpoint:
Company Profile endpoint

target_layer:
SADP

target_table:
company_profile

load_type:
FULL/INCREMENTAL according to implementation

processing_frequency:
MONTHLY

active_flag:
TRUE
```

The processing engine can use this configuration to execute the dataset.

---

# 48. Example: Stock Quote

```text
dataset_name:
finnhub_stock_quote

source_system:
Finnhub

source_endpoint:
/quote

target_layer:
SADP

target_table:
stock_quote

load_type:
INCREMENTAL

watermark_column:
event_timestamp

processing_frequency:
DAILY

active_flag:
TRUE
```

---

# 49. Metadata-Driven Design Benefits

The architecture provides:

### Reduced Hardcoding

Common logic is centralized.

### Faster Onboarding

New datasets can be registered through metadata.

### Consistent Processing

All datasets follow common operational patterns.

### Better Governance

Configuration changes are visible and traceable.

### Easier Maintenance

Changes to processing parameters do not always require notebook modifications.

### Better Scalability

The framework can support additional financial datasets and APIs.

---

# 50. Limitations

Metadata-driven processing should not attempt to eliminate all code.

Dataset-specific logic may still be required when:

- API responses have significantly different structures.
- Complex transformations are required.
- Specialized business rules are needed.
- Source authentication differs.
- Complex aggregation logic is required.

The framework should therefore use metadata for configuration and reusable behavior while retaining code for genuinely dataset-specific transformations.

---

# 51. Security Considerations

Metadata must not expose secrets.

The following should be protected:

```text
API Keys
Tokens
Credentials
Connection Secrets
```

Configuration should reference secure secret locations rather than contain secret values.

Git repository scanning should also be considered to prevent accidental credential commits.

---

# 52. Metadata Governance Rules

1. Every active dataset must have a metadata record.
2. Every incremental dataset must define a watermark strategy.
3. Every target dataset must have a defined target layer.
4. Every dataset should have DQ rules.
5. Metadata changes must be version-controlled.
6. Inactive datasets should not be processed.
7. Invalid metadata should stop processing.
8. Credentials must not be stored in metadata.
9. Metadata should remain synchronized with implementation.
10. Configuration changes should be tested before activation.

---

# 53. Metadata Entity Relationship

Conceptual relationship:

```text
                 SOURCE
                    │
                    │
                    ▼
                 DATASET
               /    │    \
              /     │     \
             ▼      ▼      ▼
           LOAD   WATERMARK  DQ
             │
             ▼
           TARGET
             │
             ▼
          PIPELINE
             │
             ▼
           AUDIT
```

---

# 54. Complete Metadata Architecture

```text
                         ┌─────────────────────┐
                         │ Environment Config  │
                         │ dev/sit/uat/prod    │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Metadata Controller │
                         └──────────┬──────────┘
                                    │
             ┌──────────────────────┼──────────────────────┐
             │                      │                      │
             ▼                      ▼                      ▼
      Source Metadata       Dataset Metadata       Pipeline Metadata
             │                      │                      │
             └──────────────────────┼──────────────────────┘
                                    │
                                    ▼
                           Processing Framework
                                    │
             ┌──────────────────────┼──────────────────────┐
             │                      │                      │
             ▼                      ▼                      ▼
        Watermark                 DQ                    Audit
             │                      │                      │
             └──────────────────────┼──────────────────────┘
                                    ▼
                              Delta Datasets
                                    │
                                    ▼
                               CADP / SQL
```

---

# 55. Metadata Acceptance Criteria

The Metadata Design is considered complete when:

1. Dataset metadata is defined.
2. Source metadata is defined.
3. Target metadata is defined.
4. Pipeline metadata is defined.
5. Watermark metadata is defined.
6. DQ metadata is defined.
7. Environment configuration is defined.
8. Active/inactive behavior is defined.
9. Load strategies are defined.
10. Metadata validation is defined.
11. Error handling is defined.
12. Audit integration is defined.
13. Secret-management expectations are documented.
14. Dataset onboarding procedure is documented.
15. Metadata relationships are documented.

---

# 56. Document Version History

| Version | Date | Author | Description |
|---|---|---|---|
| 1.0 | 2026-08-08 | Aniruddha Giri | Initial Metadata Design |

---

# 57. Conclusion

The FinPulse Metadata Design establishes a reusable configuration-driven processing framework.

The architecture separates:

```text
Configuration
      ≠
Processing Logic
```

Metadata determines **what should be processed and how it should be configured**, while reusable notebooks perform the actual processing.

The resulting architecture is:

```text
Metadata
   ↓
Controller
   ↓
Reusable Processing
   ↓
DQ
   ↓
Delta
   ↓
CADP
   ↓
Analytics
```

This design allows FinPulse to evolve from a single-source financial data project into a scalable multi-source financial data platform while maintaining consistent processing, governance, auditing, and operational controls.

**End of Metadata Design Document**