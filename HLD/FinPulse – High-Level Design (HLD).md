# FinPulse – High-Level Design (HLD)

**Project Name:** FinPulse – Enterprise Financial Market Data Platform  
**Document:** High-Level Design  
**Version:** 1.0  
**Status:** Draft  
**Author:** Aniruddha Giri  
**Role:** Data Engineer  
**Platform:** Databricks Free Edition  

---

# 1. Document Purpose

This High-Level Design document defines the major technical components, data flows, processing responsibilities, interfaces, storage structures, and control mechanisms of the FinPulse financial market data platform.

The HLD translates the architecture defined in the Architecture Design Document (ADD) into a logical component-level design.

This document focuses on:

- System components
- Component responsibilities
- Data flow
- Processing layers
- Notebook architecture
- Metadata framework
- Watermark framework
- Data-quality framework
- Logging and audit framework
- Error handling
- Configuration management
- Storage design
- Analytical consumption

Detailed implementation logic, PySpark transformations, SQL queries, and exact schemas will be covered in the Low-Level Design (LLD).

---

# 2. System Overview

FinPulse is a batch-oriented financial data processing platform that retrieves financial datasets from the Finnhub API and processes them through multiple data layers.

The high-level processing flow is:

```text
                    ┌─────────────────┐
                    │   Finnhub API   │
                    └────────┬────────┘
                             │
                             ▼
                  ┌─────────────────────┐
                  │ Ingestion Notebook  │
                  └──────────┬──────────┘
                             │
                             ▼
                  ┌─────────────────────┐
                  │ NI / Raw Data Layer │
                  └──────────┬──────────┘
                             │
                             ▼
                  ┌─────────────────────┐
                  │  SADP Processing    │
                  │ Standardization/DQ  │
                  └──────────┬──────────┘
                             │
                             ▼
                  ┌─────────────────────┐
                  │  AADP Processing    │
                  │ Transformations     │
                  └──────────┬──────────┘
                             │
                             ▼
                  ┌─────────────────────┐
                  │  CADP Processing    │
                  │ KPIs / Analytics    │
                  └──────────┬──────────┘
                             │
                             ▼
                  ┌─────────────────────┐
                  │ Databricks SQL      │
                  └──────────┬──────────┘
                             │
                             ▼
                  Investment Analysts
```

---

# 3. HLD Architecture

The platform consists of six major functional areas:

```text
┌──────────────────────────────────────────────────────────────┐
│                        FinPulse Platform                      │
│                                                              │
│ ┌───────────────┐                                            │
│ │ Source System │                                            │
│ │ Finnhub API   │                                            │
│ └───────┬───────┘                                            │
│         │                                                     │
│         ▼                                                     │
│ ┌───────────────────────┐                                    │
│ │ Ingestion Component   │                                    │
│ └───────────┬───────────┘                                    │
│             │                                                │
│             ▼                                                │
│ ┌─────────────────────────────────────────────────────────┐  │
│ │                  Data Processing Layers                 │  │
│ │                                                         │  │
│ │ NI → SADP → AADP → CADP                                │  │
│ └──────────────────────────┬──────────────────────────────┘  │
│                            │                                 │
│                            ▼                                 │
│ ┌─────────────────────────────────────────────────────────┐  │
│ │              Analytics / Consumption                    │  │
│ │                  Databricks SQL                         │  │
│ └─────────────────────────────────────────────────────────┘  │
│                                                              │
│ ┌─────────────────────────────────────────────────────────┐  │
│ │                Control Framework                        │  │
│ │ Metadata | Watermark | DQ | Audit | Logging | Errors   │  │
│ └─────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

---

# 4. Component Inventory

| Component | Responsibility |
|---|---|
| Finnhub API | External financial data source |
| Ingestion Notebook | Extract source data |
| NI Layer | Preserve raw source data |
| SADP Layer | Standardize and cleanse data |
| AADP Layer | Apply transformations |
| CADP Layer | Generate analytical datasets and KPIs |
| Metadata Framework | Control processing configuration |
| Watermark Framework | Control incremental processing |
| DQ Framework | Validate data quality |
| Audit Framework | Track pipeline execution |
| Logging Framework | Record operational events |
| Error Handler | Manage failures |
| Databricks SQL | Analytical consumption |
| Git/GitHub | Version control |

---

# 5. External Source Component

## 5.1 Finnhub API

Finnhub is the primary upstream source for financial market information.

The API may provide datasets such as:

- Stock quotes
- Company profiles
- Market information
- Company metrics
- Historical information
- Other supported financial datasets

---

## 5.2 API Interaction

The ingestion component communicates with Finnhub through HTTPS API requests.

Logical interaction:

```text
Databricks Notebook
       │
       │ HTTPS Request
       ▼
