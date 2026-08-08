# FinPulse – Data Quality Framework

**Project Name:** FinPulse – Enterprise Financial Market Data Platform  
**Document:** Data Quality Framework  
**Version:** 1.0  
**Status:** Draft  
**Author:** Aniruddha Giri  
**Role:** Data Engineer  
**Platform:** Databricks Free Edition  

---

# 1. Document Purpose

This document defines the Data Quality (DQ) framework for the FinPulse financial market data platform.

The framework ensures that data processed from the Finnhub API is:

- Complete
- Accurate
- Consistent
- Valid
- Unique
- Timely
- Reliable
- Suitable for analytical consumption

Data-quality validation is performed throughout the FinPulse processing lifecycle.

```text
Finnhub API
     │
     ▼
NI
     │
     ▼
DQ Validation
     │
     ▼
SADP
     │
     ▼
DQ Validation
     │
     ▼
AADP
     │
     ▼
DQ Validation
     │
     ▼
CADP
     │
     ▼
Analytics
```

---

# 2. Business Objective

Investment analysts depend on FinPulse outputs for improved investment decisions.

Incorrect or incomplete market data can result in:

- Incorrect stock rankings
- Incorrect gainers/losers
- Incorrect market summaries
- Incorrect company comparisons
- Incorrect trading-volume analysis
- Incorrect historical trends

Therefore, data quality is treated as a core platform capability rather than an optional validation step.

---

# 3. Data Quality Objectives

The DQ framework aims to:

1. Detect invalid records early.
2. Prevent critical-quality failures from reaching CADP.
3. Identify missing and duplicate records.
4. Validate financial values.
5. Validate timestamps and business dates.
6. Validate referential consistency.
7. Track DQ results.
8. Provide actionable failure information.
9. Support data-quality monitoring.
10. Maintain an auditable record of DQ execution.

---

# 4. Data Quality Dimensions

FinPulse uses the following primary dimensions:

```text
                    Data Quality
                         │
       ┌─────────┬───────┼───────┬─────────┐
       ▼         ▼       ▼       ▼         ▼
  Completeness Accuracy Validity Uniqueness Consistency
       │
       ├───────────────┐
       ▼               ▼
   Timeliness       Integrity
```

---

# 5. Completeness

Completeness determines whether required information is present.

Examples:

```text
symbol IS NOT NULL
current_price IS NOT NULL
business_date IS NOT NULL
```

A missing critical field can make a record unusable.

---

# 6. Accuracy

Accuracy determines whether values are reasonable and correctly represented.

Examples:

```text
current_price >= 0
day_high >= day_low
```

Where source and derived values are available, they can also be compared.

---

# 7. Validity

Validity ensures values conform to expected formats and business rules.

Examples:

```text
symbol follows expected format
currency contains valid code
business_date is valid
price fields are numeric
```

---

# 8. Uniqueness

Uniqueness ensures duplicate business records are not introduced.

For a daily stock dataset, a logical business key may be:

```text
symbol + business_date
```

Duplicate records should be detected before the dataset reaches business consumption.

---

# 9. Consistency

Consistency ensures related fields agree with one another.

Example:

```text
day_high >= day_low
```

Another example:

```text
price_change =
current_price - previous_close
```

where the calculation is applicable.

---

# 10. Timeliness

Timeliness determines whether data arrives and is processed within the expected business window.

Example:

```text
Daily market data
       │
       ▼
Expected daily processing
       │
       ▼
Validate ingestion timestamp
```

Late data should be identified and logged.

---

# 11. Integrity

Integrity checks ensure relationships between datasets remain valid.

Example:

```text
Stock Quote
     │
     ▼
Company Profile
     │
     ▼
Symbol must match
```

A stock record should not reference an invalid security identifier.

---

# 12. DQ Severity Levels

FinPulse uses four logical severity levels.

| Severity | Description | Typical Action |
|---|---|---|
| Critical | Data cannot be safely processed | Stop pipeline |
| High | Significant issue affecting analytics | Reject/quarantine affected data |
| Medium | Limited business impact | Warning / continue |
| Low | Informational issue | Log only |

---

# 13. Critical DQ Rules

Critical rules are required for the pipeline to produce a valid dataset.

Examples:

```text
symbol IS NOT NULL
business_date IS NOT NULL
```

A critical DQ failure should normally prevent downstream publication.

```text
DQ Failure
    │
    ▼
Critical?
   / \
 Yes  No
 │     │
 ▼     ▼
Stop  Continue
```

---

# 14. High-Severity Rules

Examples:

```text
current_price is numeric
price_change_percent is numeric
event_timestamp is valid
```

