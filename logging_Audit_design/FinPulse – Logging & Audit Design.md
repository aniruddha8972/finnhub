# FinPulse – Logging & Audit Design

**Project Name:** FinPulse – Enterprise Financial Market Data Platform  
**Document:** Logging & Audit Design  
**Version:** 1.0  
**Status:** Draft  
**Author:** Aniruddha Giri  
**Role:** Data Engineer  
**Platform:** Databricks Free Edition  

---

# 1. Document Purpose

This document defines the logging and audit framework for the FinPulse data platform.

The framework provides operational visibility into:

- Pipeline executions
- Dataset processing
- Record counts
- Processing duration
- Data-quality results
- Errors
- Retries
- Watermark updates
- Processing status
- Source-to-target movement

The primary objective is to make every significant data-processing operation traceable.

---

# 2. Business Objective

FinPulse processes financial market data that is used by investment analysts for improved investment decisions.

Operational failures or unexpected data movement must therefore be identifiable and traceable.

The logging and audit framework enables the engineering team to answer questions such as:

- Which pipeline ran?
- When did it run?
- Which dataset was processed?
- How many records were read?
- How many records were written?
- Did DQ validation pass?
- Did the pipeline fail?
- Why did it fail?
- How long did processing take?
- Which watermark was used?
- Was the watermark updated?
- Which environment executed the process?

---

# 3. Auditability Objectives

The framework should provide:

1. End-to-end execution traceability.
2. Dataset-level processing visibility.
3. Record-count reconciliation.
4. Error traceability.
5. DQ traceability.
6. Processing-duration monitoring.
7. Watermark tracking.
8. Retry tracking.
9. Pipeline status tracking.
10. Historical operational information.

---

# 4. Logging Architecture

The logical architecture is:

```text
                     FinPulse Pipeline
                            │
                            ▼
                    Logging Controller
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
         Run Logging    DQ Logging    Error Logging
              │             │             │
              └─────────────┼─────────────┘
                            ▼
                     Audit Datasets
                            │
                            ▼
                     Databricks SQL
                            │
                            ▼
                    Monitoring / Reports
```

---

# 5. Logging Categories

FinPulse uses four primary logging categories:

```text
Pipeline Logs
DQ Logs
Error Logs
Audit Logs
```

These may be stored in separate logical datasets while sharing common identifiers.

---

# 6. Common Correlation Identifier

The primary correlation identifier is:

```text
run_id
```

Example:

```text
RUN-20260808-001
```

The `run_id` should be associated with all relevant operational records generated during a pipeline execution.

```text
run_id
   │
   ├── Pipeline Audit
   ├── DQ Results
   ├── Error Logs
   ├── Watermark Update
   └── Processing Metrics
```

---

# 7. Pipeline Audit Dataset

Logical dataset:

```text
pipeline_audit
```

The pipeline audit table provides one or more records describing each execution.

Recommended schema:

| Field | Type | Description |
|---|---|---|
| `run_id` | STRING | Unique execution identifier |
| `pipeline_name` | STRING | Pipeline name |
| `dataset_name` | STRING | Dataset being processed |
| `environment` | STRING | DEV/SIT/UAT/PROD |
| `layer_name` | STRING | NI/SADP/AADP/CADP |
| `load_type` | STRING | FULL/INCREMENTAL/etc. |
| `start_timestamp` | TIMESTAMP | Execution start |
| `end_timestamp` | TIMESTAMP | Execution end |
| `status` | STRING | RUNNING/SUCCESS/FAILED |
| `records_read` | LONG | Input record count |
| `records_written` | LONG | Output record count |
| `records_rejected` | LONG | Rejected record count |
| `processing_duration_seconds` | LONG | Processing duration |
| `error_code` | STRING | Error identifier |
| `error_message` | STRING | Error description |
| `created_timestamp` | TIMESTAMP | Audit creation timestamp |

---

# 8. Pipeline Status

The audit framework supports logical statuses:

```text
RUNNING
SUCCESS
FAILED
WARNING
CANCELLED
```

### RUNNING

Pipeline execution has started but has not completed.

### SUCCESS

Processing completed successfully.

### FAILED

Processing encountered an unrecoverable error.

### WARNING

Processing completed but non-critical issues occurred.

### CANCELLED

Execution was intentionally or externally stopped.

---

# 9. Pipeline Start Logging

At the beginning of processing, the framework should create an audit entry.

Example:

```text
run_id:
RUN-20260808-001

pipeline_name:
finnhub_daily_ingestion

dataset_name:
finnhub_stock_quote

environment:
DEV

status:
RUNNING

start_timestamp:
<execution timestamp>
```

