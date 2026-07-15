# Event Generator

### Purpose

The Event Generator simulates continuous customer activity across an e-commerce platform by generating realistic retail interaction events. It models the complete customer journey—including product views, cart additions, and purchases—to create a steady stream of transactional data for real-time analytics.

A synthetic product catalog containing categories, brands, pricing, and weighted popularity is used to emulate realistic shopping behavior. Generated events are serialized as JSON and continuously published to the Apache Kafka topic flash_sale_events, providing the raw event stream that drives the downstream ETL pipeline.

### Key Responsibilities

* Simulates continuous customer interactions including view, cart, and purchase events.
* Generates activity for 10,000 users interacting with a catalog of 1,000 products.
* Produces realistic event payloads containing user, product, category, brand, quantity, price, revenue, and timestamp information.
* Implements weighted product selection to mimic varying product popularity and customer purchasing behavior.
* Serializes events as JSON and streams them continuously to the flash_sale_events Kafka topic.
* Maintains a configurable event throughput for consistent real-time streaming and downstream ETL processing.

### Pipeline Role

The Event Generator serves as the primary data ingestion source for the retail analytics pipeline. By continuously producing realistic customer interaction events, it establishes a stable stream of transactional data that forms the foundation of the Bronze layer and enables downstream cleansing, transformation, KPI computation, and business reporting.

--- 

## Source Code:
```python
import json
import random
import time
from datetime import datetime
from kafka import KafkaProducer

# ---------------- CONFIG ----------------
KAFKA_TOPIC = "flash_sale_events"
KAFKA_SERVER = "localhost:9092"
EVENTS_PER_SECOND = 5
NUM_PRODUCTS = 1000
NUM_USERS = 10000

# ---------------- PRODUCER ----------------
producer = KafkaProducer(
    bootstrap_servers=KAFKA_SERVER,
    value_serializer=lambda v: json.dumps(v).encode("utf-8")
)

# ---------------- STATIC DATA ----------------
CATEGORIES = [
    "electronics","fashion","home","sports","beauty",
    "books","toys","grocery","automotive","gaming"
]

BRANDS = [
    "Nike","Apple","Samsung","Sony","Adidas",
    "Puma","Dell","HP","Lenovo","Asus"
]

EVENT_TYPES = ["view", "cart", "purchase"]

# ---------------- PRODUCT CATALOG ----------------
PRODUCTS = []

for i in range(NUM_PRODUCTS):
    PRODUCTS.append({
        "product_id": random.randint(100000, 999999),
        "name": f"product_{i}",
        "category": random.choice(CATEGORIES),
        "brand": random.choice(BRANDS),
        "price": round(random.uniform(10, 2000), 2),
        "weight": random.randint(1, 10)
    })

PRODUCT_WEIGHTS = [p["weight"] for p in PRODUCTS]

# ---------------- EVENT FUNCTION ----------------
def generate_event():
    product = random.choices(PRODUCTS, weights=PRODUCT_WEIGHTS, k=1)[0]
    qty = random.randint(1, 3)

    return {
        "event_id": str(random.randint(10**10, 10**11)),
        "event_source": "user",

        "user_id": str(random.randint(10000, 99999)),
        "product_id": str(product["product_id"]),
        "category": product["category"],
        "brand": product["brand"],

        "event_type": random.choice(EVENT_TYPES),

        "quantity": qty,
        "price": product["price"],
        "revenue": round(qty * product["price"], 2),

        "timestamp": datetime.utcnow().isoformat()
    }

# ---------------- STREAM LOOP ----------------
print("Streaming USER events... Ctrl+C to stop")

try:
    while True:
        start = time.time()

        for _ in range(EVENTS_PER_SECOND):
            event = generate_event()
            producer.send(KAFKA_TOPIC, event)
            print(event)

        producer.flush()

        elapsed = time.time() - start
        time.sleep(max(0, 1 - elapsed))

except KeyboardInterrupt:
    print("\nStopped.")
```
---

## Console Output:
<img width="1920" height="1080" alt="Screenshot (48)" src="https://github.com/user-attachments/assets/357b40d3-bd87-42de-acd7-1253ccaa59cc" />
