# FinPulse – Test Strategy & Test Cases

**Project Name:** FinPulse – Enterprise Financial Market Data Platform  
**Document:** Test Strategy & Test Cases  
**Version:** 1.0  
**Status:** Draft  
**Author:** Aniruddha Giri  
**Role:** Data Engineer  
**Platform:** Databricks Free Edition  

---

# 1. Document Purpose

This document defines the testing strategy and test cases for the FinPulse financial market data platform.

The objective is to ensure that:

- Financial market data is ingested correctly.
- Data is transformed correctly across processing layers.
- Data-quality rules are enforced.
- Incremental processing works correctly.
- Watermarks are maintained correctly.
- Metadata-driven processing behaves as expected.
- Audit and logging mechanisms capture execution details.
- Error handling and recovery work correctly.
- Gold-layer KPIs produce accurate results.
- No unintended duplicate or missing data is introduced.

---

# 2. Testing Objectives

The primary testing objectives are:

1. Validate source-data ingestion.
2. Validate data transformations.
3. Validate schema and datatype conversions.
4. Validate data-quality rules.
5. Validate incremental loading.
6. Validate watermark management.
7. Validate metadata-driven execution.
8. Validate error handling.
9. Validate retry mechanisms.
10. Validate audit logging.
11. Validate quarantine processing.
12. Validate Gold-layer KPI calculations.
13. Validate end-to-end data flow.
14. Validate reprocessing and recovery.

---

# 3. Scope

Testing covers the complete FinPulse pipeline:

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
```

Testing includes:

- API ingestion
- Raw data storage
- Data cleansing
- Standardization
- Transformations
- Data-quality validation
- Incremental processing
- Metadata framework
- Watermark framework
- Audit framework
- Error handling
- KPI generation
- Reprocessing

---

# 4. Out of Scope

The following are outside the primary scope of this test strategy:

- Production infrastructure performance benchmarking
- External vendor internal testing
- End-user application testing
- Mobile application testing
- Advanced ML model validation
- Real-time streaming testing

These may be addressed in future project phases.

---

# 5. Testing Levels

FinPulse uses multiple levels of testing.

```text
                 Testing
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
       Unit      Integration   System
        │           │           │
        └───────────┼───────────┘
                    ▼
               Data Quality
                    │
                    ▼
              Regression
                    │
                    ▼
                End-to-End
```

---

# 6. Unit Testing

Unit testing validates individual transformation or processing components.

Examples:

- Column transformations
- Datatype conversions
- Date conversions
- Null handling
- Deduplication logic
- Business calculations

Example:

```text
Input:
"333.74"

Expected:
333.74 DOUBLE
```

---

# 7. Integration Testing

Integration testing validates interactions between components.

Examples:

```text
Finnhub API → NI
NI → SADP
SADP → AADP
AADP → CADP
Metadata → Pipeline
Pipeline → Audit
Pipeline → Watermark
```

---

# 8. System Testing

System testing validates the complete FinPulse workflow.

Example:

```text
API
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
KPI
```

The objective is to confirm that the complete pipeline behaves correctly.

---

# 9. Data Quality Testing

Data-quality testing validates whether processed financial data satisfies defined business and technical rules.

Typical checks include:

- Null checks
- Duplicate checks
- Datatype checks
- Range checks
- Referential checks
- Business-rule checks
- Completeness checks

---

# 10. Regression Testing

Regression testing ensures that changes to one part of the platform do not break existing functionality.

Examples:

- Metadata changes
- Transformation changes
- New source fields
- New KPI logic
- Error-handling changes
- Schema changes

---

# 11. End-to-End Testing

End-to-end testing validates the complete business flow.

Example:

```text
Finnhub API
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
Business KPI
     ↓