Finnhub API
       │
       │ JSON Response
       ▼
Databricks Notebook
```

---

# 6. Ingestion Component

The ingestion notebook is responsible for extracting source data and storing it in the NI layer.

## 6.1 Responsibilities

The ingestion component shall:

1. Read configuration.
2. Read dataset metadata.
3. Read watermark where applicable.
4. Build API request.
5. Authenticate with the API.
6. Execute API call.
7. Validate response.
8. Capture ingestion metadata.
9. Persist raw response.
10. Record execution status.

---

# 7. Ingestion Processing Flow

```text
Start
 │
 ▼
Load Environment Configuration
 │
 ▼
Load Dataset Metadata
 │
 ▼
Read Watermark
 │
 ▼
Build API Request
 │
 ▼
Call Finnhub API
 │
 ├───────────────┐
 │               │
Success         Failure
 │               │
 ▼               ▼
Validate       Retry
Response         │
 │               ▼
 ▼            Retry Exhausted
Persist Raw        │
Data               ▼
 │              Mark Failed
 ▼
Audit Success
 │
 ▼
End
```

---

# 8. NI Layer

The NI layer represents the raw ingestion area.

## 8.1 Responsibilities

- Preserve source response.
- Capture ingestion timestamp.
- Capture source name.
- Capture dataset identifier.
- Maintain ingestion metadata.
- Support reprocessing.

## 8.2 Data Characteristics

NI data should generally remain close to the source representation.

Typical metadata fields:

```text
source_system
dataset_name
ingestion_timestamp
ingestion_date
run_id
source_file / endpoint
raw_payload
```

The exact structure depends on the source dataset.

---

# 9. SADP Layer

SADP is responsible for standardizing incoming data.

## 9.1 Processing Responsibilities

- Flatten JSON.
- Rename fields.
- Convert data types.
- Standardize timestamps.
- Normalize values.
- Handle nulls.
- Remove duplicates.
- Apply schema checks.
- Perform initial data-quality validation.

---

# 10. SADP Processing Flow

```text
NI Dataset
    │
    ▼
Read Raw Data
    │
    ▼
Schema Validation
    │
    ▼
Flatten / Parse
    │
    ▼
Column Standardization
    │
    ▼
Data Type Conversion
    │
    ▼
Null Handling
    │
    ▼
Deduplication
    │
    ▼
DQ Validation
    │
    ▼
SADP Delta Table
```

---

# 11. AADP Layer

AADP is responsible for analytical transformation and business processing.

## 11.1 Responsibilities

- Calculate derived fields.
- Apply business rules.
- Join related datasets.
- Aggregate data.
- Prepare intermediate analytical datasets.
- Calculate intermediate financial metrics.

---

# 12. AADP Processing Flow

```text
SADP
 │
 ├── Business Filtering
 │
 ├── Dataset Joins
 │
 ├── Derived Columns
 │
 ├── Financial Calculations
 │
 ├── Aggregations
 │
 └── Business Rules
 │
 ▼
AADP Delta Tables
```

---

# 13. CADP Layer

CADP is the final analytical/consumption layer.

## 13.1 Responsibilities

- Generate financial KPIs.
- Produce business-ready datasets.
- Prepare reporting structures.
- Aggregate financial information.
- Support Databricks SQL queries.

---

# 14. CADP Dataset Architecture

Potential CADP datasets include:

```text
CADP
│
├── daily_stock_summary
│
├── top_gainers
│
├── top_losers
│
├── most_active_stocks
│
├── company_performance
│
├── sector_performance
│
├── market_cap_analysis
│
└── historical_price_trends
```

Actual datasets should only be implemented where the underlying source data supports the required metrics.

---

# 15. Control Framework

The control framework is a cross-cutting component that supports all processing layers.

```text
                 Control Framework
                        │
        ┌───────────────┼────────────────┐
        │               │                │
        ▼               ▼                ▼
     Metadata        Watermark           DQ
        │               │                │
        └───────────────┼────────────────┘
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
          Logging                Audit
             │                     │
             └──────────┬──────────┘
                        ▼
                Processing Engine
