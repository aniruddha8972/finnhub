# FinPulse – Data Dictionary

**Project Name:** FinPulse – Enterprise Financial Market Data Platform  
**Document:** Data Dictionary  
**Version:** 1.0  
**Status:** Draft  
**Author:** Aniruddha Giri  
**Role:** Data Engineer  
**Platform:** Databricks Free Edition  

---

# 1. Document Purpose

This Data Dictionary defines the business and technical meaning of the fields used throughout the FinPulse data platform.

It provides a common reference for:

- Data engineers
- Investment analysts
- Data-quality teams
- Developers
- Technical reviewers
- Project maintainers

The document defines:

- Column names
- Business definitions
- Data types
- Nullable behavior
- Source
- Transformation
- Business keys
- Example values
- Data-quality expectations

---

# 2. Data Layer Reference

FinPulse follows the following logical data flow:

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

---

# 3. Naming Standards

The platform follows `snake_case` naming for standardized and analytical datasets.

Examples:

```text
current_price
previous_close
price_change_percent
business_date
ingestion_timestamp
market_cap
```

Source-specific field names may remain unchanged in the NI layer to preserve source lineage.

---

# 4. Data Type Standards

| Business Data | Recommended Spark Type |
|---|---|
| Identifier | STRING |
| Name | STRING |
| Country | STRING |
| Currency | STRING |
| Exchange | STRING |
| Price | DOUBLE / DECIMAL |
| Percentage | DOUBLE |
| Volume | LONG |
| Market Capitalization | DOUBLE / DECIMAL |
| Date | DATE |
| Timestamp | TIMESTAMP |
| Rank | INTEGER / LONG |
| Boolean Flag | BOOLEAN |

For high-precision financial calculations, `DECIMAL` should be preferred where appropriate.

---

# 5. Stock Quote Dataset

The Stock Quote dataset contains current/latest market information for a security.

Logical target:

```text
stock_quote
```

---

## 5.1 `symbol`

| Attribute | Definition |
|---|---|
| Field | `symbol` |
| Data Type | STRING |
| Nullable | No |
| Business Definition | Unique security/ticker identifier |
| Source | Request parameter / source context |
| Layer | SADP / AADP / CADP |
| Example | `AAPL` |
| Business Key | Yes |
| DQ | Not null |

The symbol is used to associate market data with the corresponding security.

---

## 5.2 `current_price`

| Attribute | Definition |
|---|---|
| Field | `current_price` |
| Data Type | DOUBLE / DECIMAL |
| Nullable | No |
| Business Definition | Latest/current price returned by the source |
| Source Field | `c` |
| Layer | SADP / AADP |
| Example | `333.74` |
| DQ | Numeric |

Source lineage:

```text
c
↓
current_price
```

---

## 5.3 `price_change`

| Attribute | Definition |
|---|---|
| Field | `price_change` |
| Data Type | DOUBLE / DECIMAL |
| Nullable | Yes |
| Business Definition | Absolute price change compared with previous close |
| Source Field | `d` |
| Example | `0.48` |
| DQ | Numeric |

Conceptual calculation:

```text
current_price - previous_close
```

---

## 5.4 `price_change_percent`

| Attribute | Definition |
|---|---|
| Field | `price_change_percent` |
| Data Type | DOUBLE |
| Nullable | Yes |
| Business Definition | Percentage movement relative to previous close |
| Source Field | `dp` |
| Example | `0.144` |
| DQ | Numeric |

Conceptual calculation:

```text
((current_price - previous_close) / previous_close) × 100
```

The source value may also be compared with the calculated value for validation.

---

## 5.5 `day_high`

| Attribute | Definition |
|---|---|
| Field | `day_high` |
| Data Type | DOUBLE / DECIMAL |
| Nullable | Yes |
| Business Definition | Highest reported price for the trading period |
| Source Field | `h` |
| Example | `334.99` |
| DQ | Numeric |

---

## 5.6 `day_low`

