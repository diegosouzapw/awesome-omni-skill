---
name: apache-spark-data-processing
description: Complete guide for Apache Spark data processing including RDDs, DataFrames, Spark SQL, streaming, MLlib, and production deployment
tags: [spark, big-data, distributed-computing, dataframes, streaming, machine-learning]
tier: tier-1
---

# Apache Spark Data Processing

A comprehensive skill for mastering Apache Spark data processing, from basic RDD operations to advanced streaming, SQL, and machine learning workflows. Learn to build scalable, distributed data pipelines and analytics systems.

## When to Use This Skill

Use Apache Spark when you need to:

- **Process Large-Scale Data**: Handle datasets too large for single-machine processing (TB to PB scale)
- **Perform Distributed Computing**: Execute parallel computations across cluster nodes
- **Real-Time Stream Processing**: Process continuous data streams with low latency
- **Complex Data Analytics**: Run sophisticated analytics, aggregations, and transformations
- **Machine Learning at Scale**: Train ML models on massive datasets
- **ETL/ELT Pipelines**: Build robust data transformation and loading workflows
- **Interactive Data Analysis**: Perform exploratory analysis on large datasets
- **Unified Data Processing**: Combine batch and streaming workloads in one framework

**Not Ideal For:**
- Small datasets (<100 GB) that fit in memory on a single machine
- Simple CRUD operations (use traditional databases)
- Ultra-low latency requirements (<10ms) where specialized stream processors excel
- Workflows requiring strong ACID transactions across distributed data

## Core Concepts

### Resilient Distributed Datasets (RDDs)

RDDs are Spark's fundamental data abstraction - immutable, distributed collections of objects that can be processed in parallel.

**Key Characteristics:**
- **Resilient**: Fault-tolerant through lineage tracking
- **Distributed**: Partitioned across cluster nodes
- **Immutable**: Transformations create new RDDs, not modify existing ones
- **Lazy Evaluation**: Transformations build computation graph; actions trigger execution
- **In-Memory Computing**: Cache intermediate results for iterative algorithms

**RDD Operations:**
- **Transformations**: Lazy operations that return new RDDs (map, filter, flatMap, reduceByKey)
- **Actions**: Operations that trigger computation and return values (collect, count, reduce, saveAsTextFile)

**When to Use RDDs:**
- Low-level control over data distribution and partitioning
- Custom partitioning schemes required
- Working with unstructured data (text files, binary data)
- Migrating legacy code from early Spark versions

**Prefer DataFrames/Datasets when possible** - they provide automatic optimization via Catalyst optimizer.

### DataFrames and Datasets

DataFrames are distributed collections of data organized into named columns - similar to a database table or pandas DataFrame, but with powerful optimizations.

**DataFrames:**
- Structured data with schema
- Automatic query optimization (Catalyst)
- Cross-language support (Python, Scala, Java, R)
- Rich API for SQL-like operations

**Datasets (Scala/Java only):**
- Typed DataFrames with compile-time type safety
- Best performance in Scala due to JVM optimization
- Combine RDD type safety with DataFrame optimizations

**Key Advantages Over RDDs:**
- **Query Optimization**: Catalyst optimizer rewrites queries for efficiency
- **Tungsten Execution**: Optimized CPU and memory usage
- **Columnar Storage**: Efficient data representation
- **Code Generation**: Compile-time bytecode generation for faster execution

### Lazy Evaluation

Spark uses lazy evaluation to optimize execution:

1. **Transformations** build a Directed Acyclic Graph (DAG) of operations
2. **Actions** trigger execution of the DAG
3. Spark's optimizer analyzes the entire DAG and creates an optimized execution plan
4. Work is distributed across cluster nodes

**Benefits:**
- Minimize data movement across network
- Combine multiple operations into single stage
- Eliminate unnecessary computations
- Optimize memory usage

### Partitioning

Data is divided into partitions for parallel processing:

- **Default Partitioning**: Typically based on HDFS block size or input source
- **Hash Partitioning**: Distribute data by key hash (used by groupByKey, reduceByKey)
- **Range Partitioning**: Distribute data by key ranges (useful for sorted data)
- **Custom Partitioning**: Define your own partitioning logic

**Partition Count Considerations:**
- Too few partitions: Underutilized cluster, large task execution time
- Too many partitions: Scheduling overhead, small task execution time
- General rule: 2-4 partitions per CPU core in cluster
- Use `repartition()` or `coalesce()` to adjust partition count

### Caching and Persistence

Cache frequently accessed data in memory for performance:

```python
# Cache DataFrame in memory
df.cache()  # Shorthand for persist(StorageLevel.MEMORY_AND_DISK)

# Different storage levels
df.persist(StorageLevel.MEMORY_ONLY)       # Fast but may lose data if evicted
df.persist(StorageLevel.MEMORY_AND_DISK)   # Spill to disk if memory full
df.persist(StorageLevel.DISK_ONLY)         # Store only on disk
df.persist(StorageLevel.MEMORY_ONLY_SER)   # Serialized in memory (more compact)

# Unpersist when done
df.unpersist()
```

**When to Cache:**
- Data used multiple times in workflow
- Iterative algorithms (ML training)
- Interactive analysis sessions
- Expensive transformations reused downstream

**When Not to Cache:**
- Data used only once
- Very large datasets that exceed cluster memory
- Streaming applications with continuous new data

### Spark SQL

Spark SQL allows you to query structured data using SQL or DataFrame API:

- Execute SQL queries on DataFrames and tables
- Register DataFrames as temporary views
- Join structured and semi-structured data
- Connect to Hive metastore for table metadata
- Support for various data sources (Parquet, ORC, JSON, CSV, JDBC)

**Performance Features:**
- **Catalyst Optimizer**: Rule-based and cost-based query optimization
- **Tungsten Execution Engine**: Whole-stage code generation, vectorized processing
- **Adaptive Query Execution (AQE)**: Runtime optimization based on statistics
- **Dynamic Partition Pruning**: Skip irrelevant partitions during execution

### Broadcast Variables and Accumulators

Shared variables for efficient distributed computing:

**Broadcast Variables:**
- Read-only variables cached on each node
- Efficient for sharing large read-only data (lookup tables, ML models)
- Avoid sending large data with every task