This establishes the initial execution state.

---

# 10. Pipeline Completion Logging

After successful processing, the audit record is updated.

Example:

```text
run_id:
RUN-20260808-001

status:
SUCCESS

records_read:
500

records_written:
500

records_rejected:
0

processing_duration_seconds:
42
```

---

# 11. Pipeline Failure Logging

If processing fails:

```text
run_id:
RUN-20260808-001

status:
FAILED

error_code:
API_TIMEOUT

error_message:
Source API request timed out
```

The failure must be associated with the same `run_id`.

---

# 12. Processing Duration

Processing duration is calculated as:

```text
processing_duration =
end_timestamp - start_timestamp
```

This metric can be used to identify:

- Performance degradation
- Slow API calls
- Increasing data volume
- Transformation bottlenecks
- Infrastructure issues

---

# 13. Record Count Logging

The framework should capture:

```text
records_read
records_written
records_rejected
```

Example:

```text
Source Records       = 1000
Valid Records        = 985
Rejected Records     = 15
Target Records       = 985
```

This allows reconciliation between processing stages.

---

# 14. Record Reconciliation

A basic reconciliation check is:

```text
records_read
=
records_written
+
records_rejected
```

The exact relationship may differ when transformations involve:

- Aggregation
- Deduplication
- Filtering
- Joins
- Exploding arrays

Therefore, reconciliation rules should be defined according to the processing stage.

---

# 15. Layer-Level Auditing

FinPulse can track processing separately for each layer.

```text
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
```

Example:

| Layer | Input | Output | Status |
|---|---:|---:|---|
| NI | 1000 | 1000 | SUCCESS |
| SADP | 1000 | 995 | SUCCESS |
| AADP | 995 | 980 | SUCCESS |
| CADP | 980 | 25 | SUCCESS |

The CADP reduction may be expected because the layer can contain aggregated KPI datasets.

---

# 16. Dataset-Level Auditing

Each dataset should be identifiable in the audit framework.

Example:

```text
dataset_name:
finnhub_stock_quote
```

This enables filtering and analysis by dataset.

---

# 17. Pipeline-Level Auditing

Pipeline names should also be standardized.

Example:

```text
finnhub_daily_ingestion
finnhub_monthly_profile
aadp_market_transformation
cadp_kpi_generation
```

Actual names should follow the project's implementation naming convention.

---

# 18. Environment Auditing

Each execution should identify its environment.

Supported environments:

```text
DEV
SIT
UAT
PROD
```

Example:

```text
environment = DEV
```

This prevents confusion when analyzing operational history.

---

# 19. Load-Type Auditing

The audit record should identify how the dataset was loaded.

Examples:

```text
FULL
INCREMENTAL
REPROCESS
BACKFILL
```

Example:

```text
load_type = INCREMENTAL
```

This is particularly important when investigating duplicate or missing data.

---

# 20. Watermark Auditing

The audit framework should capture watermark information where applicable.

Recommended fields:

| Field | Description |
|---|---|
| `watermark_column` | Incremental field |
| `previous_watermark` | Previous successful value |
| `new_watermark` | New value after successful processing |
| `watermark_updated` | TRUE/FALSE |

---

# 21. Watermark Audit Flow

```text
Read Previous Watermark
          │
          ▼
       Process
          │
          ▼
      Successful?
       /      \
     Yes       No
      │         │
      ▼         ▼
Update       Do Not Update
Watermark       │
      │          │
      └────┬─────┘
           ▼
       Audit Result
```

---

# 22. DQ Logging

Data-quality results should be associated with the pipeline execution.

Logical dataset:

```text
data_quality_results
```

Recommended fields:

| Field | Type | Description |
|---|---|---|
| `dq_run_id` | STRING | DQ execution ID |
| `run_id` | STRING | Pipeline execution ID |
| `dataset_name` | STRING | Dataset |
| `rule_id` | STRING | DQ rule |
| `rule_name` | STRING | Rule description |
| `records_checked` | LONG | Records evaluated |
| `records_passed` | LONG | Passing records |
| `records_failed` | LONG | Failed records |
| `failure_percentage` | DOUBLE | Failure percentage |
| `dq_status` | STRING | PASS/WARNING/FAIL |
| `severity` | STRING | Rule severity |
| `execution_timestamp` | TIMESTAMP | Execution time |

---

# 23. Error Logging

Logical dataset:

```text
error_log
```

Recommended schema:

