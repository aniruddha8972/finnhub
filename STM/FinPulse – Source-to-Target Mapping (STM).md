# FinPulse – Source-to-Target Mapping (STM)

**Project Name:** FinPulse – Enterprise Financial Market Data Platform  
**Document:** Source-to-Target Mapping  
**Version:** 1.0  
**Status:** Draft  
**Author:** Aniruddha Giri  
**Role:** Data Engineer  
**Platform:** Databricks Free Edition  

---

# 1. Document Purpose

This Source-to-Target Mapping document defines how data moves from the external Finnhub source through the FinPulse processing layers.

The document provides:

- Source fields
- Source data types
- Target fields
- Target data types
- Transformation logic
- Business rules
- Data-quality requirements
- Nullable behavior
- Processing layer
- Target datasets

The mapping provides the technical bridge between the source API and the final analytical datasets.

---

# 2. Mapping Scope

The mapping covers:

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
```

The primary datasets currently considered are:

1. Stock Quote
2. Company Profile
3. Daily Stock Summary
4. Top Gainers
5. Top Losers
6. Most Active Stocks
7. Company Performance
8. Sector Performance
9. Historical Trends

---

# 3. Mapping Conventions

| Term | Meaning |
|---|---|
| Source | Upstream API field |
| NI | Native/raw ingestion layer |
| SADP | Standardized data layer |
| AADP | Analytical transformation layer |
| CADP | Consumption/analytics layer |
| Transformation | Logic applied to source data |
| Business Rule | Functional rule applied to the data |
| DQ | Data-quality validation |

---

# 4. Source Dataset – Stock Quote

The stock quote endpoint returns quote-level market information.

Example source response:

```json
{
  "c": 333.74,
  "d": 0.48,
  "dp": 0.144,
  "h": 334.99,
  "l": 329.0006,
  "o": 331.98,
  "pc": 333.26,
  "t": 1784318400
}
```

The source response does not necessarily contain the security symbol inside the response itself. The symbol should therefore be captured from the request context when required.

---

# 5. Stock Quote – Source-to-NI Mapping

NI is intended to preserve source information with minimal transformation.

| Source Field | Source Type | NI Field | NI Type | Transformation |
|---|---|---|---|---|
| `c` | Numeric | `c` | DOUBLE | None |
| `d` | Numeric | `d` | DOUBLE | None |
| `dp` | Numeric | `dp` | DOUBLE | None |
| `h` | Numeric | `h` | DOUBLE | None |
| `l` | Numeric | `l` | DOUBLE | None |
| `o` | Numeric | `o` | DOUBLE | None |
| `pc` | Numeric | `pc` | DOUBLE | None |
| `t` | Integer | `t` | LONG | None |
| Request Symbol | STRING | `symbol` | STRING | Captured from request |
| System generated | — | `run_id` | STRING | Generated |
| System generated | — | `ingestion_timestamp` | TIMESTAMP | Generated |
| System generated | — | `ingestion_date` | DATE | Derived |

---

# 6. Stock Quote – NI Design

The NI layer retains the source field names to maintain source lineage.

Example:

```text
symbol
c
d
dp
h
l
o
pc
t
run_id
ingestion_timestamp
ingestion_date
```

The source payload should remain recoverable from this layer.

---

# 7. Stock Quote – NI to SADP Mapping

The SADP layer converts source-specific field names into business-friendly names.

| NI Field | SADP Field | Target Type | Transformation |
|---|---|---|---|
| `symbol` | `symbol` | STRING | Direct |
| `c` | `current_price` | DOUBLE | Rename |
| `d` | `price_change` | DOUBLE | Rename |
| `dp` | `price_change_percent` | DOUBLE | Rename |
| `h` | `day_high` | DOUBLE | Rename |
| `l` | `day_low` | DOUBLE | Rename |
| `o` | `open_price` | DOUBLE | Rename |
| `pc` | `previous_close` | DOUBLE | Rename |
| `t` | `event_timestamp` | TIMESTAMP | Unix timestamp conversion |
| `ingestion_timestamp` | `ingestion_timestamp` | TIMESTAMP | Direct |
| `ingestion_date` | `ingestion_date` | DATE | Direct |
| `run_id` | `run_id` | STRING | Direct |

---

# 8. Timestamp Transformation

The source field `t` represents an epoch-based timestamp.

Transformation:

```text
t
│
▼
Unix Timestamp
│
▼
Spark Timestamp
│
▼
event_timestamp
```

The implementation should use the appropriate Spark timestamp conversion function.

The resulting field should use a consistent timezone convention defined by the platform.

---

# 9. Stock Quote – SADP Data Quality

| Field | DQ Rule |
|---|---|
| `symbol` | Must not be NULL |
| `current_price` | Must be valid numeric value |
| `previous_close` | Must be valid numeric value where required |
| `event_timestamp` | Must be valid timestamp |
| `price_change_percent` | Must be numeric |
| Business key | Must not be duplicated |

Invalid records should be handled according to the project's DQ strategy.

---

# 10. Stock Quote – SADP to AADP

The AADP layer primarily prepares analytical attributes.

| SADP Field | AADP Field | Transformation |
|---|---|---|
| `symbol` | `symbol` | Direct |
| `current_price` | `current_price` | Direct |
| `open_price` | `open_price` | Direct |
| `day_high` | `day_high` | Direct |
| `day_low` | `day_low` | Direct |
| `previous_close` | `previous_close` | Direct |
| `price_change` | `price_change` | Direct |
| `price_change_percent` | `price_change_percent` | Direct |
| `event_timestamp` | `event_timestamp` | Direct |
| `event_timestamp` | `business_date` | Date extraction |

---

# 11. AADP Derived Fields

Where required, derived metrics may be generated.

## Price Change

```text
price_change =
current_price - previous_close
```

## Price Change Percentage

```text
price_change_percent =
((current_price - previous_close) / previous_close) × 100
```

If the source already provides `d` and `dp`, those source values can be retained and validated against the calculated values rather than unnecessarily replacing them.

This provides an additional data-quality validation opportunity.

---

# 12. Source-vs-Calculated Validation

For quote data:

```text
Source dp
   │
   ├──────────────┐
   │              │
   ▼              ▼