```python
# Broadcast a lookup table
lookup_table = {"key1": "value1", "key2": "value2"}
broadcast_lookup = sc.broadcast(lookup_table)

# Use in transformations
rdd.map(lambda x: broadcast_lookup.value.get(x, "default"))
```

**Accumulators:**
- Write-only variables for aggregating values across tasks
- Used for counters and sums in distributed operations
- Only driver can read final accumulated value

```python
# Create accumulator
error_count = sc.accumulator(0)

# Increment in tasks
rdd.foreach(lambda x: error_count.add(1) if is_error(x) else None)

# Read final value in driver
print(f"Total errors: {error_count.value}")
```

## Spark SQL Deep Dive

### DataFrame Creation

Create DataFrames from various sources:

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("SparkSQLExample").getOrCreate()

# From structured data
data = [("Alice", 1), ("Bob", 2), ("Charlie", 3)]
columns = ["name", "id"]
df = spark.createDataFrame(data, columns)

# From files
df_json = spark.read.json("path/to/file.json")
df_parquet = spark.read.parquet("path/to/file.parquet")
df_csv = spark.read.option("header", "true").csv("path/to/file.csv")

# From JDBC sources
df_jdbc = spark.read \
    .format("jdbc") \
    .option("url", "jdbc:postgresql://host:port/database") \
    .option("dbtable", "table_name") \
    .option("user", "username") \
    .option("password", "password") \
    .load()
```

### DataFrame Operations

Common DataFrame transformations:

```python
# Select columns
df.select("name", "age").show()

# Filter rows
df.filter(df.age > 21).show()
df.where(df["age"] > 21).show()  # Alternative syntax

# Add/modify columns
from pyspark.sql.functions import col, lit
df.withColumn("age_plus_10", col("age") + 10).show()
df.withColumn("country", lit("USA")).show()

# Aggregations
df.groupBy("department").count().show()
df.groupBy("department").agg({"salary": "avg", "age": "max"}).show()

# Sorting
df.orderBy("age").show()
df.orderBy(col("age").desc()).show()

# Joins
df1.join(df2, df1.id == df2.user_id, "inner").show()
df1.join(df2, "id", "left_outer").show()

# Unions
df1.union(df2).show()
```

### SQL Queries

Execute SQL on DataFrames:

```python
# Register DataFrame as temporary view
df.createOrReplaceTempView("people")

# Run SQL queries
sql_result = spark.sql("SELECT name FROM people WHERE age > 21")
sql_result.show()

# Complex queries
result = spark.sql("""
    SELECT
        department,
        COUNT(*) as employee_count,
        AVG(salary) as avg_salary,
        MAX(age) as max_age
    FROM people
    WHERE age > 25
    GROUP BY department
    HAVING COUNT(*) > 5
    ORDER BY avg_salary DESC
""")
result.show()
```

### Data Sources

Spark SQL supports multiple data formats:

**Parquet** (Recommended for Analytics):
- Columnar storage format
- Excellent compression and query performance
- Schema embedded in file
- Supports predicate pushdown and column pruning

```python
# Write
df.write.parquet("output/path", mode="overwrite", compression="snappy")

# Read with partition pruning
df = spark.read.parquet("output/path").filter(col("date") == "2025-01-01")
```

**ORC** (Optimized Row Columnar):
- Similar to Parquet with slightly better compression
- Preferred for Hive integration
- Built-in indexes for faster queries

```python
df.write.orc("output/path", mode="overwrite")
df = spark.read.orc("output/path")
```

**JSON** (Semi-Structured Data):
- Human-readable but less efficient
- Schema inference on read
- Good for nested/complex data

```python
# Read with explicit schema
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

schema = StructType([
    StructField("name", StringType(), True),
    StructField("age", IntegerType(), True)
])

df = spark.read.schema(schema).json("data.json")
```

**CSV** (Legacy/Simple Data):
- Widely compatible but slow
- Requires header inference or explicit schema
- Minimal compression benefits

```python
df.write.csv("output.csv", header=True, mode="overwrite")
df = spark.read.option("header", "true").option("inferSchema", "true").csv("data.csv")
```

### Window Functions

Advanced analytics with window functions:

```python
from pyspark.sql.window import Window
from pyspark.sql.functions import row_number, rank, dense_rank, lag, lead, sum, avg

# Define window specification
window_spec = Window.partitionBy("department").orderBy(col("salary").desc())

# Ranking functions
df.withColumn("rank", rank().over(window_spec)).show()
df.withColumn("row_num", row_number().over(window_spec)).show()
df.withColumn("dense_rank", dense_rank().over(window_spec)).show()

# Aggregate functions over window
df.withColumn("dept_avg_salary", avg("salary").over(window_spec)).show()
df.withColumn("running_total", sum("salary").over(window_spec.rowsBetween(Window.unboundedPreceding, Window.currentRow))).show()

# Offset functions
df.withColumn("prev_salary", lag("salary", 1).over(window_spec)).show()
df.withColumn("next_salary", lead("salary", 1).over(window_spec)).show()
```

### User-Defined Functions (UDFs)

Create custom transformations:

```python
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType, IntegerType

# Python UDF (slower due to serialization overhead)
def categorize_age(age):
    if age < 18:
        return "Minor"
    elif age < 65:
        return "Adult"
    else:
        return "Senior"

categorize_udf = udf(categorize_age, StringType())
df.withColumn("age_category", categorize_udf(col("age"))).show()

# Pandas UDF (vectorized, faster for large datasets)
from pyspark.sql.functions import pandas_udf
import pandas as pd

@pandas_udf(IntegerType())
def square(series: pd.Series) -> pd.Series:
    return series ** 2

df.withColumn("age_squared", square(col("age"))).show()
```

**UDF Performance Tips:**
- Use built-in Spark functions when possible (always faster)
- Prefer Pandas UDFs over Python UDFs for better performance
- Use Scala UDFs for maximum performance (no serialization overhead)
- Cache DataFrames before applying UDFs if used multiple times

## Transformations and Actions

### Common Transformations

**map**: Apply function to each element

```python
# RDD
rdd = sc.parallelize([1, 2, 3, 4, 5])
squared = rdd.map(lambda x: x * 2)  # [2, 4, 6, 8, 10]