| Attribute | Definition |
|---|---|
| Field | `day_low` |
| Data Type | DOUBLE / DECIMAL |
| Nullable | Yes |
| Business Definition | Lowest reported price for the trading period |
| Source Field | `l` |
| Example | `329.0006` |
| DQ | Numeric |

---

## 5.7 `open_price`

| Attribute | Definition |
|---|---|
| Field | `open_price` |
| Data Type | DOUBLE / DECIMAL |
| Nullable | Yes |
| Business Definition | Opening price for the trading period |
| Source Field | `o` |
| Example | `331.98` |
| DQ | Numeric |

---

## 5.8 `previous_close`

| Attribute | Definition |
|---|---|
| Field | `previous_close` |
| Data Type | DOUBLE / DECIMAL |
| Nullable | Yes |
| Business Definition | Previous trading-period closing price |
| Source Field | `pc` |
| Example | `333.26` |
| DQ | Numeric |

---

## 5.9 `event_timestamp`

| Attribute | Definition |
|---|---|
| Field | `event_timestamp` |
| Data Type | TIMESTAMP |
| Nullable | Yes |
| Business Definition | Timestamp associated with the source market event |
| Source Field | `t` |
| Transformation | Unix timestamp → TIMESTAMP |
| Example | `2026-...` |
| DQ | Valid timestamp |

---

# 6. Business Date

## `business_date`

| Attribute | Definition |
|---|---|
| Field | `business_date` |
| Data Type | DATE |
| Nullable | No |
| Business Definition | Business/trading date associated with the record |
| Source | Derived from event timestamp or source date |
| Transformation | Timestamp → DATE |
| Example | `2026-08-08` |
| Business Key | Part of key |
| DQ | Valid date |

The business date should not automatically be interpreted as the ingestion date.

---

# 7. Company Profile Dataset

Logical target:

```text
company_profile
```

The dataset contains company-level reference information.

---

## 7.1 `company_name`

| Attribute | Definition |
|---|---|
| Field | `company_name` |
| Data Type | STRING |
| Nullable | Yes |
| Business Definition | Registered/display company name |
| Source | Company profile API |
| Example | `Example Corporation` |
| DQ | String validation |

---

## 7.2 `country`

| Attribute | Definition |
|---|---|
| Field | `country` |
| Data Type | STRING |
| Nullable | Yes |
| Business Definition | Country associated with the company |
| Source | Company profile API |
| Example | `United States` |

---

## 7.3 `currency`

| Attribute | Definition |
|---|---|
| Field | `currency` |
| Data Type | STRING |
| Nullable | Yes |
| Business Definition | Currency associated with the security/company data |
| Source | Company profile API |
| Example | `USD` |

---

## 7.4 `exchange`

| Attribute | Definition |
|---|---|
| Field | `exchange` |
| Data Type | STRING |
| Nullable | Yes |
| Business Definition | Exchange on which the security is listed |
| Source | Company profile API |
| Example | `NASDAQ` |

---

## 7.5 `industry`

| Attribute | Definition |
|---|---|
| Field | `industry` |
| Data Type | STRING |
| Nullable | Yes |
| Business Definition | Industry/sector classification associated with the company |
| Source | Company profile / reference data |
| Example | `Technology` |

The exact classification depends on the source data.

---

## 7.6 `market_cap`

| Attribute | Definition |
|---|---|
| Field | `market_cap` |
| Data Type | DOUBLE / DECIMAL |
| Nullable | Yes |
| Business Definition | Market capitalization of the company/security |
| Source | Financial/company dataset |
| Example | `2500000000000` |
| DQ | Numeric and unit validation |

The unit must be documented according to the specific API response.

---

# 8. Financial Metrics

Financial metrics may be included when available from the selected source endpoint.

---

## 8.1 `pe_ratio`

| Attribute | Definition |
|---|---|
| Field | `pe_ratio` |
| Data Type | DOUBLE |
| Nullable | Yes |
| Business Definition | Price-to-earnings ratio |
| Source | Financial metrics dataset |
| DQ | Numeric |

---

## 8.2 `eps`