Source Value   Calculated Value
   │              │
   └──────┬───────┘
          ▼
       Compare
          │
     ┌────┴────┐
     ▼         ▼
   Match    Difference
     │         │
     ▼         ▼
   PASS       DQ Flag
```

A tolerance may be defined because of rounding differences.

---

# 13. AADP to CADP – Daily Stock Summary

| AADP Field | CADP Field | Transformation |
|---|---|---|
| `symbol` | `symbol` | Direct |
| `business_date` | `business_date` | Direct |
| `open_price` | `opening_price` | Rename |
| `current_price` | `closing_price` | Rename according to dataset definition |
| `day_high` | `day_high` | Direct |
| `day_low` | `day_low` | Direct |
| `previous_close` | `previous_close` | Direct |
| `price_change` | `price_change` | Direct |
| `price_change_percent` | `price_change_percent` | Direct |
| `ingestion_timestamp` | `load_timestamp` | Rename |

---

# 14. Daily Stock Summary – Business Rule

The Daily Stock Summary represents the market information available for a security for a defined business date.

The dataset should have a logical uniqueness constraint such as:

```text
symbol + business_date
```

The exact key should be validated based on the frequency and behavior of the underlying source.

---

# 15. Top Gainers Mapping

Top Gainers is a derived CADP dataset.

Input:

```text
AADP Stock Data
```

Transformation:

```text
Filter valid records
        │
        ▼
price_change_percent > 0
        │
        ▼
ORDER BY price_change_percent DESC
        │
        ▼
Assign Rank
        │
        ▼
Select Top-N
```

Potential output:

| Field | Source | Transformation |
|---|---|---|
| `symbol` | AADP | Direct |
| `business_date` | AADP | Direct |
| `closing_price` | AADP | Direct |
| `price_change` | AADP | Direct |
| `price_change_percent` | AADP | Direct |
| `rank` | Derived | Ranking function |

---

# 16. Top Losers Mapping

Transformation:

```text
Filter valid records
        │
        ▼
