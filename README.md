# 📈 FinPulse – Enterprise Financial Market Data Platform

> An end-to-end Data Engineering project built on **Databricks Free Edition** using the **Finnhub API**. The platform ingests, transforms, and analyzes financial market data using a **Medallion Architecture (Bronze → Silver → Gold)** with enterprise-grade features such as metadata-driven processing, incremental loading, data quality validation, audit logging, and KPI generation.

---

## 📌 Project Overview

FinPulse is designed to simulate a real-world financial data platform used by investment firms and fintech companies.

The platform automates the ingestion of financial market data from the Finnhub API, processes it using Apache Spark, stores it in Delta Lake, and produces analytics-ready datasets for business reporting.

---

## 🎯 Business Problem

Financial organizations require timely, accurate, and scalable access to market data for analytics and decision-making.

Manual data collection is:
- Time-consuming
- Error-prone
- Difficult to scale
- Unsuitable for enterprise reporting

FinPulse addresses these challenges by automating the complete data pipeline.

---

# 🏗 Solution Architecture

```text
                 Finnhub API
                      │
                      ▼
         Bronze Ingestion Notebook
                      │
                      ▼
          Bronze Layer (Raw JSON)
                      │
                      ▼
     Silver Layer (Clean & Standardized)
                      │
                      ▼
      Gold Layer (Business KPIs)
                      │
                      ▼
        Databricks SQL Analytics
```

---

# 🏛 Medallion Architecture

## 🥉 Bronze Layer
- Raw API response
- JSON storage
- Immutable data
- Incremental ingestion

---

## 🥈 Silver Layer
- Flatten nested JSON
- Remove duplicates
- Data standardization
- Null handling
- Data type conversions

---

## 🥇 Gold Layer
Business-ready datasets including:

- Daily Stock Summary
- Top Gainers
- Top Losers
- Most Active Stocks
- Sector Performance
- Company Performance
- Market Capitalization
- Trading Volume
- Historical Trends

---

# ⚙️ Technology Stack

| Technology | Purpose |
|------------|----------|
| Databricks Free Edition | Data Processing |
| Apache Spark (PySpark) | Distributed Processing |
| Delta Lake | Data Storage |
| Python | API Integration |
| SQL | Analytics |
| Finnhub API | Financial Market Data |
| Git & GitHub | Version Control |

---

# 📂 Repository Structure

```text
FinPulse/
│
├── docs/
│   ├── BRD
│   ├── SRD
│   ├── ADD
│   ├── HLD
│   ├── LLD
│   ├── STM
│   ├── Data Dictionary
│   ├── Metadata Design
│   ├── Data Quality Framework
│   └── Logging & Audit Design
│
├── notebooks/
│   ├── bronze/
│   ├── silver/
│   ├── gold/
│   ├── common/
│   └── metadata/
│
├── sql/
│
├── configs/
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

# 📊 Business KPIs

## Market KPIs

- Daily Closing Price
- Daily Opening Price
- Daily High & Low
- Daily Trading Volume
- Daily Price Change %
- Top Gainers
- Top Losers
- Most Active Stocks

---

## Company KPIs

- Market Capitalization
- P/E Ratio
- EPS
- Dividend Yield
- 52 Week High
- 52 Week Low

---

# 🔄 Pipeline Workflow

```text
Read Pipeline Configuration
            │
            ▼
Read Watermark
            │
            ▼
Call Finnhub API
            │
            ▼
Store Raw JSON (Bronze)
            │
            ▼
Validate Data Quality
            │
            ▼
Transform to Silver
            │
            ▼
Generate Gold KPIs
            │
            ▼
Update Metadata
            │
            ▼
Write Audit Logs
            │
            ▼
Publish Analytics
```

---

# 📋 Enterprise Features

- Metadata-Driven Processing
- Incremental Data Loading
- Watermark Framework
- Data Quality Validation
- Audit Logging
- Error Handling
- Retry Mechanism
- Delta Lake Storage
- Medallion Architecture
- Modular Notebook Design

---

# 📑 Documentation

The project includes enterprise documentation:

- Business Requirements Document (BRD)
- Software Requirements Document (SRD)
- Architecture Design Document (ADD)
- High-Level Design (HLD)
- Low-Level Design (LLD)
- Source-to-Target Mapping (STM)
- Data Dictionary
- Metadata Design
- Data Quality Framework
- Logging & Audit Design
- Error Handling & Recovery Strategy

---

# 🚀 Future Enhancements

- Structured Streaming
- Databricks Workflows
- Machine Learning Price Forecasting
- Technical Indicators
- Market Sentiment Analysis
- Interactive Dashboards
- Multi-Source Financial Data Integration

---

# 👨‍💻 Author

**Aniruddha Giri**

**Data Engineer**

**Skills:** Databricks • PySpark • Python • SQL • Delta Lake • Data Engineering

---

## ⭐ If you found this project useful, consider giving it a star!
