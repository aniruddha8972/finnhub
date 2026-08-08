# FinPulse – Low-Level Design (LLD)

**Project Name:** FinPulse – Enterprise Financial Market Data Platform  
**Document:** Low-Level Design  
**Version:** 1.0  
**Status:** Draft  
**Author:** Aniruddha Giri  
**Role:** Data Engineer  
**Platform:** Databricks Free Edition  

---

# 1. Document Purpose

This Low-Level Design document defines the implementation-level design of the **FinPulse – Enterprise Financial Market Data Platform**.

The document translates the BRD, SRD, ADD, and HLD into detailed technical specifications covering:

- Notebook responsibilities
- Processing sequence
- Dataset-level processing
- Data schemas
- Transformations
- Incremental processing
- Watermark implementation
- Metadata framework
- Data-quality implementation
- Audit logging
- Error handling
- Retry behavior
- Delta Lake operations
- KPI calculations
- Configuration handling
- Failure and recovery scenarios

The LLD is intended to provide sufficient detail for a data engineer to implement or maintain the FinPulse platform.

---

# 2. Design Scope

The LLD covers the following processing stages:

```text
Finnhub API
     │
     ▼
Ingestion
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
```

Cross-cutting components:

```text
Metadata
Watermark
Data Quality
Audit
Logging
Error Handling
Configuration
```

---

# 3. Repository-Level Implementation

The implementation follows the repository structure:

```text
FinPulse/
│
├── docs/
│
├── notebooks/
│   │
│   ├── ingestion/
│   │
│   ├── consumption_SADP/
│   │
│   ├── consumption_AADP/
│   │
│   ├── consumption_CADP/
│   │
│   ├── control_framework/
│   │
│   └── metadata/
│
├── sql/
│
├── sample_data/
│
├── images/
│
├── README.md
│
└── requirements.txt
```

---

# 4. Processing Notebook Architecture

The notebooks are organized by processing responsibility.

| Notebook Area | Primary Responsibility |
|---|---|
| `ingestion/` | API extraction and NI loading |
| `consumption_SADP/` | Standardization and cleansing |
| `consumption_AADP/` | Business transformations |
| `consumption_CADP/` | KPI and reporting datasets |
| `control_framework/` | Configuration, control and operational logic |
| `metadata/` | Metadata management |
| `sql/` | Analytical queries and views |

---

# 5. End-to-End Execution Sequence

The complete processing sequence is:

```text
START
  │
  ▼
Load Environment Configuration
  │
  ▼
Load Dataset Metadata
  │
  ▼
Validate Configuration
  │
  ▼
Read Watermark
  │
  ▼
Call Finnhub API
  │
  ▼
Validate API Response
  │
  ▼
Write NI
  │
  ▼
Run SADP
  │
  ▼
Run Data Quality Checks
  │
  ▼
Write SADP
  │
  ▼
Run AADP
  │
  ▼
Write AADP
  │
  ▼
Run CADP
  │
  ▼
Generate KPIs
  │
  ▼
Write CADP
  │
  ▼
Update Metadata
  │
  ▼
Update Watermark
  │
  ▼
Write Audit = SUCCESS
  │
  ▼
END
```

Failure path:

```text
Any Critical Failure
       │
       ▼
Capture Exception
       │
       ▼
Write Error Log
       │
       ▼
Audit = FAILED
       │
       ▼
Do NOT Advance Watermark
       │
       ▼
Retry / Recovery
```

---

# 6. Notebook 1 – Ingestion

## 6.1 Purpose

The ingestion notebook retrieves financial data from the Finnhub API and persists the source information into the NI layer.

---

## 6.2 Inputs

The notebook receives configuration such as:

```text
environment
dataset_name
source_name
endpoint
symbol
load_type
watermark
retry_count
timeout
```

---

## 6.3 Processing Steps

```text
1. Initialize Spark session
2. Read configuration
3. Read metadata
4. Validate configuration
5. Read watermark
6. Build API request
7. Retrieve API response
8. Validate response
9. Add ingestion metadata
10. Persist NI data
11. Return processing statistics
```

---

# 7. API Request Design

The API request should be constructed from configuration rather than hardcoded wherever practical.