| Attribute | Definition |
|---|---|
| Field | `eps` |
| Data Type | DOUBLE / DECIMAL |
| Nullable | Yes |
| Business Definition | Earnings per share |
| Source | Financial metrics dataset |
| DQ | Numeric |

The currency and reporting period should be considered when interpreting EPS.

---

## 8.3 `dividend_yield`

| Attribute | Definition |
|---|---|
| Field | `dividend_yield` |
| Data Type | DOUBLE |
| Nullable | Yes |
| Business Definition | Dividend yield associated with the security |
| Source | Financial metrics dataset |
| DQ | Numeric |
| Unit | Must be explicitly standardized as percentage or decimal |

---

## 8.4 `week_52_high`

| Attribute | Definition |
|---|---|
| Field | `week_52_high` |
| Data Type | DOUBLE / DECIMAL |
| Nullable | Yes |
| Business Definition | Highest reported security price during the previous 52-week period |
| Source | Financial metrics dataset |
| DQ | Numeric |

---

## 8.5 `week_52_low`

| Attribute | Definition |
|---|---|
| Field | `week_52_low` |
| Data Type | DOUBLE / DECIMAL |
| Nullable | Yes |
| Business Definition | Lowest reported security price during the previous 52-week period |
| Source | Financial metrics dataset |
| DQ | Numeric |

---

# 9. Trading Volume

## `trading_volume`

| Attribute | Definition |
|---|---|
| Field | `trading_volume` |
| Data Type | LONG |
| Nullable | Yes |
| Business Definition | Number of shares/units traded during the relevant trading period |
| Source | Market data endpoint |
| DQ | Numeric and non-negative |

Trading volume is used by the Most Active Stocks KPI.

---

# 10. Ranking Fields

## `rank`

| Attribute | Definition |
|---|---|
| Field | `rank` |
| Data Type | INTEGER |
| Nullable | No |
| Business Definition | Relative position of a security within a ranked KPI dataset |
| Source | Derived |
| Transformation | Ranking function |
| Example | `1` |

Examples:

```text
Top Gainers
Top Losers
Most Active Stocks
```

---

# 11. Operational Metadata

Operational fields are generated by FinPulse.

---

## 11.1 `run_id`

| Attribute | Definition |
|---|---|
| Field | `run_id` |
| Data Type | STRING |
| Nullable | No |
| Business Definition | Unique identifier for a pipeline execution |
| Source | Generated |
| Example | `RUN-20260808-001` |
| Purpose | Audit and lineage |

---

## 11.2 `ingestion_timestamp`

| Attribute | Definition |
|---|---|
| Field | `ingestion_timestamp` |
| Data Type | TIMESTAMP |
| Nullable | No |
| Business Definition | Timestamp when data was ingested into FinPulse |
| Source | Generated |
| Purpose | Operational tracking |

---

## 11.3 `ingestion_date`

| Attribute | Definition |
|---|---|
| Field | `ingestion_date` |
| Data Type | DATE |
| Nullable | No |
| Business Definition | Calendar date on which ingestion occurred |
| Source | Derived |
| Purpose | Operational tracking and partitioning |

---

## 11.4 `processing_timestamp`

| Attribute | Definition |
|---|---|
| Field | `processing_timestamp` |
| Data Type | TIMESTAMP |
| Nullable | Yes |
| Business Definition | Time at which a processing stage handled the record |
| Source | Generated |
| Purpose | Processing lineage |

---

## 11.5 `load_timestamp`

| Attribute | Definition |
|---|---|
| Field | `load_timestamp` |
| Data Type | TIMESTAMP |
| Nullable | No |
| Business Definition | Timestamp at which a target dataset was loaded |
| Source | Generated |
| Purpose | Target audit |

---

# 12. Metadata Framework Fields

Logical metadata dataset:

```text
dataset_metadata
```

---

## 12.1 `metadata_id`

Unique identifier for a metadata configuration record.

```text
Type: STRING
Nullable: No
```

---

## 12.2 `dataset_name`