# DataFrame (use select with functions)
from pyspark.sql.functions import col
df.select(col("value") * 2).show()
```

**filter**: Select elements matching predicate

```python
# RDD
rdd.filter(lambda x: x > 2).collect()  # [3, 4, 5]

# DataFrame
df.filter(col("age") > 25).show()
```

**flatMap**: Map and flatten results

```python
# RDD - Split text into words
lines = sc.parallelize(["hello world", "apache spark"])
words = lines.flatMap(lambda line: line.split(" "))  # ["hello", "world", "apache", "spark"]
```

**reduceByKey**: Aggregate values by key

```python
# Word count example
words = sc.parallelize(["apple", "banana", "apple", "cherry", "banana", "apple"])
word_pairs = words.map(lambda word: (word, 1))
word_counts = word_pairs.reduceByKey(lambda a, b: a + b)
# Result: [("apple", 3), ("banana", 2), ("cherry", 1)]
```

**groupByKey**: Group values by key (avoid when possible - use reduceByKey instead)

```python
# Less efficient than reduceByKey
word_pairs.groupByKey().mapValues(list).collect()
# Result: [("apple", [1, 1, 1]), ("banana", [1, 1]), ("cherry", [1])]
```

**join**: Combine datasets by key

```python
# RDD join
users = sc.parallelize([("user1", "Alice"), ("user2", "Bob")])
orders = sc.parallelize([("user1", 100), ("user2", 200), ("user1", 150)])
users.join(orders).collect()
# Result: [("user1", ("Alice", 100)), ("user1", ("Alice", 150)), ("user2", ("Bob", 200))]

# DataFrame join (more efficient)
df_users.join(df_orders, "user_id", "inner").show()
```

**distinct**: Remove duplicates

```python
# RDD
rdd.distinct().collect()

# DataFrame
df.distinct().show()
df.dropDuplicates(["user_id"]).show()  # Drop based on specific columns
```

**coalesce/repartition**: Change partition count

```python
# Reduce partitions (no shuffle, more efficient)
df.coalesce(1).write.parquet("output")

# Increase/decrease partitions (involves shuffle)
df.repartition(10).write.parquet("output")
df.repartition(10, "user_id").write.parquet("output")  # Partition by column
```

### Common Actions

**collect**: Retrieve all data to driver

```python
results = rdd.collect()  # Returns list
# WARNING: Only use on small datasets that fit in driver memory
```

**count**: Count elements

```python
total = df.count()  # Number of rows
```

**first/take**: Get first N elements

```python
first_elem = rdd.first()
first_five = rdd.take(5)
```

**reduce**: Aggregate all elements

```python
total_sum = rdd.reduce(lambda a, b: a + b)
```

**foreach**: Execute function on each element

```python
# Side effects only (no return value)
rdd.foreach(lambda x: print(x))
```

**saveAsTextFile**: Write to file system

```python
rdd.saveAsTextFile("hdfs://path/to/output")
```

**show**: Display DataFrame rows (action)

```python
df.show(20, truncate=False)  # Show 20 rows, don't truncate columns
```

## Structured Streaming

Process continuous data streams using DataFrame API.

### Core Concepts

**Streaming DataFrame:**
- Unbounded table that grows continuously
- Same operations as batch DataFrames
- Micro-batch processing (default) or continuous processing

**Input Sources:**
- File sources (JSON, Parquet, CSV, ORC, text)
- Kafka
- Socket (for testing)
- Rate source (for testing)
- Custom sources

**Output Modes:**
- **Append**: Only new rows added to result table
- **Complete**: Entire result table written every trigger
- **Update**: Only updated rows written

**Output Sinks:**
- File sinks (Parquet, ORC, JSON, CSV, text)
- Kafka
- Console (for debugging)
- Memory (for testing)
- Foreach/ForeachBatch (custom logic)

### Basic Streaming Example

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col

spark = SparkSession.builder.appName("StreamingExample").getOrCreate()

# Read stream from JSON files
input_stream = spark.readStream \
    .format("json") \
    .schema(schema) \
    .option("maxFilesPerTrigger", 1) \
    .load("input/directory")

# Transform streaming data
processed = input_stream \
    .filter(col("value") > 10) \
    .select("id", "value", "timestamp")

# Write stream to Parquet
query = processed.writeStream \
    .format("parquet") \
    .option("path", "output/directory") \
    .option("checkpointLocation", "checkpoint/directory") \
    .outputMode("append") \
    .start()

# Wait for termination
query.awaitTermination()
```

### Stream-Static Joins

Join streaming data with static reference data:

```python
# Static DataFrame (loaded once)
static_df = spark.read.parquet("reference/data")

# Streaming DataFrame
streaming_df = spark.readStream.format("kafka").load()

# Inner join (supported)
joined = streaming_df.join(static_df, "type")

# Left outer join (supported)
joined = streaming_df.join(static_df, "type", "left_outer")

# Write result
joined.writeStream \
    .format("parquet") \
    .option("path", "output") \
    .option("checkpointLocation", "checkpoint") \
    .start()
```

### Windowed Aggregations

Aggregate data over time windows:

```python
from pyspark.sql.functions import window, col, count

# 10-minute tumbling window
windowed_counts = streaming_df \
    .groupBy(
        window(col("timestamp"), "10 minutes"),
        col("word")
    ) \
    .count()

# 10-minute sliding window with 5-minute slide
windowed_counts = streaming_df \
    .groupBy(
        window(col("timestamp"), "10 minutes", "5 minutes"),
        col("word")
    ) \
    .count()

# Write to console for debugging
query = windowed_counts.writeStream \
    .outputMode("complete") \
    .format("console") \
    .option("truncate", "false") \
    .start()
```

### Watermarking for Late Data

Handle late-arriving data with watermarks:

```python
from pyspark.sql.functions import window

# Define watermark (10 minutes tolerance for late data)
windowed_counts = streaming_df \
    .withWatermark("timestamp", "10 minutes") \
    .groupBy(
        window(col("timestamp"), "10 minutes"),
        col("word")
    ) \
    .count()

# Data arriving more than 10 minutes late will be dropped
```