Conceptual structure:

```text
Base URL
   +
Endpoint
   +
Request Parameters
   +
Authentication
```

For example:

```text
https://<financial-api>/<endpoint>?<parameters>
```

Credentials must be retrieved securely and must not be embedded directly in source code.

---

# 8. API Response Handling

The ingestion process should validate:

1. HTTP response status.
2. Response availability.
3. Expected response structure.
4. API error information.
5. Empty response conditions.

Conceptual logic:

```text
API Response
     │
     ▼
HTTP Success?
  /       \
No         Yes
│           │
▼           ▼
Retry     Validate
             │
             ▼
       Valid Payload?
         /       \
       No         Yes
       │           │
       ▼           ▼
     Error       Continue
```

---

# 9. Ingestion Metadata

The ingestion process should attach operational metadata where applicable.

Recommended fields:

| Field | Description |
|---|---|
| `run_id` | Unique execution identifier |
| `source_system` | Source provider |
| `dataset_name` | Dataset being processed |
| `ingestion_timestamp` | Time of ingestion |
| `ingestion_date` | Ingestion business date |
| `source_endpoint` | API endpoint |
| `environment` | Execution environment |

---

# 10. NI Layer Design

The NI layer stores source-level information.

## 10.1 NI Responsibilities

- Preserve source information.
- Preserve ingestion context.
- Support reprocessing.
- Maintain source lineage.

## 10.2 NI Design Principle

Business transformations should not be applied in NI.

The preferred sequence is:

```text
Source
  ↓
Capture
  ↓
Persist
  ↓
Transform downstream
```

---

# 11. NI Storage Strategy

NI datasets should use Delta Lake where the implementation requires persistent source-level storage.

Logical organization:

```text
NI
│
├── stock_quote
├── company_profile
└── other_financial_datasets
```

---

# 12. Notebook 2 – SADP

## 12.1 Purpose

The SADP notebook converts NI data into standardized, validated financial datasets.

---

# 13. SADP Processing Sequence

```text
Read NI
  │
  ▼
Validate Schema
  │
  ▼
Parse / Flatten
  │
  ▼
Standardize Column Names
  │
  ▼
Convert Data Types
  │
  ▼
Normalize Dates / Timestamps
  │
  ▼
Handle Nulls
  │
  ▼
Deduplicate
  │
  ▼
Apply DQ Rules
  │
  ▼
Write SADP
```

---

# 14. JSON Flattening

For source responses containing nested JSON structures, the SADP layer shall flatten the relevant attributes into relational columns.

Example conceptual transformation:

```text
Nested JSON
    │
    ▼
Parse JSON
    │
    ▼
Extract Attributes
    │
    ▼
Relational DataFrame
```

The exact flattening logic depends on the source dataset.

---

# 15. Column Standardization

Source-specific field names shall be converted into consistent naming conventions.

Example:

```text
Source:
c
d
dp
h
l
o
pc
t

Standardized:
current_price
change
change_percent
day_high
day_low
open_price
previous_close
event_timestamp
```

The exact mapping shall be maintained in the Source-to-Target Mapping document.

---

# 16. Data Type Standardization

Typical target types include:

| Business Attribute | Target Type |
|---|---|
| Symbol | STRING |
| Price | DOUBLE / DECIMAL |
| Percentage | DOUBLE |
| Volume | LONG |
| Date | DATE |
| Timestamp | TIMESTAMP |
| Company Name | STRING |
| Market Capitalization | DOUBLE / DECIMAL |

The final target type should be defined in the Data Dictionary.

---

# 17. Timestamp Conversion

Source APIs may return Unix timestamps.

The SADP layer shall convert such timestamps into standard Spark date/time types.

Conceptually:

```text
Unix Timestamp
      │
      ▼
Timestamp Conversion
      │
      ▼
Spark TIMESTAMP
      │
      ▼
Business DATE where required
```

This ensures downstream datasets can perform date-based analysis consistently.

---

# 18. Deduplication

Duplicate records shall be removed using dataset-specific business keys.

For a quote dataset, a potential logical key could be:

```text
symbol + business_date + dataset_name
```

The exact key must be validated against the actual source behavior.