Name of the dataset controlled by the metadata framework.

```text
Type: STRING
Nullable: No
Example: stock_quote
```

---

## 12.3 `source_system`

Identifies the upstream source.

```text
Type: STRING
Example: Finnhub
```

---

## 12.4 `source_endpoint`

Identifies the API endpoint associated with the dataset.

```text
Type: STRING
Nullable: Yes
```

---

## 12.5 `target_layer`

Identifies the processing layer.

Possible values:

```text
NI
SADP
AADP
CADP
```

---

## 12.6 `target_table`

Logical target dataset/table.

```text
Type: STRING
Nullable: No
```

---

## 12.7 `load_type`

Defines how the dataset is processed.

Possible values:

```text
FULL
INCREMENTAL
REPROCESS
BACKFILL
```

---

## 12.8 `watermark_column`

Identifies the field used for incremental processing.

```text
Type: STRING
Nullable: Yes
```

---

## 12.9 `processing_frequency`

Defines expected execution frequency.

Examples:

```text
DAILY
MONTHLY
ON_DEMAND
```

---

## 12.10 `active_flag`

Controls whether a metadata configuration is active.

```text
Type: BOOLEAN
Values: TRUE / FALSE
```

---

# 13. Watermark Dataset

Logical dataset:

```text
watermark_control
```

Recommended fields:

| Field | Type | Definition |
|---|---|---|
| `dataset_name` | STRING | Dataset identifier |
| `pipeline_name` | STRING | Processing pipeline |
| `watermark_value` | STRING/TIMESTAMP | Last successful processing boundary |
| `last_successful_run` | TIMESTAMP | Last successful run |
| `status` | STRING | Current status |
| `updated_timestamp` | TIMESTAMP | Last update time |

---

# 14. Audit Dataset

Logical dataset:

```text
pipeline_audit
```

Recommended fields:

| Field | Type | Definition |
|---|---|---|
| `run_id` | STRING | Execution identifier |
| `pipeline_name` | STRING | Pipeline |
| `dataset_name` | STRING | Dataset |
| `layer_name` | STRING | Processing layer |
| `start_timestamp` | TIMESTAMP | Start time |
| `end_timestamp` | TIMESTAMP | End time |
| `status` | STRING | Execution status |
| `records_read` | LONG | Records read |
| `records_written` | LONG | Records written |
| `error_code` | STRING | Error identifier |
| `error_message` | STRING | Error details |
| `processing_duration` | LONG | Processing duration |
| `created_timestamp` | TIMESTAMP | Audit creation time |

---

# 15. Data Quality Dataset

Logical dataset:

```text
data_quality_results
```

Recommended fields:

| Field | Type | Definition |
|---|---|---|
| `dq_run_id` | STRING | DQ execution ID |
| `dataset_name` | STRING | Dataset tested |
| `rule_id` | STRING | DQ rule identifier |
| `rule_name` | STRING | DQ rule |
| `records_checked` | LONG | Records evaluated |
| `records_passed` | LONG | Passing records |
| `records_failed` | LONG | Failed records |
| `dq_status` | STRING | PASS/FAIL/WARNING |
| `failure_reason` | STRING | Failure explanation |
| `execution_timestamp` | TIMESTAMP | Execution time |

---

# 16. Error Log Dataset

Logical dataset:

```text
error_log
```

Recommended fields:

| Field | Type | Definition |
|---|---|---|
| `run_id` | STRING | Pipeline run |
| `pipeline_name` | STRING | Pipeline |
| `dataset_name` | STRING | Dataset |
| `layer` | STRING | Processing layer |
| `error_type` | STRING | Error category |
| `error_message` | STRING | Error description |
| `retry_count` | INTEGER | Number of retries |
| `error_timestamp` | TIMESTAMP | Error time |

---

# 17. CADP Daily Stock Summary Dictionary

Logical dataset:

```text
cadp_daily_stock_summary
```