Databricks SQL
```

The final output should be traceable back to the source data.

---

# 12. Test Environments

The project supports environment-specific configurations.

```text
DEV
SIT
UAT
PROD
```

Testing should primarily be performed in DEV and SIT before promotion to UAT/production.

---

# 13. Test Data Strategy

Test data should contain both valid and invalid scenarios.

### Valid Data

Examples:

```text
symbol = AAPL
price = 333.74
volume = 1000000
```

### Invalid Data

Examples:

```text
symbol = NULL
price = "ABC"
volume = -100
```

### Edge Cases

Examples:

```text
price = 0
volume = 0
missing optional field
duplicate record
empty API response
```

---

# 14. Test Case Structure

Each test case contains:

- Test Case ID
- Test Area
- Description
- Preconditions
- Test Steps
- Expected Result
- Priority

---

# 15. Test Case Summary

| Test Area | Number of Test Cases |
|---|---:|
| API/Ingestion | 8 |
| NI/SADP | 8 |
| AADP | 8 |
| CADP/KPI | 8 |
| Data Quality | 10 |
| Incremental/Watermark | 7 |
| Metadata | 5 |
| Audit/Logging | 5 |
| Error Handling | 7 |
| End-to-End | 5 |
| **Total** | **71** |

---

# 16. API and Ingestion Test Cases

## TC-ING-001 – Successful API Request

**Priority:** High

### Preconditions

Finnhub API is available and valid configuration exists.

### Steps

1. Execute the ingestion notebook.
2. Trigger the API request.
3. Capture the response.
4. Store the response.

### Expected Result

- API request succeeds.
- Response is received.
- Data is stored successfully.
- Audit status is updated.

---

## TC-ING-002 – API Timeout

**Priority:** High

### Steps

1. Simulate or encounter an API timeout.
2. Execute ingestion.
3. Observe retry behavior.

### Expected Result

- Timeout is detected.
- Error is logged.
- Retry mechanism is triggered.
- Pipeline succeeds if a later retry succeeds.

---

## TC-ING-003 – API Rate Limit

**Priority:** High

### Steps

1. Trigger API rate-limit condition.
2. Observe response.
3. Monitor retry behavior.

### Expected Result

- Rate-limit error is identified.
- Retry occurs after appropriate delay.
- Error is recorded in audit logs.

---

## TC-ING-004 – Authentication Failure

**Priority:** High

### Steps

1. Provide invalid API credentials.
2. Execute ingestion.

### Expected Result

- Authentication failure is detected.
- Pipeline does not continue.
- Error is logged.
- Watermark is not updated.

---

## TC-ING-005 – Empty API Response

**Priority:** Medium

### Steps

1. Return an empty API response.
2. Execute ingestion.

### Expected Result

The pipeline follows configured behavior:

- Warning if empty response is acceptable.
- Failure if data is expected.

---

## TC-ING-006 – Invalid JSON Response

**Priority:** High

### Steps

1. Provide malformed JSON.
2. Execute ingestion.

### Expected Result

- JSON parsing failure is captured.
- Error is logged.
- Pipeline fails safely.

---

## TC-ING-007 – Successful Raw Data Storage

**Priority:** High

### Steps

1. Execute API ingestion.
2. Check NI storage.

### Expected Result

Raw response is stored successfully without unintended modification.

---

## TC-ING-008 – Duplicate API Data

**Priority:** Medium

### Steps

1. Provide duplicate source records.
2. Execute ingestion.

### Expected Result

Duplicate handling follows the configured ingestion and downstream deduplication strategy.

---

# 17. NI / SADP Test Cases

## TC-SADP-001 – JSON Flattening

**Priority:** High

### Expected Result

Nested API structures are correctly flattened into relational columns.

---

## TC-SADP-002 – Column Standardization

### Expected Result

Source fields are converted to standardized target column names.

Example:

```text
c → current_price
d → price_change
dp → price_change_percentage
h → day_high
l → day_low
o → day_open
pc → previous_close
t → timestamp
```

---

## TC-SADP-003 – Datatype Conversion

### Test

Convert financial numeric fields to appropriate numeric types.

### Expected Result

Values are stored using the expected datatype.

---

## TC-SADP-004 – Timestamp Conversion

### Test

Convert Unix timestamp to a usable timestamp/date.

### Expected Result

Timestamp is converted correctly without timezone-related corruption.

---

## TC-SADP-005 – Null Handling

### Test

Provide records containing null optional fields.

### Expected Result

Null-handling rules are applied correctly.

---

## TC-SADP-006 – Duplicate Removal

### Test

Provide duplicate records.

### Expected Result

Duplicate records are removed according to the defined business key.

---

## TC-SADP-007 – Invalid Numeric Data

### Test

Provide:

```text
price = "ABC"
```

### Expected Result

Invalid record is rejected or quarantined.

---

## TC-SADP-008 – Schema Validation

### Test

Provide unexpected datatype or missing required column.

### Expected Result

Schema error is detected and logged.

---

# 18. AADP Test Cases

## TC-AADP-001 – Transformation Execution

Verify that SADP data is successfully transformed into AADP.

**Expected Result:** Transformation completes successfully.

---

## TC-AADP-002 – Business Date Derivation

Verify that business dates are derived correctly.

**Expected Result:** Correct date is generated from source timestamp.

---

## TC-AADP-003 – Price Change Calculation

Example:

```text
Current Price = 110
Previous Close = 100
```

Expected percentage change:

```text
10%
```

---

## TC-AADP-004 – Positive Price Movement

### Test

Current price > previous close.

### Expected Result

Price movement is classified as positive/gainer.

---

## TC-AADP-005 – Negative Price Movement

### Test

Current price < previous close.

### Expected Result

Price movement is classified as negative/loser.

---

## TC-AADP-006 – Zero Previous Close

### Test

Previous close = 0.

### Expected Result

Pipeline does not produce an invalid division result.

---

## TC-AADP-007 – Invalid Business Record

### Test

Provide incomplete mandatory financial data.

### Expected Result

Record is rejected or quarantined according to DQ rules.

---

## TC-AADP-008 – AADP Record Count Validation

### Expected Result

Input/output counts reconcile according to transformation rules.

---

# 19. CADP / KPI Test Cases

## TC-CADP-001 – Daily Stock Summary

Verify daily summary generation.

Expected output includes relevant fields such as:

```text
symbol
business_date
open
high
low
close
volume
price_change_percentage
```

---

## TC-CADP-002 – Top Gainers

### Test

Provide stocks with different positive price changes.

### Expected Result

Stocks are ranked correctly based on configured gain criteria.

---

## TC-CADP-003 – Top Losers

### Expected Result

Stocks with the largest negative movement are correctly identified.

---

## TC-CADP-004 – Most Active Stocks

### Test

Provide different trading volumes.

### Expected Result

Stocks are ranked correctly by trading volume.

---

## TC-CADP-005 – Sector Performance

### Test

Provide company and sector information.

### Expected Result

Sector-level performance is calculated correctly.

---

## TC-CADP-006 – Market Capitalization

### Test

Validate market-capitalization values.

### Expected Result

Values are correctly transformed and exposed for analytics.

---

## TC-CADP-007 – Historical Trend

### Test

Provide multiple dates for the same stock.

### Expected Result

Historical trend is preserved chronologically.

---

## TC-CADP-008 – Gold Duplicate Check

### Expected Result

Gold datasets contain no unintended duplicate business keys.

---

# 20. Data Quality Test Cases

## TC-DQ-001 – Mandatory Field Null Check

### Test

Set required `symbol` to NULL.

### Expected Result

Record fails the mandatory-field DQ rule.

---

## TC-DQ-002 – Duplicate Check

### Test

Insert duplicate business keys.

### Expected Result

Duplicate records are detected.

---

## TC-DQ-003 – Numeric Datatype Check

### Test

Provide string data in a numeric field.

### Expected Result

DQ failure is generated.

---

## TC-DQ-004 – Negative Volume Check

### Test

Set:

```text
volume = -100
```

### Expected Result

Record fails the volume validation rule.

---

## TC-DQ-005 – Price Range Check

### Test

Provide invalid price.

### Expected Result

Record is flagged according to configured business rules.

---

## TC-DQ-006 – Null Optional Field

### Test

Set an optional field to NULL.

### Expected Result

Record passes if the field is not mandatory.

---

## TC-DQ-007 – Completeness Check

### Test

Remove expected source records.

### Expected Result

Completeness validation identifies the discrepancy where applicable.

---

## TC-DQ-008 – DQ Threshold

### Test

Cause DQ failures above the configured threshold.

### Expected Result

Pipeline status changes according to the DQ policy.

---

## TC-DQ-009 – Quarantine Validation

### Test

Provide invalid records.

### Expected Result

Invalid records are stored in the quarantine area with failure reasons.

---

## TC-DQ-010 – DQ Audit Logging

### Expected Result

DQ results are written to the DQ audit dataset and associated with `run_id`.

---

# 21. Incremental Loading and Watermark Test Cases

## TC-WM-001 – Initial Load

### Expected Result

Initial execution processes the configured starting range.

---

## TC-WM-002 – Successful Incremental Load

### Test

Run the pipeline twice.

### Expected Result

Second execution processes only data after the previous successful watermark.

---

## TC-WM-003 – Watermark Update

### Expected Result

Watermark is updated only after successful processing.

---

## TC-WM-004 – Watermark Preservation on Failure

### Test

Force a critical pipeline failure.

### Expected Result

Previous successful watermark remains unchanged.

---

## TC-WM-005 – Duplicate Incremental Run

### Test

Execute the same processing window again.

### Expected Result

No unintended duplicate records are created.

---

## TC-WM-006 – Recovery After Failure

### Test

Fail processing and then correct the issue.

### Expected Result

Recovery processes the missing data and advances the watermark correctly.

---

## TC-WM-007 – Watermark Audit

### Expected Result

Previous and new watermark values are captured in audit records.

---

# 22. Metadata Test Cases

## TC-META-001 – Valid Metadata

### Expected Result

Pipeline reads and applies the configured metadata successfully.

---

## TC-META-002 – Missing Metadata

### Test

Remove required dataset metadata.

### Expected Result

Pipeline fails with a metadata error.

---

## TC-META-003 – Invalid Configuration

### Test

Provide an invalid target path or processing parameter.

### Expected Result

Configuration validation fails safely.

---

## TC-META-004 – Environment Configuration

### Test

Execute using DEV configuration.

### Expected Result

DEV-specific configuration is used.

---

## TC-META-005 – Metadata-Driven Dataset Processing

### Expected Result

Processing behavior changes according to metadata without requiring unnecessary notebook-code changes.

---

# 23. Audit and Logging Test Cases

## TC-AUD-001 – Run ID Generation

### Expected Result

Every execution receives a unique `run_id`.

---

## TC-AUD-002 – Start Audit

### Expected Result

Pipeline start is recorded with status:

```text
RUNNING
```

---

## TC-AUD-003 – Success Audit

### Expected Result

Successful completion updates the audit record to:

```text
SUCCESS
```

---

## TC-AUD-004 – Failure Audit

### Expected Result

Failed execution is recorded as:

```text
FAILED
```

with an associated error.

---

## TC-AUD-005 – Record Count Audit

### Expected Result

Input, output, and rejected record counts are captured.

---

# 24. Error Handling Test Cases

## TC-ERR-001 – Retry Transient Failure

### Test

Simulate API timeout.

### Expected Result

Pipeline retries according to configured retry policy.

---

## TC-ERR-002 – Retry Limit

### Test

Make every retry fail.

### Expected Result

Pipeline stops after the configured maximum retry count.

---

## TC-ERR-003 – Permanent Error

### Test

Simulate authentication failure.

### Expected Result

Pipeline does not continuously retry.

---

## TC-ERR-004 – Error Logging

### Expected Result

Error is stored with:

```text
run_id
error_code
error_type
error_message
timestamp
```

---

## TC-ERR-005 – Secret Masking

### Test

Cause an error involving API credentials.

### Expected Result

Credentials are not exposed in logs.

---

## TC-ERR-006 – Quarantine

### Expected Result

Invalid records are isolated from valid analytical datasets.

---

## TC-ERR-007 – Recovery Reprocessing

### Expected Result

Corrected data can be reprocessed successfully without introducing duplicates.

---

# 25. End-to-End Test Cases

## TC-E2E-001 – Successful Daily Pipeline

### Steps

1. Trigger ingestion.
2. Receive Finnhub data.
3. Store NI data.
4. Process SADP.
5. Process AADP.
6. Generate CADP.
7. Validate DQ.
8. Update audit.
9. Update watermark.

### Expected Result

Complete pipeline finishes successfully.

---

## TC-E2E-002 – API Failure Recovery

### Scenario

API timeout occurs during ingestion.

### Expected Result

Retry succeeds and downstream processing continues.

---

## TC-E2E-003 – DQ Failure Propagation

### Scenario

Critical DQ failure occurs in SADP.

### Expected Result

AADP/CADP processing does not consume invalid data.

---

## TC-E2E-004 – Failed Pipeline Reprocessing

### Scenario

Pipeline fails after source ingestion.

### Expected Result

After fixing the issue, the failed scope can be reprocessed successfully.

---

## TC-E2E-005 – End-to-End Data Reconciliation

### Expected Result

Source, NI, SADP, AADP, and CADP data can be reconciled according to their respective transformation rules.

---

# 26. Test Execution Process

The recommended execution process is:

```text
Test Planning
     │
     ▼