Deduplication should occur before publishing the standardized dataset.

---

# 19. Null Handling

The SADP layer shall identify null values.

Null handling shall be based on field semantics.

Example:

```text
Required Field
     │
     ▼
NULL?
 /   \
Yes   No
│      │
▼      ▼
Reject Continue
```

For optional fields:

```text
Optional Field
      │
      ▼
NULL
      │
      ▼
Retain / Flag
```

No meaningful financial value should be replaced with an arbitrary default unless explicitly defined by business rules.

---

# 20. SADP Data Quality

The following checks should be applied where applicable:

| Rule | Purpose |
|---|---|
| Schema validation | Verify expected structure |
| Required field check | Ensure mandatory values |
| Duplicate check | Prevent duplicate records |
| Data type validation | Ensure correct types |
| Numeric range validation | Identify invalid values |
| Timestamp validation | Ensure valid dates |
| Record-count check | Detect unexpected volume |
| Freshness check | Identify stale data |

---

# 21. Notebook 3 – AADP

## 21.1 Purpose

AADP performs business-oriented transformations on standardized financial data.

---

# 22. AADP Processing Sequence

```text
Read SADP
   │
   ▼
Apply Business Filters
   │
   ▼
Join Related Datasets
   │
   ▼
Create Derived Columns
   │
   ▼
Calculate Intermediate Metrics
   │
   ▼
Aggregate
   │
   ▼
Validate Output
   │
   ▼
Write AADP
```

---

# 23. Derived Metrics

Potential derived attributes include:

### Price Change

```text
price_change = current_price - previous_close
```

### Price Change Percentage

```text
price_change_percent =
    ((current_price - previous_close) / previous_close) * 100
```

The implementation should protect against division by zero or invalid previous-close values.

---

# 24. Historical Processing

Where historical data is available, AADP can derive:

- Daily price movement
- Historical return
- Rolling comparisons
- High/low comparisons
- Volume trends

The exact calculations depend on the available source data.

---

# 25. Company-Level Transformation

Company profile and market datasets may be combined using a common business identifier such as:

```text
symbol
```

Conceptual flow:

```text
Stock Dataset
     │
     ├──────────────┐
     │              │
     ▼              ▼
Price Data      Company Data
     │              │
     └──────┬───────┘
            ▼
        AADP Join
            │
            ▼
    Company Analytics
```

---

# 26. Notebook 4 – CADP

## 26.1 Purpose

CADP generates business-ready datasets and KPIs for analytical consumption.

---

# 27. Daily Stock Summary

The daily stock summary may contain:

```text
symbol
business_date
opening_price
closing_price
day_high
day_low
previous_close
price_change
price_change_percent
trading_volume
```

Additional attributes may be included where available.

---

# 28. Top Gainers

Top gainers are identified using positive price movement.

Processing:

```text
AADP Stock Data
      │
      ▼
Filter valid price change %
      │
      ▼
Sort DESC
      │
      ▼
Select Top N
      │
      ▼
CADP Top Gainers
```

Potential output:

```text
symbol
business_date
closing_price
price_change
price_change_percent
rank
```

---

# 29. Top Losers

Processing:

```text
AADP Stock Data
      │
      ▼
Filter valid price change %
      │
      ▼
Sort ASC
      │
      ▼
Select Top N
      │
      ▼
CADP Top Losers
```

---

# 30. Most Active Stocks

Most active stocks are identified using trading volume.

```text
AADP Stock Data
      │
      ▼
Aggregate / Select Volume
      │
      ▼
Sort DESC
      │
      ▼
Top N
      │
      ▼
CADP Most Active
```

---

# 31. Company Performance

Company-level analytical datasets may combine:

- Company information
- Market price information
- Market capitalization
- Financial metrics

Potential attributes:

```text
symbol
company_name
industry
market_cap
current_price
price_change_percent
pe_ratio
eps
dividend_yield
```

Only fields available from the relevant source dataset should be populated.

---

# 32. Sector Performance

Where sector/industry information is available, stocks may be grouped by sector.

Conceptual processing:

```text
Stock Data
    │
    ▼
Map Company → Sector
    │
    ▼
Group By Sector
    │
    ▼
Calculate Metrics
    │
    ▼
Rank Sectors
```