| Field | Type | Definition |
|---|---|---|
| `symbol` | STRING | Security identifier |
| `business_date` | DATE | Trading/business date |
| `opening_price` | DOUBLE/DECIMAL | Opening price |
| `closing_price` | DOUBLE/DECIMAL | Closing/latest price used by the dataset |
| `day_high` | DOUBLE/DECIMAL | Daily high |
| `day_low` | DOUBLE/DECIMAL | Daily low |
| `previous_close` | DOUBLE/DECIMAL | Previous close |
| `price_change` | DOUBLE/DECIMAL | Absolute price change |
| `price_change_percent` | DOUBLE | Percentage change |
| `trading_volume` | LONG | Trading volume where available |
| `load_timestamp` | TIMESTAMP | Target load timestamp |

---

# 18. CADP Top Gainers Dictionary

Logical dataset:

```text
cadp_top_gainers
```

| Field | Type | Definition |
|---|---|---|
| `symbol` | STRING | Security |
| `business_date` | DATE | Business date |
| `closing_price` | DOUBLE/DECIMAL | Closing/latest price |
| `price_change` | DOUBLE/DECIMAL | Absolute change |
| `price_change_percent` | DOUBLE | Percentage change |
| `rank` | INTEGER | Gainer rank |
| `load_timestamp` | TIMESTAMP | Load time |

---

# 19. CADP Top Losers Dictionary

Logical dataset:

```text
cadp_top_losers
```

| Field | Type | Definition |
|---|---|---|
| `symbol` | STRING | Security |
| `business_date` | DATE | Business date |
| `closing_price` | DOUBLE/DECIMAL | Closing/latest price |
| `price_change` | DOUBLE/DECIMAL | Absolute change |
| `price_change_percent` | DOUBLE | Percentage change |
| `rank` | INTEGER | Loser rank |
| `load_timestamp` | TIMESTAMP | Load time |

---

# 20. CADP Most Active Dictionary

Logical dataset:

```text
cadp_most_active
```

| Field | Type | Definition |
|---|---|---|
| `symbol` | STRING | Security |
| `business_date` | DATE | Business date |
| `trading_volume` | LONG | Trading volume |
| `rank` | INTEGER | Activity rank |
| `load_timestamp` | TIMESTAMP | Load timestamp |

---

# 21. CADP Company Performance Dictionary

Logical dataset:

```text
cadp_company_performance
```

| Field | Type | Definition |
|---|---|---|
| `symbol` | STRING | Security |
| `company_name` | STRING | Company name |
| `industry` | STRING | Industry classification |
| `market_cap` | DOUBLE/DECIMAL | Market capitalization |
| `current_price` | DOUBLE/DECIMAL | Latest price |
| `price_change` | DOUBLE/DECIMAL | Price movement |
| `price_change_percent` | DOUBLE | Percentage movement |
| `business_date` | DATE | Business date |
| `load_timestamp` | TIMESTAMP | Load timestamp |

Optional financial metrics:

```text
pe_ratio
eps
dividend_yield
week_52_high
week_52_low
```

---

# 22. Sector Performance Dictionary

Logical dataset:

```text
cadp_sector_performance
```

| Field | Type | Definition |
|---|---|---|
| `sector` | STRING | Sector/industry |
| `business_date` | DATE | Business date |
| `security_count` | LONG | Number of securities |
| `average_price_change_percent` | DOUBLE | Average price movement |
| `total_trading_volume` | LONG | Total volume |
| `total_market_cap` | DOUBLE/DECIMAL | Aggregate market cap where applicable |
| `load_timestamp` | TIMESTAMP | Load timestamp |

---

# 23. Historical Trend Dictionary

Logical dataset:

```text
cadp_historical_trends
```

| Field | Type | Definition |
|---|---|---|
| `symbol` | STRING | Security |
| `business_date` | DATE | Historical trading date |
| `opening_price` | DOUBLE/DECIMAL | Opening price |
| `day_high` | DOUBLE/DECIMAL | Daily high |
| `day_low` | DOUBLE/DECIMAL | Daily low |
| `closing_price` | DOUBLE/DECIMAL | Closing price |
| `trading_volume` | LONG | Trading volume |
| `load_timestamp` | TIMESTAMP | Load timestamp |