price_change_percent < 0
        │
        ▼
ORDER BY price_change_percent ASC
        │
        ▼
Assign Rank
        │
        ▼
Select Top-N
```

Potential output:

| Field | Source | Transformation |
|---|---|---|
| `symbol` | AADP | Direct |
| `business_date` | AADP | Direct |
| `closing_price` | AADP | Direct |
| `price_change` | AADP | Direct |
| `price_change_percent` | AADP | Direct |
| `rank` | Derived | Ranking function |

---

# 17. Most Active Stocks Mapping

Most Active Stocks is based on trading volume.

Transformation:

```text
AADP Stock Data
      │
      ▼
Valid Volume
      │
      ▼
ORDER BY trading_volume DESC
      │
      ▼
Assign Rank
      │
      ▼
Select Top-N
```

Potential output:

| Field | Source | Transformation |
|---|---|---|
| `symbol` | AADP | Direct |
| `business_date` | AADP | Direct |
| `trading_volume` | AADP | Direct |
| `rank` | Derived | Ranking function |

---

# 18. Company Profile Source

The company profile dataset provides company-level attributes.

The exact response structure depends on the Finnhub endpoint and subscription level.

Potential source attributes include:

```text
symbol
name
country
currency
exchange
industry
marketCapitalization
```

Only fields actually returned by the selected API endpoint should be implemented.

---

# 19. Company Profile – Source to NI

| Source Attribute | NI Attribute | Transformation |
|---|---|---|
| Symbol | `symbol` | Direct |
| Company Name | `company_name` | Minimal rename |
| Country | `country` | Direct |
| Currency | `currency` | Direct |
| Exchange | `exchange` | Direct |
| Industry | `industry` | Direct |
| Market Capitalization | `market_cap` | Type normalization |
| System metadata | `run_id` | Generated |
| System metadata | `ingestion_timestamp` | Generated |

The exact source field names should be finalized against the actual API response.

---

# 20. Company Profile – NI to SADP

| NI Field | SADP Field | Transformation |
|---|---|---|
| `symbol` | `symbol` | Direct |
| `company_name` | `company_name` | Standardize |
| `country` | `country` | Standardize |
| `currency` | `currency` | Standardize |
| `exchange` | `exchange` | Standardize |
| `industry` | `industry` | Standardize |
| `market_cap` | `market_cap` | Numeric conversion |

---

# 21. Company Profile – SADP to AADP

AADP may enrich company information with market data.

Conceptual join:

```text
Company Profile
       │
       │ symbol
       ▼
Stock Market Data
       │
       ▼
Company Performance Dataset
```

Join key:

```text
symbol
```

The join should be validated for uniqueness on the relevant side(s).

---

# 22. Company Performance Mapping

Potential CADP attributes:

| CADP Field | Source | Transformation |
|---|---|---|
| `symbol` | AADP | Direct |
| `company_name` | Company AADP | Direct |
| `industry` | Company AADP | Direct |
| `market_cap` | Company AADP | Direct |
| `current_price` | Stock AADP | Direct |
| `price_change` | Stock AADP | Direct |
| `price_change_percent` | Stock AADP | Direct |
| `business_date` | Stock AADP | Direct |

---

# 23. Sector Performance Mapping

Sector Performance is a derived dataset.

Source:

```text
Company / Industry Data
          +
Stock Performance Data
```

Processing:

```text
Join on Symbol
      │
      ▼
Map Security → Sector/Industry
      │
      ▼
Group by Sector
      │
      ▼
