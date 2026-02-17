## Summary:
•	Simulates continuous user interaction events (view, cart, purchase) for 10,000 users and 1,000 products, streaming to a Kafka topic for analytics pipelines.\
•	Weighted product selection with category, brand, price, and custom weight to mimic realistic user-product engagement.\
•	Generates rich event payloads including user_id, product_id, category, brand, quantity, price, revenue, and timestamp for downstream ETL processing.\
•	Controls event throughput (EVENTS_PER_SECOND) with accurate timing, ensuring steady streaming for Bronze-Silver-Gold data processing and KPI monitoring.


## Code:
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