Potential metrics:

- Average price change %
- Total trading volume
- Number of securities
- Aggregate market capitalization

---

# 33. Historical Trend Dataset

Historical trends can be generated using time-series data.

Potential metrics include:

```text
symbol
business_date
closing_price
daily_change
daily_change_percent
trading_volume
```

This dataset supports:

- Trend analysis
- Performance comparisons
- Historical visualization
- Future ML use cases

---

# 34. Metadata Framework

The metadata framework controls dataset processing.

Logical metadata structure:

```text
metadata_id
dataset_name
source_system
source_endpoint
target_layer
target_table
load_type
watermark_column
processing_frequency
active_flag
created_timestamp
updated_timestamp
```

---

# 35. Metadata Processing

At runtime:

```text
Pipeline Start
     │
     ▼
Read Metadata
     │
     ▼
Filter Active Dataset
     │
     ▼
Load Configuration
     │
     ▼
Execute Dataset Processing
```

This enables the same processing framework to support multiple datasets.

---

# 36. Watermark Framework

The watermark framework maintains the last successfully processed boundary.

Logical structure:

```text
dataset_name
pipeline_name
watermark_value
last_successful_run
status
updated_timestamp
```

---

# 37. Watermark Algorithm

```text
1. Read current watermark.
2. Validate watermark.
3. Process records newer than watermark.
4. Complete downstream processing.
5. Validate output.
6. If successful:
       update watermark.
7. If failed:
       retain existing watermark.
```

Pseudo-logic:

```text
current_watermark = read_watermark()

data = extract(current_watermark)

if process(data) == SUCCESS:
    update_watermark(new_watermark)
else:
    keep_existing_watermark()
```

---

# 38. Watermark Failure Scenario

Example:

```text
Watermark = T1

Process T2 → T3
       │
       ▼
AADP Failure
       │
       ▼
Watermark remains T1
```

The next successful execution can safely reprocess the required range.

---

# 39. Audit Framework

The audit framework should maintain execution-level information.

Recommended structure:

```text
run_id
pipeline_name
dataset_name
layer_name
start_timestamp
end_timestamp
status
records_read
records_written
error_code
error_message
processing_duration
created_timestamp
```

---

# 40. Audit Lifecycle

```text
Pipeline Start
      │
      ▼
Create Audit Record
      │
      ▼
Status = RUNNING
      │
      ▼
Execute Processing
      │
 ┌────┴────┐
 ▼         ▼
Success   Failure
 │         │
 ▼         ▼
SUCCESS   FAILED
 │         │
 └────┬────┘
      ▼
Update Audit Record
```

---

# 41. Logging Framework

Logging should capture technical execution events.

Example:

```text
INFO  Pipeline started
INFO  Metadata loaded
INFO  Watermark loaded
INFO  API request started
INFO  API response received
INFO  NI write completed
INFO  SADP processing started
INFO  DQ validation completed
INFO  AADP processing completed
INFO  CADP processing completed
INFO  Watermark updated
INFO  Pipeline completed
```

Errors should include enough context to troubleshoot the failure.

---

# 42. Error Framework

Errors should be captured using structured information.

Recommended attributes:

```text
run_id
pipeline_name
dataset_name
layer
error_type
error_message
error_timestamp
retry_count
```

---

# 43. Retry Logic

Retry should primarily apply to transient failures.

Example:

```text
max_retry = 3

attempt = 1

while attempt <= max_retry:

    try:
        execute_operation()
        break

    except transient_error:

        attempt += 1

if attempt > max_retry:
    mark_pipeline_failed()
```

The exact implementation should use appropriate exception handling rather than catching all exceptions indiscriminately.

---

# 44. Delta Write Strategy

Delta tables are used for persistent datasets.

The write strategy depends on dataset behavior.

### Append

Use when new immutable records are continuously added.

```text
New Data
   │
   ▼
Append
   │
   ▼
Delta Table
```

### Merge

Use when records may need to be updated or upserted.

```text
Source
  │
  ▼
Match Business Key
 /          \
Match       No Match
 │             │
Update        Insert
```

### Overwrite