| Field | Type | Description |
|---|---|---|
| `error_id` | STRING | Unique error identifier |
| `run_id` | STRING | Pipeline run |
| `pipeline_name` | STRING | Pipeline |
| `dataset_name` | STRING | Dataset |
| `layer_name` | STRING | Processing layer |
| `error_type` | STRING | Error category |
| `error_code` | STRING | Error identifier |
| `error_message` | STRING | Error description |
| `retry_count` | INTEGER | Number of retries |
| `error_timestamp` | TIMESTAMP | Error occurrence |
| `stack_trace` | STRING | Technical error details where appropriate |

---

# 24. Error Categories

Common categories include:

```text
API_ERROR
API_TIMEOUT
AUTHENTICATION_ERROR
SCHEMA_ERROR
DATA_QUALITY_ERROR
TRANSFORMATION_ERROR
DELTA_WRITE_ERROR
CONFIGURATION_ERROR
METADATA_ERROR
SYSTEM_ERROR
```

---

# 25. Error Logging Example

```text
run_id:
RUN-20260808-001

pipeline_name:
finnhub_daily_ingestion

dataset_name:
finnhub_stock_quote

error_type:
API_TIMEOUT

error_code:
API_TIMEOUT_001

retry_count:
2

status:
FAILED
```

---

# 26. Retry Logging

Every retry should remain associated with the original `run_id`.

Example:

```text
Attempt 1 → FAILED
Attempt 2 → FAILED
Attempt 3 → SUCCESS
```

The audit framework should make this execution history visible.

Recommended field:

```text
retry_count
```

---

# 27. Retry Flow

```text
Pipeline
   │
   ▼
Attempt 1
   │
   ▼
Failure
   │
   ▼
Retry
   │
   ▼
Attempt 2
   │
   ▼
Success
```

Retry should only be used for errors that are considered transient.

---

# 28. Transient vs Permanent Errors

### Transient

Examples:

- Temporary API timeout
- Temporary network failure
- Temporary service unavailability

These may be retried.

### Permanent

Examples:

- Invalid configuration
- Invalid schema
- Missing required metadata
- Authentication failure
- Invalid transformation logic

These generally should not be repeatedly retried without intervention.

---

# 29. Logging Levels

The application can use logical logging levels:

```text
INFO
WARNING
ERROR
DEBUG
```

### INFO

Normal execution events.

### WARNING

Non-critical anomalies.

### ERROR

Failures requiring investigation.

### DEBUG

Detailed technical information useful during development.

Production logging should avoid excessive debug output.

---

# 30. Example INFO Events

```text
Pipeline started
Metadata loaded
Watermark retrieved
API request initiated
Records received
Transformation completed
Target write completed
Pipeline completed
```

---

# 31. Example WARNING Events

```text
Optional field missing
Small number of rejected records
DQ warning threshold exceeded
Unexpected but non-critical source variation
```

---

# 32. Example ERROR Events

```text
API request failed
Metadata not found
Schema mismatch
Critical DQ failure
Delta write failure
Transformation exception
```

---

# 33. Structured Logging

Logs should preferably use structured fields rather than only free-text messages.

Example:

```text
run_id = RUN-001
dataset = stock_quote
layer = SADP
status = FAILED
error_code = DQ001
```

This makes logs easier to query using SQL.

---

# 34. Logging Controller

A reusable logging component can provide functions such as:

```text
start_run()
update_run_status()
log_record_counts()
log_dq_result()
log_error()
log_retry()
complete_run()
```

Conceptual implementation:

```text
Pipeline
   │
   ▼
Logging Utility
   │
   ├── Start
   ├── Update
   ├── Error
   ├── DQ
   └── Complete
```

---

# 35. Audit Utility Design

A reusable utility should accept parameters such as:

```text
run_id
pipeline_name
dataset_name
layer_name
status
records_read
records_written
records_rejected
error_code
error_message
```

This avoids duplicating audit logic across notebooks.

---

# 36. Audit Lifecycle

```text
                START
                  │
                  ▼
             Create Run
                  │
                  ▼
             RUNNING
                  │
         ┌────────┴────────┐
         ▼                 ▼
      SUCCESS            FAILURE
         │                 │
         ▼                 ▼
      Complete          Log Error
         │                 │
         ▼                 ▼
      SUCCESS            FAILED
```

---

# 37. Pipeline Audit Example

Example record:

```text
run_id:
RUN-20260808-001

pipeline_name:
finnhub_daily_ingestion

dataset_name:
finnhub_stock_quote

environment:
DEV

layer_name:
SADP

load_type:
INCREMENTAL

start_timestamp:
2026-08-08 09:00:00

end_timestamp:
2026-08-08 09:00:42

status:
SUCCESS

records_read:
500

records_written:
500

records_rejected:
0

processing_duration_seconds:
42
```