**Watermark Benefits:**
- Limit state size by dropping old aggregation state
- Handle late data within tolerance window
- Improve performance by not maintaining infinite state

### Session Windows

Group events into sessions based on inactivity gaps:

```python
from pyspark.sql.functions import session_window, when

# Dynamic session window based on user
session_window_spec = session_window(
    col("timestamp"),
    when(col("userId") == "user1", "5 seconds")
    .when(col("userId") == "user2", "20 seconds")
    .otherwise("5 minutes")
)

sessionized_counts = streaming_df \
    .withWatermark("timestamp", "10 minutes") \
    .groupBy(session_window_spec, col("userId")) \
    .count()
```

### Stateful Stream Processing

Maintain state across micro-batches:

```python
from pyspark.sql.functions import expr

# Deduplication using state
deduplicated = streaming_df \
    .withWatermark("timestamp", "1 hour") \
    .dropDuplicates(["user_id", "event_id"])

# Stream-stream joins (stateful)
stream1 = spark.readStream.format("kafka").option("subscribe", "topic1").load()
stream2 = spark.readStream.format("kafka").option("subscribe", "topic2").load()

joined = stream1 \
    .withWatermark("timestamp", "10 minutes") \
    .join(
        stream2.withWatermark("timestamp", "20 minutes"),
        expr("stream1.user_id = stream2.user_id AND stream1.timestamp >= stream2.timestamp AND stream1.timestamp <= stream2.timestamp + interval 15 minutes"),
        "inner"
    )
```

### Checkpointing

Ensure fault tolerance with checkpoints:

```python
# Checkpoint location stores:
# - Stream metadata (offsets, configuration)
# - State information (for stateful operations)
# - Write-ahead logs

query = streaming_df.writeStream \
    .format("parquet") \
    .option("path", "output") \
    .option("checkpointLocation", "checkpoint/dir")  # REQUIRED for production \
    .start()

# Recovery: Restart query with same checkpoint location
# Spark will resume from last committed offset
```

**Checkpoint Best Practices:**
- Always set checkpointLocation for production streams
- Use reliable distributed storage (HDFS, S3) for checkpoints
- Don't delete checkpoint directory while stream is running
- Back up checkpoints for disaster recovery

## Machine Learning with MLlib

Spark's scalable machine learning library.

### Core Components

**MLlib Features:**
- **ML Pipelines**: Chain transformations and models
- **Featurization**: Vector assemblers, scalers, encoders
- **Classification & Regression**: Linear models, tree-based models, neural networks
- **Clustering**: K-means, Gaussian Mixture, LDA
- **Collaborative Filtering**: ALS (Alternating Least Squares)
- **Dimensionality Reduction**: PCA, SVD
- **Model Selection**: Cross-validation, train-test split, parameter tuning

### ML Pipelines

Chain transformations and estimators:

```python
from pyspark.ml import Pipeline
from pyspark.ml.feature import VectorAssembler, StandardScaler
from pyspark.ml.classification import LogisticRegression

# Load data
df = spark.read.format("libsvm").load("data/sample_libsvm_data.txt")

# Define pipeline stages
assembler = VectorAssembler(
    inputCols=["feature1", "feature2", "feature3"],
    outputCol="features"
)

scaler = StandardScaler(
    inputCol="features",
    outputCol="scaled_features",
    withStd=True,
    withMean=True
)

lr = LogisticRegression(
    featuresCol="scaled_features",
    labelCol="label",
    maxIter=10,
    regParam=0.01
)

# Create pipeline
pipeline = Pipeline(stages=[assembler, scaler, lr])

# Split data
train_df, test_df = df.randomSplit([0.8, 0.2], seed=42)

# Train model
model = pipeline.fit(train_df)

# Make predictions
predictions = model.transform(test_df)
predictions.select("label", "prediction", "probability").show()
```

### Feature Engineering

Transform raw data into features:

```python
from pyspark.ml.feature import StringIndexer, OneHotEncoder, VectorAssembler, MinMaxScaler

# Categorical encoding
indexer = StringIndexer(inputCol="category", outputCol="category_index")
encoder = OneHotEncoder(inputCol="category_index", outputCol="category_vec")

# Numerical scaling
scaler = MinMaxScaler(inputCol="features", outputCol="scaled_features")

# Assemble features
assembler = VectorAssembler(
    inputCols=["category_vec", "numeric_feature1", "numeric_feature2"],
    outputCol="features"
)

# Text processing
from pyspark.ml.feature import Tokenizer, HashingTF, IDF

tokenizer = Tokenizer(inputCol="text", outputCol="words")
hashing_tf = HashingTF(inputCol="words", outputCol="raw_features", numFeatures=10000)
idf = IDF(inputCol="raw_features", outputCol="features")
```

### Streaming Linear Regression

Train models on streaming data:

```python
from pyspark.mllib.regression import LabeledPoint
from pyspark.streaming import StreamingContext
from pyspark.streaming.ml import StreamingLinearRegressionWithSGD

# Create StreamingContext
ssc = StreamingContext(sc, batchDuration=1)

# Define data streams
training_stream = ssc.textFileStream("training/data/path")
testing_stream = ssc.textFileStream("testing/data/path")

# Parse streams into LabeledPoint objects
def parse_point(line):
    values = [float(x) for x in line.strip().split(',')]
    return LabeledPoint(values[0], values[1:])

parsed_training = training_stream.map(parse_point)
parsed_testing = testing_stream.map(parse_point)

# Initialize model
num_features = 3
model = StreamingLinearRegressionWithSGD(initialWeights=[0.0] * num_features)

# Train and predict
model.trainOn(parsed_training)
predictions = model.predictOnValues(parsed_testing.map(lambda lp: (lp.label, lp.features)))

# Print predictions
predictions.pprint()

# Start streaming
ssc.start()
ssc.awaitTermination()
```

### Model Evaluation

Evaluate model performance:

```python
from pyspark.ml.evaluation import BinaryClassificationEvaluator, MulticlassClassificationEvaluator, RegressionEvaluator

# Binary classification
binary_evaluator = BinaryClassificationEvaluator(
    labelCol="label",
    rawPredictionCol="rawPrediction",
    metricName="areaUnderROC"
)
auc = binary_evaluator.evaluate(predictions)
print(f"AUC: {auc}")

# Multiclass classification
multi_evaluator = MulticlassClassificationEvaluator(
    labelCol="label",
    predictionCol="prediction",
    metricName="accuracy"
)
accuracy = multi_evaluator.evaluate(predictions)
print(f"Accuracy: {accuracy}")

# Regression
regression_evaluator = RegressionEvaluator(
    labelCol="label",
    predictionCol="prediction",
    metricName="rmse"
)
rmse = regression_evaluator.evaluate(predictions)
print(f"RMSE: {rmse}")
```

### Hyperparameter Tuning

Optimize model parameters with cross-validation:

```python
from pyspark.ml.tuning import ParamGridBuilder, CrossValidator
from pyspark.ml.classification import RandomForestClassifier
from pyspark.ml.evaluation import MulticlassClassificationEvaluator

# Define model
rf = RandomForestClassifier(labelCol="label", featuresCol="features")

# Build parameter grid
param_grid = ParamGridBuilder() \
    .addGrid(rf.numTrees, [10, 20, 50]) \
    .addGrid(rf.maxDepth, [5, 10, 15]) \
    .addGrid(rf.minInstancesPerNode, [1, 5, 10]) \
    .build()

# Define evaluator
evaluator = MulticlassClassificationEvaluator(metricName="accuracy")

# Cross-validation
cv = CrossValidator(
    estimator=rf,
    estimatorParamMaps=param_grid,
    evaluator=evaluator,
    numFolds=5,
    parallelism=4
)

# Train
cv_model = cv.fit(train_df)

# Best model
best_model = cv_model.bestModel
print(f"Best numTrees: {best_model.getNumTrees}")
print(f"Best maxDepth: {best_model.getMaxDepth()}")

# Evaluate on test set
predictions = cv_model.transform(test_df)
accuracy = evaluator.evaluate(predictions)
print(f"Test Accuracy: {accuracy}")
```

### Distributed Matrix Operations

MLlib provides distributed matrix representations:

```python
from pyspark.mllib.linalg.distributed import RowMatrix, IndexedRowMatrix, CoordinateMatrix
from pyspark.mllib.linalg import Vectors

# RowMatrix: Distributed matrix without row indices
rows = sc.parallelize([
    Vectors.dense([1.0, 2.0, 3.0]),
    Vectors.dense([4.0, 5.0, 6.0]),
    Vectors.dense([7.0, 8.0, 9.0])
])
row_matrix = RowMatrix(rows)

# Compute statistics
print(f"Rows: {row_matrix.numRows()}")
print(f"Cols: {row_matrix.numCols()}")
print(f"Column means: {row_matrix.computeColumnSummaryStatistics().mean()}")

# IndexedRowMatrix: Matrix with row indices
from pyspark.mllib.linalg.distributed import IndexedRow
indexed_rows = sc.parallelize([
    IndexedRow(0, Vectors.dense([1.0, 2.0, 3.0])),
    IndexedRow(1, Vectors.dense([4.0, 5.0, 6.0]))
])
indexed_matrix = IndexedRowMatrix(indexed_rows)

# CoordinateMatrix: Sparse matrix using (row, col, value) entries
from pyspark.mllib.linalg.distributed import MatrixEntry
entries = sc.parallelize([
    MatrixEntry(0, 0, 1.0),
    MatrixEntry(0, 2, 3.0),
    MatrixEntry(1, 1, 5.0)
])
coord_matrix = CoordinateMatrix(entries)
```

### Stratified Sampling

Sample data while preserving class distribution:

```python
# Scala/Java approach
data = [("a", 1), ("b", 2), ("a", 3), ("b", 4), ("a", 5), ("c", 6)]
rdd = sc.parallelize(data)

# Define sampling fractions per key
fractions = {"a": 0.5, "b": 0.5, "c": 0.5}

# Approximate sample (faster, one pass)
sampled_rdd = rdd.sampleByKey(withReplacement=False, fractions=fractions)

# Exact sample (slower, guaranteed exact counts)
exact_sampled = rdd.sampleByKeyExact(withReplacement=False, fractions=fractions)

print(sampled_rdd.collect())
```

## Performance Tuning

### Memory Management

**Memory Breakdown:**
- **Execution Memory**: Used for shuffles, joins, sorts, aggregations
- **Storage Memory**: Used for caching and broadcast variables
- **User Memory**: Used for user data structures and UDFs
- **Reserved Memory**: Reserved for Spark internal operations

**Configuration:**
```python
spark = SparkSession.builder \
    .appName("MemoryTuning") \
    .config("spark.executor.memory", "4g") \
    .config("spark.driver.memory", "2g") \
    .config("spark.memory.fraction", "0.6")  # Fraction for execution + storage \
    .config("spark.memory.storageFraction", "0.5")  # Fraction of above for storage \
    .getOrCreate()
```

**Memory Best Practices:**
- Monitor memory usage via Spark UI
- Use appropriate storage levels for caching
- Avoid collecting large datasets to driver
- Increase executor memory for memory-intensive operations
- Use kryo serialization for better memory efficiency

### Shuffle Optimization

Shuffles are expensive operations - minimize them:

**Causes of Shuffles:**
- groupByKey, reduceByKey, aggregateByKey
- join, cogroup
- repartition, coalesce (with increase)
- distinct, intersection
- sortByKey

**Optimization Strategies:**
```python
# 1. Use reduceByKey instead of groupByKey
# Bad: groupByKey shuffles all data
word_pairs.groupByKey().mapValues(sum)

# Good: reduceByKey combines locally before shuffle
word_pairs.reduceByKey(lambda a, b: a + b)

# 2. Broadcast small tables in joins
from pyspark.sql.functions import broadcast
large_df.join(broadcast(small_df), "key")

# 3. Partition data appropriately
df.repartition(200, "user_id")  # Partition by key for subsequent aggregations

# 4. Coalesce instead of repartition when reducing partitions
df.coalesce(10)  # No shuffle, just merge partitions

# 5. Tune shuffle partitions
spark.conf.set("spark.sql.shuffle.partitions", 200)  # Default is 200
```