Should be used carefully and primarily for controlled full-refresh datasets.

---

# 45. Schema Management

The target schema should be explicitly defined for important datasets.

Schema controls should verify:

- Required columns
- Data types
- Nullable attributes
- Business keys
- Expected structure

Unexpected schema changes should be logged and handled according to dataset requirements.

---

# 46. Data Reconciliation

The platform should support basic source-to-target reconciliation.

Potential checks:

```text
Source Record Count
        │
        ▼
Processed Record Count
        │
        ▼
Target Record Count
```

Differences should be investigated when they exceed expected filtering or deduplication behavior.

---

# 47. Record Count Framework

Example:

```text
records_received
records_valid
records_rejected
records_duplicate
records_written
```

Relationship:

```text
records_received
      =
records_valid
+ records_rejected
+ records_duplicate
```

The exact relationship may vary depending on where duplicate and rejection counts are measured.

---

# 48. Data Quality Result Structure

A DQ result can conceptually contain:

```text
dq_run_id
dataset_name
rule_id
rule_name
execution_timestamp
records_checked
records_passed
records_failed
dq_status
failure_reason
```

Example statuses:

```text
PASS
FAIL
WARNING
```

---

# 49. DQ Rule Categories

## Completeness

Check required fields.

## Uniqueness

Check duplicate business keys.

## Validity

Check acceptable values and ranges.

## Consistency

Check relationships between fields.

## Freshness

Check whether data was received within the expected processing window.

## Schema

Check expected column structure.

---

# 50. Configuration Management

Environment-specific configuration should be externalized.

Example:

```text
config/
├── dev.json
├── sit.json
├── uat.json
└── prod.json
```

A configuration file may contain:

```json
{
    "environment": "dev",
    "dataset_name": "stock_quote",
    "load_type": "incremental",
    "retry_count": 3,
    "active": true
}
```

Secrets should not be stored in these files.

---

# 51. Security Implementation

## API Credentials

Credentials should be referenced securely.

Incorrect:

```python
API_KEY = "actual-secret-key"
```

Correct conceptual approach:

```python
api_key = get_secret("financial_api_key")
```

The actual secret-management mechanism depends on the available Databricks environment.

---

# 52. Git Security

The following should not be committed:

```text
API keys
Passwords
Tokens
Private credentials
Local secret files
Production secrets
```

The `.gitignore` should protect local credential/configuration files.

---

# 53. Processing Idempotency

The pipeline should avoid generating duplicate outputs when the same processing range is executed more than once.

Idempotency can be achieved through:

- Business keys
- Deduplication
- Merge/upsert logic
- Controlled watermark updates
- Run-level tracking

---

# 54. Reprocessing Strategy

If a downstream transformation fails:

```text
Raw / NI
   │
   ▼
Reprocess
   │
   ▼
SADP
   │
   ▼
AADP
   │
   ▼
CADP
```

Raw source data should allow downstream layers to be rebuilt without necessarily calling the external API again.

---

# 55. Recovery Scenarios

## Scenario 1 – API Failure

```text
API Failure
   │
   ▼
Retry
   │
   ▼
Success → Continue
   │
Failure
   ▼
Pipeline Failed
```

---

## Scenario 2 – SADP Failure

```text
NI Available
   │
   ▼
SADP Failure
   │
   ▼
Fix Transformation
   │
   ▼
Reprocess NI
```

---

## Scenario 3 – AADP Failure

```text
SADP Available
   │
   ▼
AADP Failure
   │
   ▼
Fix Transformation
   │
   ▼
Reprocess SADP
```

---

## Scenario 4 – CADP Failure

```text
AADP Available
   │
   ▼
CADP Failure
   │
   ▼
Fix KPI Logic
   │
   ▼
Reprocess AADP
```

---

# 56. Parameterization

Notebooks should use parameters rather than hardcoded dataset values where practical.

Example parameters:

```text
environment
dataset_name
run_type
process_name
source_name
load_type
business_date
```

This enables the same notebook framework to process different datasets.

---

# 57. Run Types

The system can logically support:

```text
FULL
INCREMENTAL
REPROCESS
BACKFILL
```

### FULL

Process the complete available dataset.

### INCREMENTAL