---

# 38. Error Audit Example

```text
run_id:
RUN-20260808-002

pipeline_name:
finnhub_daily_ingestion

dataset_name:
finnhub_stock_quote

layer_name:
NI

status:
FAILED

error_code:
API_TIMEOUT_001

error_message:
Source API request timed out

retry_count:
3
```

---

# 39. DQ Audit Example

```text
run_id:
RUN-20260808-003

dataset_name:
stock_quote

rule_id:
DQ010

rule_name:
Duplicate Business Key

records_checked:
500

records_failed:
3

failure_percentage:
0.60

dq_status:
WARNING

severity:
High
```

---

# 40. Data Lineage Through Audit

The audit framework provides a simplified operational lineage:

```text
Source
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
```

Each step can be associated with:

```text
run_id
dataset_name
layer_name
timestamp
record_count
status
```

---

# 41. Source-to-Target Audit

Example:

```text
Finnhub /quote
       │
       ▼
NI.stock_quote_raw
       │
       ▼
SADP.stock_quote
       │
       ▼
AADP.market_data
       │
       ▼
CADP.daily_stock_summary
```

Audit records can connect each stage using execution and dataset identifiers.

---

# 42. Audit Retention

Audit data should normally be retained longer than transient application logs because it provides historical operational evidence.

Retention should be determined based on:

- Project requirements
- Storage availability
- Operational needs
- Compliance requirements if applicable

The exact retention period should be configured before production deployment.

---

# 43. Audit Security

Audit records may contain technical information such as:

- Error messages
- Notebook paths
- Dataset names
- Configuration information

They should therefore be accessible only to appropriate users.

Secrets should never appear in:

```text
Audit logs
Error messages
Stack traces
DQ results
```

---

# 44. Secret Masking

Sensitive values should be masked.

Instead of:

```text
api_key=abc123456
```

logs should contain:

```text
api_key=********
```

The actual secret should never be written to operational logs.

---

# 45. Operational Monitoring

The audit datasets can be queried using Databricks SQL.

Example monitoring metrics:

```text
Total Runs
Successful Runs
Failed Runs
Average Processing Time
Records Processed
Records Rejected
DQ Failures
Retry Count
```

---

# 46. Pipeline Success Rate

A useful operational KPI is:

```text
success_rate =
successful_runs / total_runs × 100
```

Example:

```text
Successful Runs = 98
Total Runs      = 100

Success Rate = 98%
```

---

# 47. Failure Rate

```text
failure_rate =
failed_runs / total_runs × 100
```

This can be monitored over time to identify recurring operational issues.

---

# 48. Average Processing Duration

```text
average_duration =
SUM(processing_duration)
/
COUNT(successful_runs)
```

An increase in processing duration may indicate:

- Increased data volume
- API slowdown
- Transformation inefficiency
- Resource constraints

---

# 49. Record Processing KPI

Operational monitoring can track:

```text
total_records_read
total_records_written
total_records_rejected
```

This can identify unexpected changes in source volume.

---

# 50. Audit Monitoring Dashboard

A future Databricks SQL dashboard may contain:

```text
┌─────────────────────────────────────────┐
│ FinPulse Pipeline Operations            │
├─────────────────────────────────────────┤
│ Total Runs              100             │
│ Success Rate             98%            │
│ Failed Runs               2             │
│ Avg Duration             45 sec         │
│ Records Processed      50,000           │
│ DQ Failures               3             │
└─────────────────────────────────────────┘
```

Values are illustrative.

---

# 51. Failure Trend Monitoring

Historical audit data can identify recurring failures.

Example:

```text
Date        Failed Runs
2026-08-05      1
2026-08-06      0
2026-08-07      3
2026-08-08      1
```

This helps engineering teams identify patterns.

---

# 52. Operational SLA Monitoring

The audit framework can support future SLA measurements.

Example:

```text
Expected completion:
Before 08:00

Actual completion:
07:42

SLA:
MET
```

Late completion should generate a warning or alert.

---

# 53. Audit and DQ Integration

The relationship is:

```text
Pipeline Run
     │
     ├──────────────► Audit
     │
     ├──────────────► DQ
     │
     ├──────────────► Error
     │
     └──────────────► Watermark
```

All operational records should share common identifiers where possible.

---

# 54. Audit and Error Handling

When a failure occurs:

```text
Error
 │
 ├── Error Log
 │
 ├── Pipeline Audit = FAILED
 │
 ├── Retry Information
 │
 └── Watermark NOT Updated
```

This maintains operational consistency.

---

# 55. Audit and Metadata