```

---

# 16. Metadata Component

The metadata component provides configuration required by the processing framework.

## 16.1 Metadata Responsibilities

- Identify datasets.
- Identify source endpoints.
- Identify target tables.
- Define load type.
- Define processing frequency.
- Define active/inactive status.
- Define watermark information.
- Store processing parameters.

---

# 17. Logical Metadata Model

A conceptual metadata table may contain:

| Column | Description |
|---|---|
| metadata_id | Unique metadata identifier |
| dataset_name | Dataset identifier |
| source_system | Source system |
| source_endpoint | API endpoint |
| target_layer | Target processing layer |
| target_table | Target Delta table |
| load_type | FULL / INCREMENTAL |
| watermark_column | Incremental control field |
| frequency | Processing frequency |
| active_flag | Dataset status |
| created_timestamp | Metadata creation time |
| updated_timestamp | Last modification |

---

# 18. Watermark Component

The watermark component controls incremental processing.

## 18.1 Responsibilities

- Read last successful watermark.
- Provide processing boundary.
- Validate watermark.
- Update watermark after successful processing.
- Preserve previous watermark on failure.

---

# 19. Watermark Flow

```text
Watermark Table
      │
      ▼
Read Last Successful Value
      │
      ▼
Start Processing
      │
      ▼
Process Incremental Data
      │
      ▼
Processing Successful?
    /       \
  Yes        No
   │          │
   ▼          ▼
Update      Keep Existing
Watermark   Watermark
```

---

# 20. Data Quality Component

The DQ component validates datasets at appropriate processing stages.

## 20.1 DQ Checks

The framework may perform:

- Null checks
- Duplicate checks
- Schema checks
- Data type checks
- Range checks
- Validity checks
- Record-count checks
- Freshness checks

---

# 21. DQ Result Handling

```text
Dataset
   │
   ▼
DQ Rules
   │
   ▼
DQ Evaluation
   │
 ┌─┴──────────────┐
 ▼                ▼
PASS             FAIL
 │                │
 ▼                ├── Log Failure
Continue          ├── Audit Result
                  └── Reject / Flag
```

The specific action for failed records depends on the rule.

---

# 22. Logging Component

The logging component records technical execution events.

Examples:

```text
Pipeline Started
API Request Started
API Response Received
DQ Started
DQ Completed
Transformation Started
Delta Write Started
Delta Write Completed
Pipeline Completed
Pipeline Failed
```

---

# 23. Audit Component

The audit framework captures execution-level information.

A conceptual audit record:

| Attribute | Example |
|---|---|
| run_id | Unique execution ID |
| pipeline_name | daily_quote_ingestion |
| dataset_name | stock_quote |
| start_time | Execution start |
| end_time | Execution end |
| status | SUCCESS |
| records_read | Source records |
| records_written | Target records |
| error_message | Failure details |
| created_timestamp | Audit timestamp |

---

# 24. Error Handling Component

Errors are handled at different levels.

```text
                    Error Handler
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
     Source           Data            Processing
     Errors           Errors            Errors
        │                │                │
        └────────────────┼────────────────┘
                         ▼
                     Logging
                         │
                         ▼
                      Audit
                         │
                         ▼
                  Retry / Recovery
```

---

# 25. Error Categories

| Category | Examples |
|---|---|
| API | Timeout, rate limit, unavailable endpoint |
| Authentication | Invalid/expired credentials |
| Parsing | Invalid JSON |
| Schema | Unexpected fields |
| Data | Null/invalid values |
| Spark | Transformation failure |
| Delta | Write/schema conflict |
| SQL | Query failure |
| Configuration | Missing/invalid parameter |

---

# 26. Retry Component

Retries are intended primarily for transient failures.

Example policy:

```text
Attempt 1
   │
Failure
   ▼
Wait
   │
Attempt 2
   │
Failure
   ▼
Wait
   │
Attempt 3
   │
Failure
   ▼