Prepare Test Data
     │
     ▼
Execute Unit Tests
     │
     ▼
Execute Integration Tests
     │
     ▼
Execute DQ Tests
     │
     ▼
Execute System Tests
     │
     ▼
Execute E2E Tests
     │
     ▼
Regression Testing
     │
     ▼
Defect Resolution
     │
     ▼
Retesting
     │
     ▼
Test Sign-Off
```

---

# 27. Defect Classification

Defects should be categorized by severity.

| Severity | Description |
|---|---|
| P1 – Critical | Complete pipeline/business functionality blocked |
| P2 – High | Major functionality incorrect |
| P3 – Medium | Limited functionality affected |
| P4 – Low | Minor issue or cosmetic/logging issue |

---

# 28. Defect Lifecycle

```text
New
 │
 ▼
Assigned
 │
 ▼
In Progress
 │
 ▼
Fixed
 │
 ▼
Retest
 │
 ├── Pass → Closed
 │
 └── Fail → Reopened
```

---

# 29. Test Evidence

For each important test, evidence should include where practical:

- Execution timestamp
- `run_id`
- Input data
- Output data
- DQ result
- Audit result
- Error logs
- Expected result
- Actual result

This improves traceability.

---

# 30. Data Reconciliation Testing

Data reconciliation should be performed between processing layers.

Example:

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

For each transition, validate:

- Record counts
- Key fields
- Datatypes
- Business dates
- Null counts
- Duplicate counts

Exact record-count equality should not be assumed for aggregation layers.

---

# 31. KPI Validation

Gold KPIs should be validated using independently calculated expected values.

Example:

```text
Input:

AAPL
Previous Close = 100
Current Close  = 110
```

Expected price movement:

```text
+10%
```

The CADP KPI should match the independently calculated result.

---

# 32. Negative Testing

Negative testing intentionally introduces invalid conditions.

Examples:

```text
Invalid API credentials
Missing configuration
Malformed JSON
Missing column
Invalid datatype
Null mandatory field
Duplicate records
Negative volume
API timeout
Delta write failure
```

Expected behavior must be safe and predictable.

---

# 33. Boundary Testing

Boundary conditions should be tested.

Examples:

```text
Price = 0
Volume = 0
Price = maximum expected value
One record
Zero records
Maximum configured batch
Exactly at DQ threshold
Just above DQ threshold
```

---

# 34. Performance Testing

Although full production-scale performance testing is outside the current scope, basic performance validation should verify:

- Pipeline completion time
- API response time
- Spark transformation duration
- Data volume handled
- Delta write duration

Performance metrics should be captured through the audit framework.

---

# 35. Regression Testing Strategy

Regression tests should be executed after changes to:

- API integration
- Schemas
- Transformations
- Metadata
- DQ rules
- Watermark logic
- KPI logic
- Audit framework
- Error handling

Priority should be given to tests covering affected components.

---

# 36. Test Automation

Where practical, repeatable tests should be automated.

Potential automation areas:

```text
Schema Validation
DQ Rules
Transformation Validation
Record Counts
Duplicate Checks
KPI Calculations
Watermark Validation
Audit Validation
```

This reduces manual regression effort.

---

# 37. Test Data Isolation

Test execution should not unintentionally modify production datasets.

Recommended approach:

```text
DEV → Development datasets
SIT → Integration datasets
UAT → User acceptance datasets
PROD → Production datasets
```

Environment-specific configuration should control target locations.

---

# 38. Test Security

Testing must ensure that:

- API credentials are protected.
- Secrets are not committed to Git.
- Secrets are not present in sample datasets.
- Secrets are not written to logs.
- Configuration files do not expose sensitive values.

---

# 39. Test Completion Criteria

Testing can be considered complete when:

1. All critical test cases pass.
2. No unresolved P1 defects remain.
3. No unresolved P2 defects remain without approved exception.
4. DQ tests pass.
5. End-to-end testing passes.
6. Incremental processing passes.
7. Watermark behavior is validated.
8. Error recovery is validated.
9. Audit logging is validated.
10. Gold KPI calculations are validated.

---

# 40. Test Sign-Off Criteria

Before promotion to the next environment:

```text
Unit Testing       → PASS
Integration        → PASS
Data Quality       → PASS
System Testing     → PASS
Regression         → PASS
E2E Testing        → PASS
Critical Defects   → 0
```

---

# 41. Overall Test Coverage

The testing framework covers:

```text
Source/API
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
KPI
   │
   ▼