Calculate Aggregates
```

Potential output:

| Field | Transformation |
|---|---|
| `sector` | Source classification |
| `business_date` | Date extraction |
| `security_count` | COUNT |
| `average_price_change_percent` | AVG |
| `total_trading_volume` | SUM where available |
| `total_market_cap` | SUM where appropriate |

---

# 24. Historical Trend Mapping

Historical Trend datasets require historical/time-series data.

Potential source fields:

```text
symbol
date
open
high
low
close
volume
```

Mapping:

| Source | Target | Transformation |
|---|---|---|
| `symbol` | `symbol` | Direct |
| `date` | `business_date` | Date conversion |
| `open` | `opening_price` | Rename |
| `high` | `day_high` | Rename |
| `low` | `day_low` | Rename |
| `close` | `closing_price` | Rename |
| `volume` | `trading_volume` | Type conversion |

This mapping applies only if the historical endpoint supplies these attributes.

---

# 25. Market Capitalization Mapping

Market capitalization should be standardized to a consistent numeric representation.

Conceptual mapping:

```text
Source Market Capitalization
          │
          ▼
Numeric Conversion
          │
          ▼
Standard Market Cap
```

The unit of measure must be explicitly documented.

For example, if the API returns market capitalization in millions, the unit must not be interpreted as absolute currency without conversion.

---

# 26. Financial Metrics Mapping

Where the source provides financial metrics such as:

- P/E ratio
- EPS
- Dividend yield
- 52-week high
- 52-week low

the mapping should follow:

```text
Source Financial Metric
        │
        ▼
SADP Standardization
        │
        ▼
AADP Validation / Transformation
        │
        ▼
CADP Company Performance
```

Example:

| Source | Target | Rule |
|---|---|---|
| P/E | `pe_ratio` | Numeric conversion |
| EPS | `eps` | Numeric conversion |
| Dividend Yield | `dividend_yield` | Standardize unit |
| 52W High | `week_52_high` | Numeric |
| 52W Low | `week_52_low` | Numeric |

These fields are optional and depend on the selected Finnhub endpoint and available data.

---

# 27. Data Type Mapping Standards

Recommended standards:

| Business Type | Spark Type |
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

For financial values requiring strict precision, DECIMAL may be preferred over DOUBLE.

---

# 28. Nullability Rules

### Mandatory

Examples:

```text
symbol
business_date
```

These should generally be non-null.

### Conditional

Examples:

```text
market_cap
pe_ratio
eps
dividend_yield
```

These may be nullable because availability depends on the source dataset.

---

# 29. Data Quality Mapping

Each important field should have an associated DQ rule.

Example:

| Field | DQ Rule | Severity |
|---|---|---|
| `symbol` | Not null | Critical |
| `current_price` | Numeric | Critical |
| `previous_close` | Numeric where available | High |
| `price_change_percent` | Numeric/range validation | High |
| `business_date` | Valid date | Critical |
| `market_cap` | Non-negative where applicable | Medium |
| `trading_volume` | Non-negative | High |

Severity should determine whether the pipeline fails, warns, or continues.

---

# 30. Business Key Mapping

Potential business keys:

### Stock Quote

```text
symbol + business_date
```

### Company Profile

```text
symbol
```

### Historical Data

```text
symbol + business_date
```

### Top Gainers

```text
symbol + business_date
```

### Top Losers

```text
symbol + business_date
```

### Most Active

```text
symbol + business_date
```

The final key should reflect the actual processing frequency and dataset semantics.

---

# 31. Incremental Mapping

Incremental processing is controlled through a watermark.

Logical flow:

```text
Last Watermark
      │
      ▼
Source Extraction
      │
      ▼
Filter New/Changed Data
      │
      ▼
Process
      │
      ▼
Target
      │
      ▼
Successful?
   /       \
 Yes       No
  │         │
  ▼         ▼
Update     Retain
Watermark  Watermark
```

---

# 32. Source-to-Target Lineage

A representative quote lineage is:

```text
Finnhub
  │
  ├── c ──────► current_price
  │
  ├── d ──────► price_change
  │
  ├── dp ─────► price_change_percent
  │
  ├── h ──────► day_high
  │
  ├── l ──────► day_low
  │
  ├── o ──────► open_price
  │
  ├── pc ─────► previous_close
  │
  └── t ──────► event_timestamp
                       │
                       ▼
                AADP Metrics
                       │
                       ▼
                  CADP KPIs