Metadata provides context for the audit framework.

Example:

```text
Metadata:
dataset = stock_quote
load_type = incremental
target = SADP.stock_quote
```

Audit records then capture:

```text
run_id
dataset
load_type
target
status
record counts
duration
```

---

# 56. Audit and Data Quality

DQ results are linked to pipeline executions.

```text
run_id
   │
   ├── pipeline_audit
   │
   └── data_quality_results
```

This allows engineers to investigate exactly which DQ rules affected a failed or warning execution.

---

# 57. Audit Investigation Workflow

When a pipeline fails:

```text
1. Find run_id
       ↓
2. Open pipeline_audit
       ↓
3. Identify failed layer
       ↓
4. Check error_log
       ↓
5. Check DQ results
       ↓
6. Check watermark
       ↓
7. Identify root cause
       ↓
8. Resolve issue
       ↓
9. Reprocess
```

---

# 58. Root Cause Analysis

Audit information should help identify:

```text
WHAT failed?
WHEN did it fail?
WHERE did it fail?
WHY did it fail?
HOW many records were affected?
WAS the watermark updated?
WAS retry attempted?
```

---

# 59. Logging Best Practices

1. Use a common `run_id`.
2. Use structured logging.
3. Log start and completion events.
4. Log record counts.
5. Log DQ results.
6. Log errors with codes.
7. Track retry attempts.
8. Never log secrets.
9. Keep logs searchable.
10. Avoid excessive debug logging.
11. Record processing duration.
12. Maintain consistent naming.

---

# 60. Audit Data Model

```text
                         RUN
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
        PIPELINE_AUDIT  DQ_RESULTS  ERROR_LOG
             │            │            │
             └────────────┼────────────┘
                          │
                          ▼
                    WATERMARK
```

---

# 61. Recommended Audit Tables

The FinPulse platform can maintain:

```text
pipeline_audit
data_quality_results
error_log
watermark_control
```

Optional future tables:

```text
pipeline_metrics
execution_history
alert_history
```

---

# 62. Logging Flow

Complete execution:

```text
Job Trigger
    │
    ▼
Generate run_id
    │
    ▼
Write RUNNING audit
    │
    ▼
Read Metadata
    │
    ▼
Read Watermark
    │
    ▼
Execute Pipeline
    │
    ├──────────────┐
    ▼              ▼
   DQ             Error
    │              │
    ▼              ▼
Write Results    Write Error
    │              │
    └───────┬──────┘
            ▼
       Final Status
            │
      ┌─────┴─────┐
      ▼           ▼
   SUCCESS       FAILED
      │           │
      ▼           ▼
Update         Retain
Watermark      Watermark
```

---

# 63. Example End-to-End Audit

```text
Run ID:
RUN-20260808-001

Pipeline:
finnhub_daily_ingestion

Dataset:
finnhub_stock_quote

Environment:
DEV

Load Type:
INCREMENTAL

Input Records:
500

DQ Failed:
2

Rejected:
2

Output Records:
498

Processing Duration:
42 seconds

Final Status:
WARNING

Watermark Updated:
YES
```

Whether a warning execution is allowed to update the watermark should be determined by the DQ policy. Critical failures must not advance it.

---

# 64. Logging Framework Acceptance Criteria

The framework is considered complete when:

1. Every pipeline execution has a `run_id`.
2. Pipeline start is logged.
3. Pipeline completion is logged.
4. Pipeline failures are logged.
5. Record counts are captured.
6. Processing duration is captured.
7. DQ results are associated with executions.
8. Errors are captured separately.
9. Retry attempts are tracked.
10. Watermark changes are auditable.
11. Environment is captured.
12. Layer is captured.
13. Dataset is captured.
14. Secrets are excluded from logs.
15. Operational metrics can be queried.

---

# 65. Document Version History

| Version | Date | Author | Description |
|---|---|---|---|
| 1.0 | 2026-08-08 | Aniruddha Giri | Initial Logging & Audit Design |

---

# 66. Conclusion

The FinPulse Logging & Audit framework provides operational traceability across the complete data lifecycle.

The framework connects:

```text
Metadata
   │
   ▼
Pipeline
   │
   ├── Audit
   ├── DQ
   ├── Error Logging
   └── Watermark
   │
   ▼
Operational Monitoring
```

By maintaining consistent `run_id`, dataset, layer, status, record-count, error, and timing information, FinPulse can support effective troubleshooting and operational monitoring.

The design also provides a foundation for future Databricks SQL dashboards, alerting, SLA monitoring, automated incident detection, and production-grade observability.

**End of Logging & Audit Design**