## Summary:
•	Silver layer creation: stores cleaned and transformed events from the raw Kafka stream, ensuring deduplication and consistent schema.\
•	Data normalization: trims and lowercases category and event_type fields, computes revenue as quantity × price.\
•	Data quality enforcement: filters out null or invalid values for key fields (event_id, user_id, product_id, category, event_type, quantity, price, ts).\
•	Analytics-ready dataset: generates a structured, clean table suitable for KPI aggregation, dashboarding, and downstream reporting.
  
## Query:
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