Mark FAILED
```

Retry count should be configurable.

---

# 27. Configuration Component

Configuration is separated from processing code.

Example repository structure:

```text
control_framework/
│
├── config/
│   ├── dev.json
│   ├── sit.json
│   ├── uat.json
│   └── prod.json
```

Configuration may contain:

- Environment
- Dataset
- API endpoint
- Target layer
- Target table
- Load type
- Retry count
- Timeout
- Active flag

Credentials should not be stored directly in these files.

---

# 28. Storage Architecture

Delta Lake is used as the primary persistent storage format.

Logical organization:

```text
Delta Lake
│
├── NI
│
├── SADP
│
├── AADP
│
├── CADP
│
├── Metadata
│
├── Watermark
│
└── Audit / Logs
```

---

# 29. Storage Responsibility

| Storage Area | Responsibility |
|---|---|
| NI | Raw source data |
| SADP | Standardized data |
| AADP | Transformed data |
| CADP | Business-ready data |
| Metadata | Processing configuration |
| Watermark | Incremental control |
| Audit | Execution history |
| DQ | Data-quality results |

---

# 30. Notebook Architecture

The notebook repository is logically organized as follows:

```text
notebooks/
│
├── ingestion/
│
├── consumption_SADP/
│
├── consumption_AADP/
│
├── consumption_CADP/
│
├── control_framework/
│   ├── config/
│   └── supporting_control_notebooks/
│
└── metadata/
```

---

# 31. Notebook Responsibilities

| Notebook Area | Responsibility |
|---|---|
| ingestion | Source extraction |
| consumption_SADP | Standardization |
| consumption_AADP | Transformation |
| consumption_CADP | KPI/consumption |
| control_framework | Pipeline controls |
| metadata | Metadata management |

---

# 32. End-to-End Component Interaction

```text
                        Finnhub API
                             │
                             ▼
                    ┌─────────────────┐
                    │ Ingestion       │
                    │ Notebook        │
                    └────────┬────────┘
                             │
                             ▼
                         NI Layer
                             │
                             ▼
                    ┌─────────────────┐
                    │ SADP Notebook   │
                    └────────┬────────┘
                             │
                             ▼
                        SADP Layer
                             │
                             ▼
                    ┌─────────────────┐
                    │ AADP Notebook   │
                    └────────┬────────┘
                             │
                             ▼
                        AADP Layer
                             │
                             ▼
                    ┌─────────────────┐
                    │ CADP Notebook   │
                    └────────┬────────┘
                             │
                             ▼
                        CADP Layer
                             │
                             ▼
                     Databricks SQL
                             │
                             ▼
                   Investment Analysts
```

Control framework:

```text
              ┌────────────────────────────┐
              │      Control Framework     │
              │                            │
              │ Metadata                   │
              │ Watermark                  │
              │ Data Quality               │
              │ Logging                    │
              │ Audit                      │
              │ Error Handling             │
              └──────────────┬─────────────┘
                             │
                             ▼
                   All Processing Layers
```

---

# 33. Data Flow Between Layers

## NI → SADP

Input:

```text
Raw source payload
```

Processing:

```text
Parse
Flatten
Standardize
Validate
Deduplicate
```

Output:

```text
Standardized financial dataset
```

---

## SADP → AADP

Input:

```text
Standardized dataset
```

Processing:

```text
Business transformations
Derived metrics
Joins
Aggregations
```

Output:

```text
Analytical intermediate dataset
```

---

## AADP → CADP

Input:

```text
Transformed financial dataset
```

Processing:

```text
KPI calculation
Ranking
Aggregation
Business reporting logic
```

Output:

```text
Business-ready analytical dataset
```

---

# 34. Analytical Consumption

Databricks SQL provides the consumption interface for curated datasets.

Potential analytical queries include:

```text
Top 10 gainers
Top 10 losers
Highest trading volume
Daily stock performance
Company performance
Market capitalization ranking
Historical price trends
```

The analytical layer should consume primarily CADP datasets rather than raw source data.

---

# 35. Example Analytical Flow

```text
Stock Quote
     │
     ▼
Daily Price Change
     │
     ▼
Price Change %
     │
     ▼
Rank Stocks
     │
     ▼
Top Gainers
```

Another example:

```text
Stock Quote
     │
     ▼
Trading Volume
     │
     ▼
Aggregate by Symbol
     │
     ▼
Sort Descending
     │
     ▼
Most Active Stocks
```

---

# 36. Data Lineage

The high-level lineage is:

```text
Finnhub API
     │
     ▼
NI
     │
     ▼
SADP
     │
     ▼
AADP
     │
     ▼
CADP
     │
     ▼
Databricks SQL
     │
     ▼
Investment Analyst
```

Each downstream dataset should be traceable to its upstream dataset.

---

# 37. Security Design

The HLD security model includes:

### Credential Protection

API credentials shall not be stored in:

- Notebook source code
- Git repository
- Plain-text logs
- Public configuration files

### Configuration Separation

Configuration values and credentials should be logically separated.

### Access Control

Access to financial data and control tables should be restricted according to user responsibilities where supported by the platform.

---

# 38. Environment Design

Logical environments:

```text
DEV
 │
 ▼
SIT
 │
 ▼
UAT
 │
 ▼
