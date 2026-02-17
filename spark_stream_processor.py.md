## Summary:
•	Real-time ETL pipeline consuming Kafka flash_sale_events stream, parsing JSON, applying schema, and computing revenue per event.\
•	Data quality handling: fills missing values for user_id, category, price, quantity, filters invalid rows, and calculates batch-level metrics.\
•	Batch writes to MySQL with fault-tolerance, appending clean, analytics-ready data for downstream KPI and reporting layers.\
•	Streaming orchestration with Spark Structured Streaming, using 5-second micro-batches for low-latency processing and dashboard-ready insights.

## SQL Query:
```SQL
DROP TABLE IF EXISTS flash_sale_events;

CREATE TABLE flash_sale_events (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    event_id VARCHAR(50),
    user_id VARCHAR(50),
    product_id VARCHAR(50),
    category VARCHAR(50),
    event_type VARCHAR(20),
    price FLOAT,
    quantity INT,
    revenue FLOAT,
    ts TIMESTAMP
);
```

## Python Code:
```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
from pyspark.sql.types import *

# ---------------- SPARK ----------------
spark = SparkSession.builder \
    .appName("FlashSaleProcessor") \
    .config(
        "spark.jars",
        "C:/Users/mysql-connector-j-8.0.32/mysql-connector-j-8.0.32.jar"
    ) \
    .config(
        "spark.jars.packages",
        "org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.0"
    ) \
    .getOrCreate()

spark.sparkContext.setLogLevel("ERROR")

# ---------------- SCHEMA ----------------
schema = StructType([
    StructField("event_id", StringType()),
    StructField("user_id", StringType()),
    StructField("product_id", StringType()),
    StructField("category", StringType()),
    StructField("event_type", StringType()),
    StructField("price", FloatType()),
    StructField("quantity", IntegerType()),
    StructField("timestamp", StringType())
])

# ---------------- READ KAFKA ----------------
df = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers","localhost:9092") \
    .option("subscribe","flash_sale_events") \
    .option("startingOffsets","latest") \
    .load()

# ---------------- PARSE ----------------
parsed = df.selectExpr("CAST(value AS STRING) as json") \
    .select(from_json(col("json"), schema).alias("data")) \
    .select("data.*")

# ---------------- FIX MISSING VALUES ----------------
cleaned = parsed \
    .withColumn("user_id", when(col("user_id").isNull(), concat(lit("guest_"), col("product_id")) ).otherwise(col("user_id")) ) \
    .withColumn("category", when(col("category").isNull(), concat(lit("category_"), substring(col("product_id"),1,2)) ).otherwise(col("category")) ) \
    .withColumn("quantity", when(col("quantity").isNull(), lit(1)) .otherwise(col("quantity")) ) \
    .withColumn("price", when(col("price").isNull(), lit(0.0)) .otherwise(col("price")) ) \
    .withColumn("revenue", col("price") * col("quantity")) \
    .withColumn("ts", col("timestamp").cast(TimestampType())) \
    .drop("timestamp")


# ---------------- MYSQL WRITER ----------------
def write_to_mysql(batch_df, batch_id):

    print(f"\n========== BATCH {batch_id} ==========")

    if batch_df.count() == 0:
        print("Empty batch → skipping")
        return

    try:
        # DATA QUALITY FILTER
        clean_df = batch_df.filter(
            col("event_id").isNotNull() &
            col("user_id").isNotNull() &
            col("category").isNotNull()
        )

        valid_count = clean_df.count()
        dropped = batch_df.count() - valid_count

        print(f"Valid rows → {valid_count}")
        print(f"Dropped invalid rows → {dropped}")

        if valid_count == 0:
            print("No valid rows → skipping write")
            return

        clean_df.write \
            .format("jdbc") \
            .option("url", "jdbc:mysql://localhost:3306/flash_sale_db?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC") \
            .option("driver", "com.mysql.cj.jdbc.Driver") \
            .option("dbtable", "flash_sale_events") \
            .option("user", "xxxx") \
            .option("password", "xxxx") \
            .mode("append") \
            .save()

        print("Batch written successfully ✅")

    except Exception as e:
        print("MYSQL WRITE ERROR")
        print(e)


# ---------------- STREAM ----------------
query = cleaned.writeStream \
    .foreachBatch(write_to_mysql) \
    .outputMode("append") \
    .trigger(processingTime="5 seconds") \
    .start()

query.awaitTermination()
```