Analytics
```

Cross-cutting testing covers:

```text
Metadata
DQ
Watermark
Audit
Logging
Error Handling
Recovery
Security
```

---

# 42. Test Traceability

Each major requirement should map to one or more test cases.

Example:

| Requirement | Test Cases |
|---|---|
| API ingestion | TC-ING-001 to TC-ING-008 |
| SADP transformation | TC-SADP-001 to TC-SADP-008 |
| AADP processing | TC-AADP-001 to TC-AADP-008 |
| Gold KPIs | TC-CADP-001 to TC-CADP-008 |
| Data quality | TC-DQ-001 to TC-DQ-010 |
| Incremental processing | TC-WM-001 to TC-WM-007 |
| Metadata framework | TC-META-001 to TC-META-005 |
| Audit framework | TC-AUD-001 to TC-AUD-005 |
| Error handling | TC-ERR-001 to TC-ERR-007 |
| End-to-end processing | TC-E2E-001 to TC-E2E-005 |

---

# 43. Test Strategy Summary

The FinPulse test strategy follows a layered approach:

```text
              Unit Tests
                  │
                  ▼
          Integration Tests
                  │
                  ▼
            DQ Testing
                  │
                  ▼
           System Testing
                  │
                  ▼
         Regression Testing
                  │
                  ▼
         End-to-End Testing