Process newly available data.

### REPROCESS

Re-run an existing processing range.

### BACKFILL

Process historical data for a specified period.

---

# 58. Business Date Handling

The platform should distinguish between:

```text
Source Timestamp
Ingestion Timestamp
Processing Timestamp
Business Date
```

These timestamps have different meanings and should not be treated as interchangeable.

---

# 59. Example Stock Quote Dataset

A standardized quote dataset may contain:

| Column | Type | Description |
|---|---|---|
| `symbol` | STRING | Security identifier |
| `current_price` | DOUBLE | Current/latest price |
| `change` | DOUBLE | Price change |
| `change_percent` | DOUBLE | Percentage price change |
| `day_high` | DOUBLE | Daily high |
| `day_low` | DOUBLE | Daily low |
| `open_price` | DOUBLE | Opening price |
| `previous_close` | DOUBLE | Previous closing price |
| `event_timestamp` | TIMESTAMP | Source event time |
| `business_date` | DATE | Business date |
| `ingestion_timestamp` | TIMESTAMP | Ingestion time |
| `run_id` | STRING | Pipeline execution ID |

This is a logical design; the final implementation schema should be validated against the actual notebook and source payload.

---

# 60. Company Profile Dataset

A company profile dataset may contain:

| Column | Type | Description |
|---|---|---|
| `symbol` | STRING | Security identifier |
| `company_name` | STRING | Company name |
| `country` | STRING | Company country |
| `currency` | STRING | Reporting currency |
| `exchange` | STRING | Exchange |
| `industry` | STRING | Industry/sector classification |
| `market_cap` | DOUBLE | Market capitalization |
| `ingestion_timestamp` | TIMESTAMP | Ingestion timestamp |
| `run_id` | STRING | Pipeline run identifier |

Only source-supported attributes should be populated.

---

# 61. CADP Daily Stock Summary

Logical target structure:

| Column | Description |
|---|---|
| `symbol` | Security |
| `business_date` | Business date |
| `opening_price` | Opening price |
| `closing_price` | Closing/current price |
| `day_high` | High |
| `day_low` | Low |
| `previous_close` | Previous close |
| `price_change` | Absolute change |
| `price_change_percent` | Percentage change |
| `trading_volume` | Trading volume |
| `load_timestamp` | Processing timestamp |

---

# 62. KPI Calculation Design

## Price Change

```text
Current Price - Previous Close
```

## Price Change Percentage

```text
(Current Price - Previous Close)
-------------------------------- × 100
       Previous Close
```

If `Previous Close = 0` or is invalid, the calculation should return a controlled null/invalid result and be captured by DQ logic.

---

# 63. Ranking Logic

## Top Gainers

```text
ORDER BY price_change_percent DESC
```

## Top Losers

```text
ORDER BY price_change_percent ASC
```

## Most Active

```text
ORDER BY trading_volume DESC
```

A configurable Top-N value should be preferred over hardcoding a specific number.

---

# 64. Spark Processing Design

PySpark should be used for transformation workloads.

Typical processing pattern:

```text
Read
 │
 ▼
Select
 │
 ▼
Filter
 │
 ▼
WithColumn
 │
 ▼
Join
 │
 ▼
GroupBy
 │
 ▼
Aggregate
 │
 ▼
Write Delta
```

Transformations should be kept modular and readable.

---

# 65. Spark Optimization Considerations

Potential optimization techniques include:

- Filter early.
- Select only required columns.
- Avoid unnecessary data movement.
- Use appropriate join strategies.
- Avoid repeated transformations.
- Cache only when reuse justifies the memory cost.
- Write only required columns.
- Process incrementally.

Optimization should be driven by actual workload characteristics rather than applied indiscriminately.

---

# 66. SQL Design

Databricks SQL should primarily query curated CADP datasets.

Example conceptual query:

```sql
SELECT
    symbol,
    business_date,
    closing_price,
    price_change_percent
FROM cadp_daily_stock_summary
WHERE business_date = CURRENT_DATE()
ORDER BY price_change_percent DESC;
```

Actual table names depend on the implemented catalog/schema structure.

---

# 67. View Design

Where appropriate, SQL views may expose business-friendly analytical datasets.