**Shuffle Configuration:**
```python
spark = SparkSession.builder \
    .config("spark.sql.shuffle.partitions", 200) \
    .config("spark.default.parallelism", 200) \
    .config("spark.sql.adaptive.enabled", "true")  # Enable AQE \
    .config("spark.sql.adaptive.coalescePartitions.enabled", "true") \
    .getOrCreate()
```

### Partitioning Strategies

**Partition Count Guidelines:**
- Too few: Underutilized cluster, OOM errors
- Too many: Task scheduling overhead
- Sweet spot: 2-4x number of CPU cores
- For large shuffles: 100-200+ partitions

**Partition by Column:**
```python
# Partition writes by date for easy filtering
df.write.partitionBy("date", "country").parquet("output")

# Read with partition pruning (only reads relevant partitions)
spark.read.parquet("output").filter(col("date") == "2025-01-15").show()
```

**Custom Partitioning:**
```python
from pyspark.rdd import portable_hash

# Custom partitioner for RDD
def custom_partitioner(key):
    return portable_hash(key) % 100

rdd.partitionBy(100, custom_partitioner)
```

### Caching Strategies

**When to Cache:**
```python
# Iterative algorithms (ML)
training_data.cache()
for i in range(num_iterations):
    model = train_model(training_data)

# Multiple aggregations on same data
base_df.cache()
result1 = base_df.groupBy("country").count()
result2 = base_df.groupBy("city").avg("sales")

# Interactive analysis
df.cache()
df.filter(condition1).show()
df.filter(condition2).show()
df.groupBy("category").count().show()
```

**Storage Levels:**
```python
from pyspark import StorageLevel

# Memory only (fastest, but may lose data)
df.persist(StorageLevel.MEMORY_ONLY)

# Memory and disk (spill to disk if needed)
df.persist(StorageLevel.MEMORY_AND_DISK)

# Serialized in memory (more compact, slower access)
df.persist(StorageLevel.MEMORY_ONLY_SER)

# Disk only (slowest, but always available)
df.persist(StorageLevel.DISK_ONLY)

# Replicated (fault tolerance)
df.persist(StorageLevel.MEMORY_AND_DISK_2)  # 2 replicas
```

### Broadcast Joins

Optimize joins with small tables:

```python
from pyspark.sql.functions import broadcast

# Automatic broadcast (tables < spark.sql.autoBroadcastJoinThreshold)
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", 10 * 1024 * 1024)  # 10 MB

# Explicit broadcast hint
large_df.join(broadcast(small_df), "key")

# Benefits:
# - No shuffle of large table
# - Small table sent to all executors once
# - Much faster for small dimension tables
```

### Adaptive Query Execution (AQE)

Enable runtime query optimization:

```python
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")

# AQE Benefits:
# - Dynamically coalesce partitions after shuffle
# - Handle skewed joins by splitting large partitions
# - Optimize join strategy at runtime
```

### Data Format Selection

**Performance Comparison:**
1. **Parquet** (Best for analytics): Columnar, compressed, fast queries
2. **ORC** (Best for Hive): Similar to Parquet, slightly better compression
3. **Avro** (Best for row-oriented): Good for write-heavy workloads
4. **JSON** (Slowest): Human-readable but inefficient
5. **CSV** (Legacy): Compatible but slow and no schema

**Recommendation:**
- Use Parquet for most analytics workloads
- Enable compression (snappy, gzip, lzo)
- Partition by commonly filtered columns
- Use columnar formats for read-heavy workloads

### Catalyst Optimizer

Understand query optimization:

```python
# View physical plan
df.explain(mode="extended")

# Optimizations include:
# - Predicate pushdown: Push filters to data source
# - Column pruning: Read only required columns
# - Constant folding: Evaluate constants at compile time
# - Join reordering: Optimize join order
# - Partition pruning: Skip irrelevant partitions
```

## Production Deployment

### Cluster Managers

**Standalone:**
- Simple, built-in cluster manager
- Easy setup for development and small clusters
- No resource sharing with other frameworks

```bash
# Start master
$SPARK_HOME/sbin/start-master.sh

# Start workers
$SPARK_HOME/sbin/start-worker.sh spark://master:7077

# Submit application
spark-submit --master spark://master:7077 app.py
```

**YARN:**
- Hadoop's resource manager
- Share cluster resources with MapReduce, Hive, etc.
- Two modes: cluster (driver on YARN) and client (driver on local machine)

```bash
# Cluster mode (driver runs on YARN)
spark-submit --master yarn --deploy-mode cluster app.py

# Client mode (driver runs locally)
spark-submit --master yarn --deploy-mode client app.py
```

**Kubernetes:**
- Modern container orchestration
- Dynamic resource allocation
- Cloud-native deployments

```bash
spark-submit \
  --master k8s://https://k8s-master:443 \
  --deploy-mode cluster \
  --name spark-app \
  --conf spark.executor.instances=5 \
  --conf spark.kubernetes.container.image=spark:latest \
  app.py
```

**Mesos:**
- General-purpose cluster manager
- Fine-grained or coarse-grained resource sharing

### Application Submission

**Basic spark-submit:**
```bash
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --driver-memory 4g \
  --executor-memory 8g \
  --executor-cores 4 \
  --num-executors 10 \
  --conf spark.sql.shuffle.partitions=200 \
  --py-files dependencies.zip \
  --files config.json \
  application.py
```

**Configuration Options:**
- `--master`: Cluster manager URL
- `--deploy-mode`: Where to run driver (client or cluster)
- `--driver-memory`: Memory for driver process
- `--executor-memory`: Memory per executor
- `--executor-cores`: Cores per executor
- `--num-executors`: Number of executors
- `--conf`: Spark configuration properties
- `--py-files`: Python dependencies
- `--files`: Additional files to distribute

### Resource Allocation

**General Guidelines:**
- **Driver Memory**: 1-4 GB (unless collecting large results)
- **Executor Memory**: 4-16 GB per executor
- **Executor Cores**: 4-5 cores per executor (diminishing returns beyond 5)
- **Number of Executors**: Fill cluster capacity, leave resources for OS/other services
- **Parallelism**: 2-4x total cores