Affected records may be rejected or quarantined depending on the implementation.

---

# 15. Medium-Severity Rules

Examples:

```text
optional company_name is missing
optional industry is missing
optional financial metric is unavailable
```

The pipeline may continue while recording the issue.

---

# 16. Low-Severity Rules

Examples:

- Informational source anomalies
- Non-critical metadata differences
- Optional formatting inconsistencies

These issues should be logged but should not normally stop processing.

---

# 17. DQ Execution Strategy

DQ validation occurs at multiple points.

```text
Source
  │
  ▼
Ingestion Validation
  │
  ▼
NI
  │
  ▼
SADP Validation
  │
  ▼
SADP
  │
  ▼
AADP Validation
  │
  ▼
AADP
  │
  ▼
CADP Validation
  │
  ▼
Publish
```

This allows problems to be detected close to where they originate.

---

# 18. Ingestion-Level Validation

Initial validation should verify that the API response is usable.

Checks include:

- API response received
- HTTP/request success
- Response structure valid
- Response is not empty
- Required source fields exist
- Source response can be parsed

Example:

```text
API Response
     │
     ▼
Response Valid?
   /       \
 Yes       No
 │          │
 ▼          ▼
Process    Error Log
```

---

# 19. NI-Level Validation

The NI layer preserves source data.

Validation includes:

- Raw response exists
- JSON is parseable
- Ingestion timestamp exists
- Source identifier exists
- Run identifier exists
- Raw payload is not unexpectedly empty

The objective is to ensure that the source response was successfully captured.

---

# 20. SADP-Level Validation

SADP validation focuses on standardized data.

Checks include:

- Required fields present
- Correct column names
- Correct data types
- Null validation
- Duplicate validation
- Numeric validation
- Timestamp conversion
- Business-date derivation

---

# 21. AADP-Level Validation

AADP validation focuses on transformed analytical data.

Checks include:

- Transformation correctness
- Derived metric validation
- Business-rule validation
- Aggregation validation
- Duplicate detection
- Unexpected record loss

---

# 22. CADP-Level Validation

CADP contains business-ready datasets.

Validation includes:

- KPI correctness
- Ranking correctness
- Aggregation correctness
- Record-count validation
- Business-key uniqueness
- Null checks
- Range checks
- Referential consistency

---

# 23. Core Stock Quote DQ Rules

The following rules apply to the standardized stock quote dataset.

| Rule ID | Field | Rule | Severity |
|---|---|---|---|
| DQ001 | symbol | Must not be null | Critical |
| DQ002 | current_price | Must be numeric | High |
| DQ003 | current_price | Must be >= 0 where applicable | High |
| DQ004 | previous_close | Must be numeric where available | High |
| DQ005 | day_high | Must be >= 0 | High |
| DQ006 | day_low | Must be >= 0 | High |
| DQ007 | day_high/day_low | High >= Low | High |
| DQ008 | event_timestamp | Must be valid | High |
| DQ009 | business_date | Must not be null | Critical |
| DQ010 | symbol + business_date | Must be unique | Critical |

---

# 24. Price Validation

Price fields should be validated as numeric values.

Example:

```text
current_price >= 0
open_price >= 0
previous_close >= 0
day_high >= 0
day_low >= 0
```

Where zero is not valid for a particular instrument or business condition, the rule can be tightened accordingly.

---

# 25. High/Low Validation

A basic market-data consistency rule is:

```text
day_high >= day_low
```

Invalid example:

```text
day_high = 320
day_low  = 330
```

Result:

```text
DQ FAIL
```

---

# 26. Open/High/Low Relationship

Where applicable:

```text
day_high >= open_price
day_high >= current_price
day_high >= day_low
```

Similarly:

```text
day_low <= open_price
day_low <= current_price
day_low <= day_high
```

These rules should account for source semantics and market conditions before being enforced as hard failures.

---

# 27. Price Change Validation

Where the source provides both current price and previous close:

```text
calculated_change =
current_price - previous_close
```

The calculated value can be compared with the source `price_change`.

Similarly:

```text
calculated_change_percent =
((current_price - previous_close) / previous_close) * 100
```

Small differences may be permitted because of rounding.

---

# 28. Volume Validation

Trading volume should normally satisfy:

```text
trading_volume >= 0
```

The value should also be numeric and compatible with the expected integer/long data type.

---

# 29. Timestamp Validation

The source timestamp should be convertible into a valid timestamp.

Example:

```text
Unix Timestamp
      │
      ▼
TIMESTAMP
      │
      ▼
DATE
```

Invalid timestamps should be rejected or quarantined.

---

# 30. Business Date Validation

Business date should:

- Be a valid date.
- Not be unexpectedly null.
- Be derived consistently.
- Align with the source event where applicable.

Example:

```text
event_timestamp
       ↓
business_date
```

---

# 31. Duplicate Validation

Duplicate detection should be based on the business key.

For daily stock data:

```text
symbol + business_date
```

Example:

```text
AAPL | 2026-08-07
AAPL | 2026-08-07
```

Result:

```text
Duplicate detected
```

The duplicate should not create multiple analytical records.

---

# 32. Company Profile DQ Rules

| Rule ID | Field | Rule | Severity |
|---|---|---|---|
| DQ020 | symbol | Must not be null | Critical |
| DQ021 | company_name | Should be populated where source provides it | Medium |
| DQ022 | country | Valid string where available | Medium |
| DQ023 | currency | Valid currency value | Medium |
| DQ024 | market_cap | Numeric where available | High |
| DQ025 | symbol | Unique per company profile snapshot | High |

---

# 33. Financial Metric DQ Rules

For fields such as:

```text
pe_ratio
eps
dividend_yield
week_52_high
week_52_low
```

checks should include:

- Numeric datatype
- Reasonable range
- Nullability according to source availability
- Correct unit
- Correct reporting period where applicable

Not every financial metric should automatically be treated as non-null because some companies legitimately have unavailable metrics.

---

# 34. Market Capitalization Validation

Market capitalization should:

```text
be numeric
```

and should have a clearly defined unit.

The platform should avoid mixing:

```text
USD
USD thousands
USD millions
USD billions
```

without explicit standardization.

---

# 35. Percentage Validation

Percentage fields must have a documented representation.

For example, the platform should consistently choose either:

```text
0.144
```

or:

```text
14.4
```

for a 14.4% movement.

The selected convention must be consistent across SADP, AADP, and CADP.

---

# 36. KPI Validation

Business KPIs require additional validation.

Examples:

### Top Gainers

Records should be ordered by:

```text
price_change_percent DESC
```

### Top Losers

Records should be ordered by:

```text
price_change_percent ASC
```

### Most Active

Records should be ordered by:

```text
trading_volume DESC
```

---

# 37. Ranking Validation

For a ranked dataset:

```text
rank = 1
rank = 2
rank = 3
...
```

The framework should verify that:

- Rank is not null.
- Rank is numeric.
- Rank is within expected range.
- Ordering matches the KPI definition.

---

# 38. Record Count Validation

Record counts should be monitored between layers.

Example:

```text
NI
1000 records
   │
   ▼
SADP
995 records
```

The difference should be explainable.

For example:

```text
1000 source records
- 5 invalid records
= 995 valid records
```

Unexpected record loss should generate a DQ warning or failure.

---

# 39. Record Reconciliation

A reconciliation framework should track:

```text
source_records
valid_records
invalid_records
duplicate_records
output_records
```

Example:

```text
Source Records       = 1000
Valid Records        = 985
Invalid Records      = 10
Duplicate Records    = 5
Output Records       = 985
```

This makes data loss explainable.

---

# 40. DQ Thresholds

Thresholds can be configured depending on the rule.

Example:

```text
Critical null threshold = 0%
```

For non-critical fields:

```text
Warning threshold = 5%
Failure threshold = 20%
```

The actual values should be configured according to business requirements rather than treated as universal financial-data standards.

---

# 41. DQ Metadata

DQ rules can be stored in metadata.

Logical structure:

```text
dq_metadata
```

| Field | Definition |
|---|---|
| `rule_id` | Unique rule |
| `dataset_name` | Target dataset |
| `column_name` | Tested column |
| `rule_type` | Rule category |
| `rule_expression` | Validation expression |
| `severity` | Critical/High/Medium/Low |
| `threshold` | Allowed threshold |
| `active_flag` | Rule status |

---

# 42. DQ Execution Flow

```text
              Dataset
                 │
                 ▼
            Load DQ Rules
                 │
                 ▼
          Execute Validations
                 │
        ┌────────┴────────┐
        ▼                 ▼
      PASS              FAIL
        │                 │
        ▼                 ▼
 Continue          Evaluate Severity
                          │
                    ┌─────┴─────┐
                    ▼           ▼
                 Critical      Non-Critical
                    │           │
                    ▼           ▼
                  Stop       Quarantine/
                             Warning
```

---

# 43. DQ Result Dataset

Logical dataset:

```text
data_quality_results
```

Recommended structure:

| Field | Type | Definition |
|---|---|---|
| `dq_run_id` | STRING | DQ execution identifier |
| `run_id` | STRING | Pipeline run |
| `dataset_name` | STRING | Dataset |
| `rule_id` | STRING | Rule identifier |
| `rule_name` | STRING | Rule name |
| `records_checked` | LONG | Records evaluated |
| `records_passed` | LONG | Passing records |
| `records_failed` | LONG | Failed records |
| `failure_percentage` | DOUBLE | Failure percentage |
| `dq_status` | STRING | PASS/FAIL/WARNING |
| `severity` | STRING | Rule severity |
| `execution_timestamp` | TIMESTAMP | Execution time |

---

# 44. DQ Status

The framework uses:

```text
PASS
WARNING
FAIL
```

### PASS

DQ requirements satisfied.

### WARNING

Issue exists but processing can continue.

### FAIL

DQ requirement violated beyond the allowed threshold.

---

# 45. Quarantine Strategy

Invalid records should be isolated where appropriate.

Logical structure:

```text
                    Input
                      │
                      ▼
                    DQ
                 /       \
              Valid     Invalid
                │          │
                ▼          ▼
             Target    Quarantine
```

Quarantine records should retain enough information to identify:

- Original dataset
- Rule failure
- Run ID
- Processing timestamp
- Original record or relevant fields

---

# 46. Quarantine Dataset

Logical dataset:

```text
dq_quarantine
```

Recommended fields:

| Field | Definition |
|---|---|
| `run_id` | Pipeline execution |
| `dataset_name` | Dataset |
| `rule_id` | Failed rule |
| `failure_reason` | Failure explanation |
| `record_data` | Failed record/payload |
| `quarantine_timestamp` | Time quarantined |

---

# 47. Fail-Fast Strategy

Critical configuration and data-quality issues should fail fast.

Example:

```text
Missing symbol
      │
      ▼
Critical DQ Failure
      │
      ▼
Stop Pipeline
      │
      ▼
Audit Failure
      │
      ▼
No CADP Publication
```

This protects business consumers from invalid datasets.

---

# 48. Warning Strategy

Non-critical issues may allow processing to continue.

Example:

```text
Missing optional industry
       │
       ▼
DQ Warning
       │
       ▼
Continue Processing
       │
       ▼
Record Warning in DQ Results
```

---

# 49. DQ and Pipeline Control

The pipeline should evaluate DQ status before advancing to the next layer.

```text
SADP
 │
 ▼
DQ
 │
 ├── FAIL ──► Stop
 │
 └── PASS/WARNING
          │
          ▼
         AADP
```

The exact policy for warnings should be configurable.

---

# 50. DQ and Watermark

The watermark should only advance after successful processing.

```text
Extract
  │
  ▼
DQ
  │
 ▼
Transform
  │
 ▼
Write Target
  │
 ▼
Success?
 /     \
Yes     No
 │       │
 ▼       ▼
Update  Keep Previous
Watermark Watermark
```

This prevents failed records from becoming the new incremental boundary.

---

# 51. DQ and Audit Integration

Every DQ execution should be associated with a pipeline `run_id`.

Example:

```text
run_id:
RUN-20260808-001

dataset:
finnhub_stock_quote

DQ:
DQ001

records_checked:
500

records_failed:
2

status:
WARNING
```

This enables complete operational traceability.

---

# 52. DQ Monitoring Metrics

Recommended monitoring metrics include:

```text
DQ Pass Rate
DQ Failure Rate
Null Rate
Duplicate Rate
Invalid Record Count
Quarantine Count
Record Reconciliation Difference
Late Data Count
```

---

# 53. DQ Dashboard Metrics

A future Databricks SQL dashboard can display:

```text
┌─────────────────────────────────────┐
│ Data Quality Overview               │
├─────────────────────────────────────┤
│ Pass Rate             98.7%         │
│ Failed Rules              3         │
│ Quarantined Records       25        │
│ Duplicate Rate          0.2%        │
│ Null Rate               0.5%        │
└─────────────────────────────────────┘
```

These values are illustrative.

---

# 54. DQ Alerting

Future production implementation may generate alerts for:

- Critical DQ failure
- High failure percentage
- Unexpected record loss
- Duplicate spike
- Missing source data
- Late ingestion
- Repeated DQ failures

---

# 55. DQ Rule Categories

Rules can be grouped into:

```text
Structural
    │
    ├── Schema
    └── Datatype

Content
    │
    ├── Null
    ├── Range
    └── Format

Business
    │
    ├── Price relationships
    ├── KPI calculations
    └── Ranking

Operational
    │
    ├── Timeliness
    └── Reconciliation
```

---

# 56. DQ Example – Stock Quote

Example input:

```text
symbol          = AAPL
current_price   = 333.74
previous_close  = 333.26
day_high        = 334.99
day_low         = 329.00
```