Example:

```text
CADP Table
    │
    ▼
SQL View
    │
    ▼
Investment Analyst
```

Views should simplify analytical consumption without duplicating unnecessary physical datasets.

---

# 68. End-to-End Audit Example

A successful run may generate:

```text
Run ID: RUN-20260808-001

Pipeline: daily_stock_pipeline

Source:
Finnhub API

Records Received:
N

NI Records:
N

SADP Records:
N

AADP Records:
N

CADP Records:
N

Status:
SUCCESS

Watermark:
Updated
```

Actual values are generated dynamically by the pipeline.

---

# 69. Failure Audit Example

```text
Run ID: RUN-20260808-002

Pipeline:
daily_stock_pipeline

Stage:
SADP

Status:
FAILED

Error Type:
Schema Validation

Error Message:
Unexpected source field / incompatible schema

Watermark:
Not Updated

Recovery:
Reprocess after schema correction
```

---

# 70. Dependency Flow

Processing dependencies are:

```text
Configuration
      │
      ▼
Metadata
      │
      ▼
Watermark
      │
      ▼
Ingestion
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
Analytics
```

A downstream stage should not be considered successfully completed if its required upstream stage failed.

---

# 71. Processing Status Model

The pipeline status lifecycle is:

```text
INITIATED
    │
    ▼
RUNNING
    │
 ┌──┴─────────┐
 ▼            ▼
SUCCESS      FAILED
```

Optional:

```text
PARTIAL_SUCCESS
```

may be used where individual datasets can independently succeed or fail.

---

# 72. Data Lineage Model

The logical lineage model is:

```text
source_system
      │
      ▼
dataset
      │
      ▼
pipeline
      │
      ▼
run_id
      │
      ▼
processing_layer
      │
      ▼
target_dataset
```

This lineage should be represented through metadata and audit attributes where practical.

---

# 73. Testing Considerations

The implementation should support:

### Unit Testing

Individual transformations.

### Integration Testing

Source-to-target processing.

### Data Quality Testing

Validation rules.

### Reconciliation Testing

Record-count and data comparison.

### Failure Testing

API and transformation failures.

### Recovery Testing

Reprocessing after failure.

### Regression Testing

Ensuring changes do not break existing datasets.

---

# 74. LLD Acceptance Criteria

The LLD is considered complete when:

1. Every major notebook has a defined responsibility.
2. Processing sequences are documented.
3. NI/SADP/AADP/CADP responsibilities are defined.
4. Metadata structure is defined.
5. Watermark behavior is defined.
6. DQ implementation is defined.
7. Audit structure is defined.
8. Error-handling behavior is defined.
9. Retry behavior is defined.
10. Delta write strategies are defined.
11. KPI calculation logic is documented.
12. Configuration behavior is documented.
13. Reprocessing behavior is documented.
14. Data lineage is documented.
15. Logical schemas are documented.
16. The design can be translated into implementation notebooks.

---

# 75. Relationship With Other Documents

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
HLD
 │
 ▼
LLD  ◄── Current Document
 │
 ├───────────────┐
 ▼               ▼
STM          Data Dictionary
 │               │
 └───────┬───────┘
         ▼
DQ Framework
         │
         ▼
Logging & Audit
         │
         ▼
Implementation
```

---

# 76. Document Version History

| Version | Date | Author | Description |
|---|---|---|---|
| 1.0 | 2026-08-08 | Aniruddha Giri | Initial Low-Level Design |

---

# 77. Conclusion

The FinPulse Low-Level Design defines the implementation approach for the platform's ingestion, standardization, transformation, analytical processing, control framework, and operational capabilities.

The design establishes a consistent processing pattern:

```text
Configuration
      ↓
Metadata
      ↓
Watermark
      ↓
Ingestion
      ↓
NI
      ↓
SADP
      ↓
AADP
      ↓
CADP
      ↓
Analytics
```

The LLD also establishes the technical controls required for a reliable data platform, including data-quality validation, audit logging, error handling, retry mechanisms, incremental processing, idempotency, and recovery.

The next documentation artifacts will define the **exact source-to-target mappings and data structures** used by the implementation.

**End of Low-Level Design Document**