**Example Calculations:**
```
Cluster: 10 nodes, 32 cores each, 128 GB RAM each

Option 1: Many small executors
  - 30 executors (3 per node)
  - 10 cores per executor
  - 40 GB memory per executor
  - Total: 300 cores

Option 2: Fewer large executors (RECOMMENDED)
  - 50 executors (5 per node)
  - 5 cores per executor
  - 24 GB memory per executor
  - Total: 250 cores
```

### Dynamic Allocation

Automatically scale executors based on workload:

```python
spark = SparkSession.builder \
    .appName("DynamicAllocation") \
    .config("spark.dynamicAllocation.enabled", "true") \
    .config("spark.dynamicAllocation.minExecutors", 2) \
    .config("spark.dynamicAllocation.maxExecutors", 100) \
    .config("spark.dynamicAllocation.initialExecutors", 10) \
    .config("spark.dynamicAllocation.executorIdleTimeout", "60s") \
    .getOrCreate()
```

**Benefits:**
- Better resource utilization
- Automatic scaling for varying workloads
- Reduced costs in cloud environments

### Monitoring and Logging

**Spark UI:**
- Web UI at http://driver:4040
- Stages, tasks, storage, environment, executors
- SQL query plans and execution details
- Identify bottlenecks and performance issues

**History Server:**
```bash
# Start history server
$SPARK_HOME/sbin/start-history-server.sh

# Configure event logging
spark.conf.set("spark.eventLog.enabled", "true")
spark.conf.set("spark.eventLog.dir", "hdfs://namenode/spark-logs")
```

**Metrics:**
```python
# Enable metrics collection
spark.conf.set("spark.metrics.conf.*.sink.console.class", "org.apache.spark.metrics.sink.ConsoleSink")
spark.conf.set("spark.metrics.conf.*.sink.console.period", 10)
```

**Logging:**
```python
# Configure log level
spark.sparkContext.setLogLevel("WARN")  # ERROR, WARN, INFO, DEBUG

# Custom logging
import logging
logger = logging.getLogger(__name__)
logger.info("Custom log message")
```

### Fault Tolerance

**Automatic Recovery:**
- Task failures: Automatically retry failed tasks
- Executor failures: Reschedule tasks on other executors
- Driver failures: Restore from checkpoint (streaming)
- Node failures: Recompute lost partitions from lineage

**Checkpointing:**
```python
# Set checkpoint directory
spark.sparkContext.setCheckpointDir("hdfs://namenode/checkpoints")

# Checkpoint RDD (breaks lineage for very long chains)
rdd.checkpoint()

# Streaming checkpoint (required for production)
query = streaming_df.writeStream \
    .option("checkpointLocation", "hdfs://namenode/streaming-checkpoint") \
    .start()
```

**Speculative Execution:**
```python
# Enable speculative execution for slow tasks
spark.conf.set("spark.speculation", "true")
spark.conf.set("spark.speculation.multiplier", 1.5)
spark.conf.set("spark.speculation.quantile", 0.75)
```

### Data Locality

Optimize data placement for performance:

**Locality Levels:**
1. **PROCESS_LOCAL**: Data in same JVM as task (fastest)
2. **NODE_LOCAL**: Data on same node, different process
3. **RACK_LOCAL**: Data on same rack
4. **ANY**: Data on different rack (slowest)

**Improve Locality:**
```python
# Increase locality wait time
spark.conf.set("spark.locality.wait", "10s")
spark.conf.set("spark.locality.wait.node", "5s")
spark.conf.set("spark.locality.wait.rack", "3s")

# Partition data to match cluster topology
df.repartition(num_nodes * cores_per_node)
```

## Best Practices

### Code Organization

1. **Modular Design**: Separate data loading, transformation, and output logic
2. **Configuration Management**: Externalize configuration (use config files)
3. **Error Handling**: Implement robust error handling and logging
4. **Testing**: Unit test transformations, integration test pipelines
5. **Documentation**: Document complex transformations and business logic

### Performance

1. **Avoid Shuffles**: Use reduceByKey instead of groupByKey
2. **Cache Wisely**: Only cache data reused multiple times
3. **Broadcast Small Tables**: Use broadcast joins for small reference data
4. **Partition Appropriately**: 2-4x CPU cores, partition by frequently filtered columns
5. **Use Parquet**: Columnar format for analytical workloads
6. **Enable AQE**: Leverage adaptive query execution for runtime optimization
7. **Tune Memory**: Balance executor memory and cores
8. **Monitor**: Use Spark UI to identify bottlenecks

### Development Workflow

1. **Start Small**: Develop with sample data locally
2. **Profile Early**: Monitor performance from the start
3. **Iterate**: Optimize incrementally based on metrics
4. **Test at Scale**: Validate with production-sized data before deployment
5. **Version Control**: Track code, configurations, and schemas

### Data Quality

1. **Schema Validation**: Enforce schemas on read/write
2. **Null Handling**: Explicitly handle null values
3. **Data Validation**: Check for expected ranges, formats, constraints
4. **Deduplication**: Remove duplicates based on business logic
5. **Audit Logging**: Track data lineage and transformations

### Security

1. **Authentication**: Enable Kerberos for YARN/HDFS
2. **Authorization**: Use ACLs for data access control
3. **Encryption**: Encrypt data at rest and in transit
4. **Secrets Management**: Use secure credential providers
5. **Audit Trails**: Log data access and modifications

### Cost Optimization

1. **Right-Size Resources**: Don't over-provision executors
2. **Dynamic Allocation**: Scale executors based on workload
3. **Spot Instances**: Use spot/preemptible instances in cloud
4. **Data Compression**: Use efficient formats (Parquet, ORC)
5. **Partitioning**: Prune unnecessary data reads
6. **Auto-Shutdown**: Terminate idle clusters

## Common Patterns

### ETL Pipeline Pattern