```

The strategy validates both technical correctness and business correctness.

---

# 44. Final Acceptance Criteria

FinPulse is considered test-ready when:

- Source ingestion is validated.
- All major transformations are validated.
- DQ rules are validated.
- Incremental loading is validated.
- Watermark behavior is validated.
- Metadata processing is validated.
- Audit logging is validated.
- Error handling is validated.
- Recovery and reprocessing are validated.
- Gold KPIs are independently validated.
- End-to-end processing is successful.
- Security controls are validated.

---

# 45. Document Version History

| Version | Date | Author | Description |
|---|---|---|---|
| 1.0 | 2026-08-08 | Aniruddha Giri | Initial Test Strategy & Test Cases |

---

# 46. Conclusion

The FinPulse Test Strategy ensures that the platform is validated across ingestion, transformation, data quality, incremental processing, business KPI generation, operational controls, and recovery.

The testing approach is designed around the platform's enterprise features:

```text
API Integration
      +
Medallion Architecture
      +
Metadata
      +
DQ
      +
Incremental Loading
      +
Watermark
      +
Audit
      +
Error Handling
      +
Recovery
      +
Business KPIs
```

This provides confidence that FinPulse can consistently transform financial market data into reliable analytics-ready datasets while detecting and recovering from operational and data-related failures.

**End of Test Strategy & Test Cases**