# Flash Sale Generator

### Purpose

The Flash Sale Generator simulates high-intensity promotional events by generating bursts of customer activity that resemble real-world flash sale traffic. Unlike the standard Event Generator, this component dynamically alternates between normal and spike traffic conditions, allowing the pipeline to process sudden increases in event volume and evaluate streaming performance under peak demand.

Each generated event contains customer interaction details, product information, pricing, quantity, revenue, and timestamps before being published to the Apache Kafka topic flash_sale_events for downstream processing.

### Key Responsibilities

* Simulates flash sale customer interactions including view, cart, and purchase events.
* Generates dynamic traffic patterns by alternating between normal (5–15 events/sec) and spike (40–80 events/sec) workloads.
* Produces realistic retail event payloads containing user, product, category, quantity, price, revenue, and timestamp information.
* Models sudden demand surges commonly observed during promotional campaigns and limited-time sales.
* Continuously publishes streaming JSON events to the flash_sale_events Kafka topic.
* Enables downstream analysis of customer demand, conversion behavior, inventory pressure, and sales performance during high-traffic periods.

### Pipeline Role

The Flash Sale Generator functions as the high-volume event simulation service within the streaming architecture. By introducing unpredictable traffic spikes alongside regular customer activity, it stress-tests the ingestion pipeline, validates streaming scalability, and provides realistic retail workloads for downstream Bronze, Silver, and Gold processing.

--- 

## Source Code:
```python
import json
import time
import random
from datetime import datetime
from kafka import KafkaProducer

producer = KafkaProducer(
    bootstrap_servers="localhost:9092",
    value_serializer=lambda v: json.dumps(v).encode("utf-8")
)

TOPIC = "flash_sale_events"

product_ids = list(range(1000, 1100))
EVENT_TYPES = ["view", "cart", "purchase"]

def generate_event():
    price = round(random.uniform(20, 500), 2)
    qty = random.randint(1, 3)

    return {
        "event_id": str(random.randint(100000, 999999)),
        "event_source": "flash_sale",

        "user_id": random.randint(10000,99999),
        "product_id": str(random.choice(product_ids)),
        "category": random.choice(["electronics","fashion","home","sports","beauty","books","toys","grocery"]),
        "brand": None,

        "event_type": random.choice(EVENT_TYPES),

        "quantity": qty,
        "price": round(random.uniform(10,2000),2),
        "revenue": round(qty * price, 2),

        "timestamp": datetime.utcnow().isoformat()
    }

print("Streaming FLASH SALE events... Ctrl+C to stop")

try:
    while True:

        traffic_mode = random.choices(
            ["normal", "spike"],
            weights=[0.8, 0.2]
        )[0]

        if traffic_mode == "normal":
            rate = random.randint(5, 15)
        else:
            rate = random.randint(40, 80)
            print("🔥 SPIKE TRAFFIC!")

        for _ in range(rate):
            event = generate_event()
            producer.send(TOPIC, event)
            print(event)

        producer.flush()
        time.sleep(1)

except KeyboardInterrupt:
    print("\nStopped.")
```

## Console Output:

<img width="1920" height="1080" alt="Screenshot (49)" src="https://github.com/user-attachments/assets/dd662a08-716a-47b6-ba03-27f8ce1491e8" />
