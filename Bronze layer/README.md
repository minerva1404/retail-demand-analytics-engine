# 🥉 Bronze Layer

### Overview

The Bronze layer serves as the raw ingestion layer of the retail analytics pipeline. It continuously captures simulated customer interactions and flash sale events, publishing them to Apache Kafka before ingesting them into the Bronze MySQL table through Spark Structured Streaming.

This layer preserves the original event structure with minimal transformation, creating a reliable source of truth for downstream data cleansing and business analytics.

By separating event generation from analytical processing, the Bronze layer enables scalable, fault-tolerant streaming while maintaining complete event lineage across the pipeline.

---

### 🎯 Objectives

* Simulate realistic retail customer interactions in real time.
* Generate high-volume flash sale traffic with dynamic load patterns.
* Stream raw events into Apache Kafka with minimal preprocessing.
* Preserve original event payloads for downstream transformations.
* Establish a reliable streaming foundation for Spark Structured Streaming.
* Provide replayable raw datasets for Silver layer processing.

---

### 🔄 Bronze Layer Workflow

Normal User Activity\
        ↓\
User Activity Producer
(event_generator.py)\
        ↓\
Apache Kafka Topic
(flash_sale_events)\
        ↑\
Flash Sale Producer
(flash_sale_generator.py)\
        ↓\
High Traffic Simulation

⸻

## Component 1 — Event Generator

### Purpose

The Event genarator Producer simulates continuous customer interactions across an e-commerce platform by generating realistic user behavior events. It models how customers browse products, add items to their carts, and complete purchases, creating a continuous stream of retail events for downstream analytics.

The producer publishes these events directly to the Kafka topic flash_sale_events, providing a realistic streaming workload for the data pipeline.

### Key Responsibilities

* Simulates customer view, cart, and purchase events.
* Generates activity for 10,000 users across a catalog of 1,000 products.
* Produces realistic product metadata including category, brand, price, quantity, and revenue.
* Uses weighted product selection to mimic real customer purchasing behavior.
* Serializes events as JSON and streams them continuously to Apache Kafka.
* Maintains a configurable event rate for consistent streaming throughput.

### Pipeline Role

The Event generator Producer acts as the primary event generation service for the retail analytics pipeline. By simulating normal customer behavior, it continuously feeds raw interaction events into Kafka, establishing the baseline workload for downstream Spark processing and KPI generation.

### Console Output

<img width="1920" height="1080" alt="Screenshot (48)" src="https://github.com/user-attachments/assets/29e9a431-c890-4ff8-9c75-67415cacb1de" />


---
## Component 2 — Flash Sale Generator

### Purpose

The Flash Sale Generator simulates high-demand promotional events by generating bursts of customer activity that closely resemble real-world flash sale traffic. It introduces dynamic traffic patterns, allowing the streaming pipeline to process both steady-state workloads and sudden demand spikes.

This producer publishes flash sale events directly to the flash_sale_events Kafka topic, enabling stress testing of the ingestion pipeline and supporting real-time retail analytics.

### Key Responsibilities

* Simulates flash sale view, cart, and purchase events.
* Generates both normal traffic (5–15 events/sec) and spike traffic (40–80 events/sec).
* Produces randomized customer interactions with product, category, quantity, price, revenue, and timestamp information.
* Streams serialized JSON events continuously to Apache Kafka.
* Mimics realistic demand surges during promotional campaigns.
* Supports downstream computation of conversion, demand pressure, and purchasing trends.

### Pipeline Role

The Flash Sale Genarator functions as the high-volume event generator within the streaming architecture. By introducing sudden traffic spikes alongside normal user activity, it enables the pipeline to evaluate scalability, throughput, and business KPI computation under realistic retail workloads.

### Console Output

<img width="1920" height="1080" alt="Screenshot (49)" src="https://github.com/user-attachments/assets/0273bd39-672b-484a-a59a-e39ef4fc91e3" />


---

## 📂 Bronze Layer Output

The Bronze layer produces a continuous stream of raw retail events that are consumed by Spark Structured Streaming and persisted into the Bronze MySQL table with minimal transformation.

Generated Dataset

MySQL\
└── flash_sale_db/\
    └── flash_sale_events

Each record represents a raw customer interaction and includes:

* Event ID
* Event Source
* User ID
* Product ID
* Category
* Brand
* Event Type
* Quantity
* Price
* Revenue
* Timestamp

These raw events become the input for the Silver layer, where records undergo validation, cleansing, normalization, deduplication, and data quality checks before being transformed into analytics-ready datasets.

---

## ✅ Bronze Layer Summary

The Bronze layer establishes the streaming foundation of the Retail Demand Analytics Engine by combining two complementary Kafka producers that simulate both everyday customer activity and high-volume flash sale traffic. Together, they generate realistic retail event streams that preserve raw business events, support scalable Spark ingestion, and provide a reliable source of truth for downstream Silver and Gold transformations.