```

---

# 33. Example End-to-End Mapping

A single quote record follows:

```text
Source
 c = 333.74
        │
        ▼
NI
 c = 333.74
        │
        ▼
SADP
 current_price = 333.74
        │
        ▼
AADP
 current_price = 333.74
 price_change = 0.48
 price_change_percent = 0.144
        │
        ▼
CADP
 Daily Stock Summary
```

---

# 34. Source Field `c`

| Attribute | Value |
|---|---|
| Source Field | `c` |
| Meaning | Current/latest price |
| Source Type | Numeric |
| NI Field | `c` |
| SADP Field | `current_price` |
| AADP Field | `current_price` |
| CADP Field | `closing_price` where dataset semantics support it |
| Transformation | Rename/type standardization |
| DQ | Numeric, non-negative where applicable |

---

# 35. Source Field `d`

| Attribute | Value |
|---|---|
| Source Field | `d` |
| Meaning | Price change |
| Source Type | Numeric |
| NI Field | `d` |
| SADP Field | `price_change` |
| AADP Field | `price_change` |
| CADP Field | `price_change` |
| Transformation | Rename |
| DQ | Numeric |

---

# 36. Source Field `dp`

| Attribute | Value |
|---|---|
| Source Field | `dp` |
| Meaning | Percentage price change |
| Source Type | Numeric |
| NI Field | `dp` |
| SADP Field | `price_change_percent` |
| AADP Field | `price_change_percent` |
| CADP Field | `price_change_percent` |
| Transformation | Rename |
| DQ | Numeric and consistency check |

---

# 37. Source Field `h`

| Attribute | Value |
|---|---|
| Source Field | `h` |
| Meaning | Daily high |
| Source Type | Numeric |
| NI Field | `h` |
| SADP Field | `day_high` |
| AADP Field | `day_high` |
| CADP Field | `day_high` |
| Transformation | Rename |
| DQ | Numeric |

---

# 38. Source Field `l`

| Attribute | Value |
|---|---|
| Source Field | `l` |
| Meaning | Daily low |
| Source Type | Numeric |
| NI Field | `l` |
| SADP Field | `day_low` |
| AADP Field | `day_low` |
| CADP Field | `day_low` |
| Transformation | Rename |
| DQ | Numeric |

---

# 39. Source Field `o`

| Attribute | Value |
|---|---|
| Source Field | `o` |
| Meaning | Opening price |
| Source Type | Numeric |
| NI Field | `o` |
| SADP Field | `open_price` |
| AADP Field | `open_price` |
| CADP Field | `opening_price` |
| Transformation | Rename |
| DQ | Numeric |

---

# 40. Source Field `pc`

| Attribute | Value |
|---|---|
| Source Field | `pc` |
| Meaning | Previous close |
| Source Type | Numeric |
| NI Field | `pc` |
| SADP Field | `previous_close` |
| AADP Field | `previous_close` |
| CADP Field | `previous_close` |
| Transformation | Rename |
| DQ | Numeric |

---

# 41. Source Field `t`

| Attribute | Value |
|---|---|
| Source Field | `t` |
| Meaning | Source event timestamp |
| Source Type | Integer |
| NI Field | `t` |
| SADP Field | `event_timestamp` |
| AADP Field | `event_timestamp` |
| CADP Field | `business_date` derived |
| Transformation | Unix timestamp → timestamp → date |
| DQ | Valid timestamp |

---

# 42. Metadata Field Mapping

Operational fields are generated by FinPulse rather than sourced from Finnhub.

| Generated Field | Purpose |
|---|---|
| `run_id` | Execution tracking |
| `ingestion_timestamp` | Source ingestion time |
| `ingestion_date` | Partition/date tracking |
| `processing_timestamp` | Transformation tracking |
| `load_timestamp` | Target load time |

These fields support lineage, auditing, troubleshooting, and operational monitoring.

---

# 43. Rejection Handling

Records failing critical DQ rules should not silently disappear.

Logical flow:

```text
Source Record
      │
      ▼