```python
def etl_pipeline(spark, input_path, output_path):
    # Extract
    raw_df = spark.read.parquet(input_path)

    # Transform
    cleaned_df = raw_df \
        .dropDuplicates(["id"]) \
        .filter(col("value").isNotNull()) \
        .withColumn("processed_date", current_date())

    # Enrich
    enriched_df = cleaned_df.join(broadcast(reference_df), "key")

    # Aggregate
    aggregated_df = enriched_df \
        .groupBy("category", "date") \
        .agg(
            count("*").alias("count"),
            sum("amount").alias("total_amount"),
            avg("value").alias("avg_value")
        )

    # Load
    aggregated_df.write \
        .partitionBy("date") \
        .mode("overwrite") \
        .parquet(output_path)
```

### Incremental Processing Pattern

```python
def incremental_process(spark, input_path, output_path, checkpoint_path):
    # Read last processed timestamp
    last_timestamp = read_checkpoint(checkpoint_path)

    # Read new data
    new_data = spark.read.parquet(input_path) \
        .filter(col("timestamp") > last_timestamp)

    # Process
    processed = transform(new_data)

    # Write
    processed.write.mode("append").parquet(output_path)

    # Update checkpoint
    max_timestamp = new_data.agg(max("timestamp")).collect()[0][0]
    write_checkpoint(checkpoint_path, max_timestamp)
```

### Slowly Changing Dimension (SCD) Pattern

```python
def scd_type2_upsert(spark, dimension_df, updates_df):
    # Mark existing records as inactive if updated
    inactive_records = dimension_df \
        .join(updates_df, "business_key") \
        .select(
            dimension_df["*"],
            lit(False).alias("is_active"),
            current_date().alias("end_date")
        )

    # Add new records
    new_records = updates_df \
        .withColumn("is_active", lit(True)) \
        .withColumn("start_date", current_date()) \
        .withColumn("end_date", lit(None))

    # Union unchanged, inactive, and new records
    result = dimension_df \
        .join(updates_df, "business_key", "left_anti") \
        .union(inactive_records) \
        .union(new_records)

    return result
```

### Window Analytics Pattern

```python
def calculate_running_metrics(df):
    from pyspark.sql.window import Window
    from pyspark.sql.functions import row_number, lag, sum, avg

    # Define window
    window_spec = Window.partitionBy("user_id").orderBy("timestamp")

    # Calculate metrics
    result = df \
        .withColumn("row_num", row_number().over(window_spec)) \
        .withColumn("prev_value", lag("value", 1).over(window_spec)) \
        .withColumn("running_total", sum("value").over(window_spec.rowsBetween(Window.unboundedPreceding, Window.currentRow))) \
        .withColumn("moving_avg", avg("value").over(window_spec.rowsBetween(-2, 0)))

    return result
```

## Troubleshooting

### Out of Memory Errors

**Symptoms:**
- `java.lang.OutOfMemoryError`
- Executor failures
- Slow garbage collection

**Solutions:**
```python
# Increase executor memory
spark.conf.set("spark.executor.memory", "8g")

# Increase driver memory (if collecting data)
spark.conf.set("spark.driver.memory", "4g")

# Reduce memory pressure
df.persist(StorageLevel.MEMORY_AND_DISK)  # Spill to disk
df.coalesce(100)  # Reduce partition count
spark.conf.set("spark.sql.shuffle.partitions", 400)  # Increase shuffle partitions

# Avoid collect() on large datasets
# Use take() or limit() instead
df.take(100)
```

### Shuffle Performance Issues

**Symptoms:**
- Long shuffle read/write times
- Skewed partition sizes
- Task stragglers

**Solutions:**
```python
# Increase shuffle partitions
spark.conf.set("spark.sql.shuffle.partitions", 400)

# Handle skew with salting
df_salted = df.withColumn("salt", (rand() * 10).cast("int"))
result = df_salted.groupBy("key", "salt").agg(...)

# Use broadcast for small tables
large_df.join(broadcast(small_df), "key")

# Enable AQE for automatic optimization
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
```

### Streaming Job Failures

**Symptoms:**
- Streaming query stopped
- Checkpoint corruption
- Processing lag increasing

**Solutions:**
```python
# Increase executor memory for stateful operations
spark.conf.set("spark.executor.memory", "8g")

# Tune watermark for late data
.withWatermark("timestamp", "15 minutes")

# Increase trigger interval to reduce micro-batch overhead
.trigger(processingTime="30 seconds")

# Monitor lag and adjust parallelism
spark.conf.set("spark.sql.shuffle.partitions", 200)

# Recover from checkpoint corruption
# Delete checkpoint directory and restart (data loss possible)
# Or implement custom state recovery logic
```

### Data Skew

**Symptoms:**
- Few tasks take much longer than others
- Unbalanced partition sizes
- Executor OOM errors

**Solutions:**
```python
# 1. Salting technique (add random prefix to keys)
from pyspark.sql.functions import concat, lit, rand

df_salted = df.withColumn("salted_key", concat(col("key"), lit("_"), (rand() * 10).cast("int")))
result = df_salted.groupBy("salted_key").agg(...)

# 2. Repartition by skewed column
df.repartition(200, "skewed_column")

# 3. Isolate skewed keys
skewed_keys = df.groupBy("key").count().filter(col("count") > threshold).select("key")
skewed_df = df.join(broadcast(skewed_keys), "key")
normal_df = df.join(broadcast(skewed_keys), "key", "left_anti")

# Process separately
skewed_result = process_with_salting(skewed_df)
normal_result = process_normally(normal_df)
final = skewed_result.union(normal_result)
```

## Context7 Code Integration

This skill integrates real-world code examples from Apache Spark's official repository. All code snippets in the EXAMPLES.md file are sourced from Context7's Apache Spark library documentation, ensuring production-ready patterns and best practices.

## Version and Compatibility

- **Apache Spark Version**: 3.x (compatible with 2.4+)
- **Python**: 3.7+
- **Scala**: 2.12+
- **Java**: 8+
- **R**: 3.5+

## References

- Official Documentation: https://spark.apache.org/docs/latest/
- API Reference: https://spark.apache.org/docs/latest/api.html
- GitHub Repository: https://github.com/apache/spark
- Databricks Blog: https://databricks.com/blog
- Context7 Library: /apache/spark

---

**Skill Version**: 1.0.0
**Last Updated**: October 2025
**Skill Category**: Big Data, Distributed Computing, Data Engineering, Machine Learning
**Context7 Integration**: /apache/spark with 8000 tokens of documentation
