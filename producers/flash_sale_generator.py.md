## Summary:
•	Simulates high-volume flash sale events by generating randomized user-product interactions (view, cart, purchase) and streaming them to a Kafka topic.\
•	Dynamic traffic simulation with normal (5–15 events/sec) and spike modes (40–80 events/sec) to mimic real-world e-commerce demand surges.\
•	Generates rich event payloads including user_id, product_id, category, quantity, price, revenue, and timestamp for analytics pipelines.\
•	Supports continuous streaming with real-time ingestion via KafkaProducer, enabling downstream Bronze-Silver-Gold ETL processing and KPI computation.

## Code:
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