Validation:

```text
symbol not null                PASS
current_price numeric          PASS
current_price >= 0             PASS
day_high >= day_low             PASS
previous_close numeric          PASS
```

Result:

```text
DQ STATUS = PASS
```

---

# 57. DQ Example – Invalid Record

Example:

```text
symbol          = NULL
current_price   = "ABC"
day_high        = 300
day_low         = 350
```

Results:

```text
symbol not null       FAIL
current_price numeric FAIL
day_high >= day_low   FAIL
```

The record should be rejected or quarantined depending on the severity policy.

---

# 58. DQ Example – Duplicate

Input:

```text
AAPL | 2026-08-08
AAPL | 2026-08-08
```

Business key:

```text
symbol + business_date
```

Result:

```text
Duplicate = TRUE
DQ STATUS = FAIL
```

---

# 59. DQ Framework Architecture

```text
                         ┌───────────────┐
                         │ DQ Metadata   │
                         └───────┬───────┘
                                 │
                                 ▼
Input Dataset ───────────► DQ Engine
                                 │
                   ┌─────────────┼─────────────┐
                   ▼             ▼             ▼
              Completeness   Validity      Uniqueness
                   │             │             │
                   └─────────────┼─────────────┘
                                 ▼
                         DQ Result Processor
                                 │
                ┌────────────────┼────────────────┐
                ▼                ▼                ▼
              PASS            WARNING           FAIL
                │                │                │
                ▼                ▼                ▼
            Continue           Log            Quarantine/
                                              Stop
                                 │
                                 ▼
                         DQ Results Table
```

---

# 60. DQ Framework Integration

The DQ framework integrates with:

```text
Metadata
   │
   ├── Dataset configuration
   ├── DQ rules
   └── Thresholds
   │
   ▼
Processing
   │
   ▼
DQ Engine
   │
   ├── Results
   ├── Quarantine
   └── Status
   │
   ▼
Audit
```

---

# 61. DQ Implementation Principles

The implementation should follow:

1. Validate early.
2. Validate critical business fields.
3. Avoid unnecessary hard failures.
4. Preserve failed records where possible.
5. Make rules configurable.
6. Record DQ results.
7. Associate DQ results with pipeline runs.
8. Prevent watermark advancement after critical failure.
9. Maintain traceability.
10. Keep DQ rules aligned with the Data Dictionary.

---

# 62. Relationship With Data Dictionary

The Data Dictionary defines:

```text
What a field means
```

The DQ Framework defines:

```text
What makes the field acceptable
```

Example:

```text
Data Dictionary
current_price
= Latest security price

DQ Framework
current_price
→ numeric
→ non-negative where applicable
→ required
```

---

# 63. Relationship With Metadata

Metadata determines:

```text
Which DQ rules apply to which dataset.
```

Example:

```text
dataset_name:
stock_quote

rule:
DQ001

column:
symbol

severity:
Critical
```

---

# 64. Relationship With Audit

The DQ framework produces operational evidence that can be consumed by the audit framework.

```text
Pipeline Run
     │
     ├── Processing Logs
     ├── DQ Results
     ├── Error Logs
     └── Record Counts
```

---

# 65. DQ Acceptance Criteria

The Data Quality Framework is considered complete when:

1. DQ dimensions are defined.
2. Severity levels are defined.
3. Critical rules are identified.
4. Stock quote rules are defined.
5. Company profile rules are defined.
6. Financial metric rules are defined.
7. Duplicate rules are defined.
8. Null checks are defined.
9. Range checks are defined.
10. Timestamp validation is defined.
11. Record reconciliation is defined.
12. Quarantine strategy is defined.
13. DQ results are auditable.
14. DQ integrates with metadata.
15. DQ integrates with watermark processing.
16. DQ integrates with audit logging.

---

# 66. Document Version History

| Version | Date | Author | Description |
|---|---|---|---|
| 1.0 | 2026-08-08 | Aniruddha Giri | Initial Data Quality Framework |

---

# 67. Conclusion

The FinPulse Data Quality Framework provides a structured mechanism for identifying, controlling, and monitoring data-quality issues throughout the platform.

The framework combines:

```text
DQ Rules
   +
Metadata
   +
Validation
   +
Quarantine
   +
Audit
   +
Monitoring
```

This ensures that data progressing from:

```text
NI → SADP → AADP → CADP
```

is sufficiently reliable for downstream analytical consumption.

The framework also provides a foundation for future enhancements such as automated alerts, Databricks SQL DQ dashboards, anomaly detection, and expanded validation for additional financial data sources.

**End of Data Quality Framework**