# 🛒 Retail Demand Analytics Engine

## Project Overview
This project implements a *flash sale retail analytics engine* for flash sale events. It ingests streaming sales data, processes it through *Spark Structured Streaming* and *MySQL*, and creates a *Bronze → Silver → Gold* data pipeline to ensure *clean, analytics-ready datasets.  

The pipeline enables *business-critical KPIs* for conversion optimization, cart abandonment analysis, and product trend monitoring, providing actionable insights for *sales and marketing teams*.

---

## Key Achievements
- Processed *10K daily flash sale events* into Bronze-Silver-Gold layers with full *data validation, deduplication, and anomaly detection*.  
- Generated KPI intelligence for revenue optimization:  
- *Conversion rate:* 84.63% of views converted to purchases  
- *Cart abandonment:* 7.88%  
- *Average order value:* $4.78K per purchase  
- *Demand pressure index:* 0.54 (events per unit sold)  
- Built *Power BI dashboards* to visualize product trends, category performance, and sales patterns.  
- Standardized *data quality and governance protocols*, removing nulls and duplicates to ensure reliable analytics.

---

## Tech Stack
- *Languages & Libraries:* Python 3.x, PySpark, Pandas, SQLAlchemy  
- *Streaming & Storage:* Spark Structured Streaming, MySQL  
- *Visualization:* Power BI  
- *OS:* Cross-platform  

---

## Installation

1. Clone the repository:
    ```bash
    git clone <your-repo-url>
    cd retail-demand-analytics-engine
    ```
2.	Set up a virtual environment:
    ```bash
    python -m venv venv
    # Activate environment
    source venv/bin/activate  # Linux / Mac
    venv\Scripts\activate     # Windows
    ```
3.	Install dependencies:
    ```bash
    pip install -r requirements.txt
    ```
4.	Configure database:

	•	Create MySQL database and tables for Bronze, Silver, and Gold layers as defined in the pipeline scripts.



## Usage

### 1️⃣ Producers

Simulate flash sale events and send to streaming ingestion.

flash_sale_generator.py , event_generator.py

### 2️⃣ Bronze Layer

Ingest raw sales events into Bronze layer with minimal transformation.

spark_stream_processor.py gives data to flash_Sale.sql

### 3️⃣ Silver Layer

Cleanse, validate, and deduplicate raw events for analytics readiness.

flash_sale_silver.sql

### 4️⃣ Gold Layer

Aggregate sales data, compute KPIs, and store analytics-ready datasets.

flash_sale_gold.sql

### 5️⃣ Dashboards

Generate visual insights and monitor sales performance in Power BI.

retail_analytics.pbix


## Insights Delivered
•	Enabled real-time tracking of conversion, cart abandonment, and average order values.\
•	Identified high-demand products and categories for targeted promotions.\
•	Designed scalable, analytics-ready pipelines suitable for live flash sale environments.

## Notes
•	MySQL database must be configured before pipeline execution.\
•	Batch sizes and micro-batch intervals can be tuned for performance\
•	Fully extendable to other retail events or product catalogs.\
•	Ensures high data quality and governance, eliminating nulls and duplicates.