PROD
```

Each environment should have its own configuration.

For the current Free Edition implementation, the project may primarily operate in DEV while preserving the architecture for future environment promotion.

---

# 39. Version Control

Git/GitHub is used for version control.

Repository responsibilities include:

- Notebook source
- SQL
- Configuration templates
- Documentation
- Sample data
- Project structure

Sensitive credentials must be excluded.

Example:

```text
.gitignore
│
├── secrets
├── credentials
├── local configuration
└── environment-specific secrets
```

---

# 40. Processing Orchestration

The logical orchestration sequence is:

```text
        Start
          │
          ▼
  Read Configuration
          │
          ▼
   Read Metadata
          │
          ▼
   Read Watermark
          │
          ▼
   Execute Ingestion
          │
          ▼
       NI Write
          │
          ▼
     SADP Process
          │
          ▼
     AADP Process
          │
          ▼
     CADP Process
          │
          ▼
     KPI Generation
          │
          ▼
    Update Metadata
          │
          ▼
    Update Watermark
          │
          ▼
     Write Audit
          │
          ▼
         End
```

If any critical stage fails:

```text
Failure
   │
   ├── Write Error Log
   │
   ├── Audit = FAILED
   │
   ├── Watermark remains unchanged
   │
   └── Trigger Recovery/Retry
```

---

# 41. High-Level Data Model

The logical relationship between major datasets is:

```text
                 Dataset Metadata
                       │
                       │ controls
                       ▼
                 Pipeline Execution
                       │
                       │ processes
                       ▼
Source Dataset ──► NI Dataset
                       │
                       ▼
                  SADP Dataset
                       │
                       ▼
                  AADP Dataset
                       │
                       ▼
                  CADP Dataset
                       │
                       ▼
                Analytical Output
```

---

# 42. Record-Level Processing

A typical financial record may follow:

```text
Finnhub Source
      │
      ▼
Raw JSON
      │
      ▼
Parsed Record
      │
      ▼
Standardized Record
      │
      ▼
Transformed Record
      │
      ▼
KPI / Analytical Record
```

Metadata such as `run_id`, processing timestamp, and source information should be added where required to support lineage and auditing.

---

# 43. HLD Performance Considerations

The platform should monitor:

- API response time
- Number of API requests
- Records processed
- Processing duration
- Spark transformation duration
- Delta write duration
- DQ processing duration

Potential optimization approaches include:

- Incremental processing
- Avoiding unnecessary full scans
- Appropriate Spark transformations
- Efficient Delta writes
- Filtering data early
- Reusing processed datasets

---

# 44. HLD Reliability Considerations

Reliability is achieved through:

- Raw data preservation
- Incremental processing
- Watermark control
- Retry mechanism
- Data-quality checks
- Audit logging
- Error logging
- Modular processing
- Recovery capability

---

# 45. HLD Scalability Considerations

The architecture can scale by:

### Horizontal Data Growth

Adding more financial instruments.

### Dataset Growth

Adding additional financial datasets.

### Source Growth

Adding additional financial APIs.

### Processing Growth

Using Spark distributed processing.

### Analytical Growth

Adding new CADP datasets and KPIs.

---

# 46. HLD Acceptance Criteria

The HLD is considered complete when:

1. All major platform components are identified.
2. Responsibilities of each component are defined.
3. Data flow between NI, SADP, AADP, and CADP is defined.
4. External API integration is defined.
5. Metadata architecture is defined.
6. Watermark architecture is defined.
7. Data-quality architecture is defined.
8. Logging and audit architecture is defined.
9. Error handling is defined.
10. Storage responsibilities are defined.
11. Analytical consumption is defined.
12. Security responsibilities are defined.
13. Repository components are mapped to architecture.
14. The design provides sufficient foundation for detailed implementation design.

---

# 47. Relationship With Other Documents

```text
BRD
 │
 ▼
SRD
 │
 ▼
ADD
 │
 ▼
HLD  ◄── Current Document
 │
 ▼
LLD
 │
 ├── STM
 ├── Data Dictionary
 ├── Metadata Design
 ├── DQ Design
 └── Logging/Audit Design
```

The HLD defines the major components and their interactions. The LLD will define the implementation-level details.

---

# 48. Document Version History

| Version | Date | Author | Description |
|---|---|---|---|
| 1.0 | 2026-08-08 | Aniruddha Giri | Initial High-Level Design |

---

# 49. Conclusion

The FinPulse High-Level Design establishes the component-level structure of the financial market data platform.

The design separates the platform into ingestion, NI, SADP, AADP, and CADP processing layers while providing centralized control through metadata, watermark, data-quality, logging, audit, and error-handling frameworks.

The architecture provides a clear separation between raw source data, standardized datasets, transformed analytical datasets, and business-ready KPIs.

This HLD provides the foundation for the next stage of documentation: the **Low-Level Design (LLD)**, which will define the implementation details of individual notebooks, transformations, schemas, control tables, processing logic, and dataset-level behavior.

**End of High-Level Design Document**