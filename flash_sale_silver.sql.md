# 🥈 Silver Layer

## Overview

The Silver layer is responsible for transforming raw retail events into trusted, analytics-ready datasets. It applies data quality checks, validation rules, standardization, and deduplication to ensure that only clean and consistent records progress through the pipeline.

Building upon the Bronze layer, this stage eliminates incomplete and invalid records, normalizes categorical values, recalculates business metrics, and produces a structured dataset optimized for downstream aggregation and reporting.

By enforcing consistent data quality standards, the Silver layer provides a reliable foundation for KPI computation, dashboarding, and business intelligence.

---
## 🎯 Objectives

* Clean and validate raw retail events ingested from the Bronze layer.
* Remove duplicate records to ensure analytical accuracy.
* Standardize categorical fields for consistent reporting.
* Filter incomplete or invalid events before aggregation.
* Compute derived business metrics such as event revenue.
* Produce a trusted dataset for Gold layer KPI generation.

---
## 🔄 Silver Layer Workflow

Bronze Layer
(flash_sale_events)\
          ↓\
Data Validation\
          ↓\
Duplicate Removal\
          ↓\
Data Standardization
(Category & Event Type)\
          ↓\
Revenue Calculation\
          ↓\
Invalid Record Filtering\
          ↓\
Silver Layer
(flash_sale_silver)

---
## 🧹 Component — Silver Data Transformation

### Purpose

The Silver Data Transformation component refines raw retail events collected in the Bronze layer into a clean, standardized, and analytics-ready dataset. It performs validation, normalization, deduplication, and quality enforcement to ensure downstream business metrics are generated from reliable data.

This transformation stage removes incomplete or inconsistent records while preserving only high-quality events required for sales analytics, customer behavior analysis, and KPI computation.

---
## Key Responsibilities

* Reads raw retail events from the flash_sale_events Bronze table.
* Removes duplicate records using SELECT DISTINCT.
* Validates mandatory fields including event ID, user ID, product ID, category, event type, quantity, price, and timestamp.
* Standardizes category and event_type values using trimming and lowercase normalization.
* Computes event revenue as Quantity × Price.
* Filters invalid records containing null values or non-positive quantities and prices.
* Stores cleansed records in the flash_sale_silver table for downstream aggregation.

---
## Data Quality Rules

The Silver layer enforces several quality checks before records are promoted:

* Duplicate events are removed.
* Missing Event IDs are rejected.
* Missing User IDs are rejected.
* Missing Product IDs are rejected.
* Missing Categories are rejected.
* Missing Event Types are rejected.
* Quantity must be greater than zero.
* Price must be greater than zero.
* Timestamp must be present.
* Category values are trimmed and converted to lowercase.
* Event Type values are trimmed and converted to lowercase.
* Revenue is recalculated to ensure consistency.

--- 
## Pipeline Role

The Silver layer serves as the data quality and transformation stage of the retail analytics pipeline. By validating, cleansing, and standardizing incoming retail events, it ensures that downstream business metrics and dashboards are generated from accurate, consistent, and trustworthy data.

This layer acts as the bridge between raw event ingestion and business-level analytics, significantly improving the reliability of reporting and decision-making.

---

## Source Code

```SQL
DROP TABLE IF EXISTS flash_sale_silver;

CREATE TABLE flash_sale_silver (
    silver_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    event_id VARCHAR(50),
    user_id VARCHAR(50),
    product_id VARCHAR(50),
    category VARCHAR(50),
    event_type VARCHAR(20),
    quantity INT,
    price FLOAT,
    revenue FLOAT,
    ts TIMESTAMP
);

INSERT INTO flash_sale_silver (
    event_id,
    user_id,
    product_id,
    category,
    event_type,
    quantity,
    price,
    revenue,
    ts
)
SELECT DISTINCT
    event_id,
    user_id,
    product_id,
    LOWER(TRIM(category)),
    LOWER(TRIM(event_type)),
    quantity,
    price,
    quantity * price,
    ts
FROM flash_sale_events
WHERE
    event_id IS NOT NULL
    AND user_id IS NOT NULL
    AND product_id IS NOT NULL
    AND category IS NOT NULL
    AND event_type IS NOT NULL
    AND quantity > 0
    AND price > 0
    AND ts IS NOT NULL;
```
---
## Console Output

(Insert screenshot of successful SQL execution here.)

---

## 📂 Silver Layer Output

The Silver layer produces a structured, validated dataset optimized for business analytics.

Generated Dataset

MySQL\
└── flash_sale_db/\
    └── flash_sale_silver

Each record contains:

* Silver ID
* Event ID
* User ID
* Product ID
* Category
* Event Type
* Quantity
* Price
* Revenue
* Timestamp

The resulting dataset serves as the trusted input for the Gold layer, where product-level aggregations, customer behavior metrics, conversion analysis, cart abandonment rates, demand pressure, and other business KPIs are computed.

---

## ✅ Silver Layer Summary

The Silver layer transforms raw streaming retail events into a clean, validated, and analytics-ready dataset through comprehensive data quality enforcement, normalization, deduplication, and derived metric computation. By ensuring data consistency and integrity before aggregation, it establishes a reliable foundation for the Gold layer, enabling accurate KPI generation, business reporting, and real-time retail analytics.