---

# 24. Field Classification

FinPulse fields can be classified into four categories.

## Source Fields

Directly received from the source.

Examples:

```text
c
d
dp
h
l
o
pc
t
```

## Standardized Fields

Renamed and standardized source attributes.

Examples:

```text
current_price
price_change
price_change_percent
day_high
day_low
```

## Derived Fields

Calculated by the platform.

Examples:

```text
price_change
rank
business_date
average_price_change_percent
```

## Operational Fields

Generated by the platform.

Examples:

```text
run_id
ingestion_timestamp
processing_timestamp
load_timestamp
```

---

# 25. Data Sensitivity Classification

The FinPulse datasets primarily contain market and company information rather than customer PII.

Suggested classification:

| Data Type | Classification |
|---|---|
| Stock prices | Business/Public |
| Trading volume | Business/Public |
| Company profile | Business/Public |
| Market capitalization | Business/Public |
| Financial metrics | Business/Public |
| API credentials | Confidential |
| Pipeline configuration | Internal |
| Audit logs | Internal |
| Error logs | Internal |

Secrets must never be stored in analytical datasets.

---

# 26. Data Retention Considerations

Retention should be defined according to the project's operational requirements.

Logical principles:

```text
NI
↓
Longer retention for reprocessing/lineage

SADP
↓
Retention based on analytical needs

AADP
↓
Retention based on business requirements

CADP
↓
Retention based on reporting requirements
```

The actual retention period should be defined separately if the project moves toward production.

---

# 27. Data Dictionary Governance

The Data Dictionary should be updated whenever:

- A new source field is introduced.
- A target field is added.
- A field is removed.
- A datatype changes.
- A business definition changes.
- A KPI is introduced.
- A transformation rule changes.
- A source endpoint changes.

The dictionary should remain synchronized with the STM and implementation.

---

# 28. Relationship With STM

The STM answers:

> **How does the field move?**

The Data Dictionary answers:

> **What does the field mean?**

Example:

```text
Finnhub
  │
  │ c
  ▼
NI
  │
  │ rename
  ▼
SADP
  │
  │ current_price
  ▼
AADP
  │
  ▼
CADP
```

The Data Dictionary defines:

```text
current_price
=
Latest/current security price
```

---

# 29. Relationship With DQ Framework

The Data Dictionary provides the foundation for DQ rules.

Example:

```text
Field:
current_price

Definition:
Latest security price

Type:
DOUBLE

Nullable:
No

DQ:
Numeric
Non-negative where applicable
```

This allows DQ rules to be derived systematically from field definitions.

---

# 30. Data Dictionary Acceptance Criteria

The document is considered complete when:

1. Every important analytical field has a definition.
2. Data types are documented.
3. Nullable behavior is documented.
4. Source lineage is documented.
5. Business meaning is documented.
6. Derived fields have formulas or transformation rules.
7. Operational fields are documented.
8. Business keys are identified.
9. DQ expectations are documented.
10. Dataset-level dictionaries exist for major CADP outputs.
11. Optional source fields are clearly identified.
12. The dictionary matches the STM and implementation.

---

# 31. Document Version History

| Version | Date | Author | Description |
|---|---|---|---|
| 1.0 | 2026-08-08 | Aniruddha Giri | Initial Data Dictionary |

---

# 32. Conclusion

The FinPulse Data Dictionary establishes a common technical and business vocabulary for the platform.

It defines the meaning and characteristics of data as it moves through:

```text
NI
 ↓
SADP
 ↓
AADP
 ↓
CADP
```

The dictionary provides the foundation for:

- Data quality
- Data lineage
- Source-to-target mapping
- Analytical development
- KPI generation
- Troubleshooting
- Future dashboard development
- Future machine-learning workloads

Together with the STM and LLD, this document provides a field-level reference for implementing and maintaining FinPulse.

**End of Data Dictionary**