Validation
      │
 ┌────┴─────┐
 ▼          ▼
PASS       FAIL
 │           │
 ▼           ▼
Target     Rejected/
           Quarantine
```

Rejected records should be logged with sufficient information to identify the failure reason.

---

# 44. Transformation Classification

Each mapping should fall into one of the following categories:

### Direct

```text
source → target
```

### Rename

```text
c → current_price
```

### Type Conversion

```text
integer → timestamp
```

### Derived

```text
current_price - previous_close
```

### Aggregation

```text
AVG(price_change_percent)
```

### Ranking

```text
ORDER BY price_change_percent DESC
```

### Join

```text
stock.symbol = company.symbol
```

---

# 45. Mapping Governance

Changes to source or target structures should update the STM.

Examples:

- New source field
- Removed source field
- Data type change
- Target column rename
- New business rule
- New KPI
- New dataset
- API response structure change

The STM should remain synchronized with the implementation.

---

# 46. Mapping Validation

The implementation should validate:

```text
Source Field
     │
     ▼
Expected Mapping
     │
     ▼
Target Field
     │
     ▼
Expected Data Type
     │
     ▼
DQ Validation
```

Any mismatch should be recorded as a processing issue.

---

# 47. Mapping Dependencies

Some target datasets depend on multiple source datasets.

Example:

```text
Finnhub Stock Quote
        │
        ▼
      SADP
        │
        ▼
      AADP
        │
        ├──────────────┐
        │              │
        ▼              ▼
Company Profile    Market Data
        │              │
        └──────┬───────┘
               ▼
       Company Performance
```

---

# 48. Final Source-to-Target Flow

```text
                  Finnhub API
                       │
                       ▼
                Raw API Fields
                       │
                       ▼
                       NI
                       │
                       ▼
                Standardization
                       │
                       ▼
                      SADP
                       │
                       ▼
              Business Transformations
                       │
                       ▼
                      AADP
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      Daily        Gainers       Losers
      Summary
          │            │            │
          └────────────┼────────────┘
                       ▼
                      CADP
                       │
                       ▼
                Databricks SQL
                       │
                       ▼
              Investment Analysts
```

---

# 49. STM Acceptance Criteria

The STM is considered complete when:

1. All implemented source datasets are identified.
2. Source fields are mapped to NI.
3. NI fields are mapped to SADP.
4. SADP fields are mapped to AADP.
5. AADP fields are mapped to CADP.
6. Transformations are documented.
7. Business rules are documented.
8. Data types are defined.
9. DQ rules are identified.
10. Business keys are defined.
11. Derived fields are documented.
12. Operational fields are documented.
13. Source-to-target lineage is traceable.
14. Optional source attributes are clearly identified.
15. The mapping remains synchronized with implementation.

---

# 50. Relationship With Other Documents

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
LLD
 │
 ▼
STM ◄── Current Document
 │
 ├── Data Dictionary
 │
 ├── Metadata Design
 │
 ├── Data Quality Framework
 │
 └── Logging & Audit Design
```

---

# 51. Document Version History

| Version | Date | Author | Description |
|---|---|---|---|
| 1.0 | 2026-08-08 | Aniruddha Giri | Initial Source-to-Target Mapping |

---

# 52. Conclusion

The FinPulse Source-to-Target Mapping establishes traceability from the external Finnhub API to the final business-ready analytical datasets.

The mapping follows:

```text
Source
  ↓
NI
  ↓
SADP
  ↓
AADP
  ↓
CADP
```

The primary stock quote fields are standardized from source-specific names such as `c`, `d`, `dp`, `h`, `l`, `o`, `pc`, and `t` into business-readable attributes such as `current_price`, `price_change`, `price_change_percent`, `day_high`, `day_low`, `open_price`, `previous_close`, and `event_timestamp`.

The STM also establishes the foundation for data lineage, DQ validation, schema management, and downstream KPI generation.

**End of Source-to-Target Mapping Document**