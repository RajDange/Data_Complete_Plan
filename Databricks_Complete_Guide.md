# 🧱 DATABRICKS MASTERY - Complete Learning Guide

## 📋 Table of Contents
1. [Databricks Fundamentals](#fundamentals)
2. [Lakehouse Architecture](#lakehouse)
3. [Apache Spark Deep Dive](#spark)
4. [Delta Lake](#delta-lake)
5. [Databricks Notebooks](#notebooks)
6. [Data Engineering Workflows](#workflows)
7. [Databricks SQL](#sql)
8. [MLflow & ML](#mlflow)
9. [Performance Optimization](#optimization)
10. [Unity Catalog](#unity-catalog)
11. [Production Best Practices](#production)
12. [Projects & Interview Prep](#projects)

---

## 🎯 LEVEL 1: DATABRICKS FUNDAMENTALS

### 1.1 What is Databricks?

**Databricks = Unified Analytics Platform**

```
┌─────────────────────────────────────────────────────────────────┐
│                        DATABRICKS                                │
├─────────────────────────────────────────────────────────────────┤
│  Data Engineering  │  Data Science  │  ML Engineering  │  SQL   │
├─────────────────────────────────────────────────────────────────┤
│              Apache Spark (Processing Engine)                    │
├─────────────────────────────────────────────────────────────────┤
│                    Delta Lake (Storage)                          │
├─────────────────────────────────────────────────────────────────┤
│           Cloud Storage (S3, ADLS, GCS)                         │
└─────────────────────────────────────────────────────────────────┘
```

**Key Components:**
- **Workspace**: Collaborative environment (notebooks, dashboards, jobs)
- **Compute**: Clusters for running workloads
- **Delta Lake**: ACID transactions on data lakes
- **MLflow**: ML lifecycle management
- **Unity Catalog**: Unified governance
- **Databricks SQL**: BI and analytics

---

### 1.2 Databricks vs Competitors

| Feature | Databricks | Snowflake | AWS EMR |
|---------|------------|-----------|---------|
| **Primary Use** | Unified analytics (ETL + ML + BI) | Data warehouse (SQL) | DIY Spark |
| **Engine** | Apache Spark | Proprietary | Apache Spark |
| **ML Support** | Native (MLflow) | Limited | Manual setup |
| **Storage Format** | Delta Lake (open) | Proprietary | Your choice |
| **Streaming** | Structured Streaming | Limited | Kafka/Kinesis |
| **Notebooks** | Built-in | None | Separate (Jupyter) |
| **Best For** | Big data + ML + ETL | SQL analytics | Custom Spark needs |

**When to Choose Databricks:**
✓ Big data processing (TB-PB scale)
✓ Machine learning pipelines
✓ Streaming analytics
✓ Complex transformations
✓ Python/Scala heavy workloads
✓ Need both ETL and ML

**When to Choose Snowflake:**
✓ SQL-focused analytics
✓ BI and reporting primary use
✓ Simple transformations
✓ Cost-effective for pure DWH needs

---

### 1.3 Lakehouse Architecture

**What is a Lakehouse?**
- Combines **Data Lake** (cheap storage, any format) + **Data Warehouse** (ACID, performance)
- Best of both worlds!

```
Traditional Approach:
Data Lake (S3/ADLS) → ETL → Data Warehouse (Snowflake/Redshift) → BI Tools
  ↓ Problems:
  - Data duplication
  - Expensive ETL
  - Data staleness
  - Complexity

Lakehouse Approach:
Data Lake (S3/ADLS) + Delta Lake → BI Tools + ML Tools
  ↓ Benefits:
  - Single source of truth
  - ACID transactions
  - Time travel
  - Unified governance
```

---

### 1.4 Getting Started - Key Concepts

**Workspace Structure:**
```
Workspace
├── Repos (Git integration)
├── Notebooks (SQL, Python, Scala, R)
├── Workflows (Jobs, Pipelines)
├── Data (Tables, Databases)
├── Compute (Clusters)
└── SQL (Queries, Dashboards)
```

**First Steps:**

```python
# In a Databricks notebook

# 1. Check your current context
print(f"Current Database: {spark.catalog.currentDatabase()}")
print(f"Available Databases: {[db.name for db in spark.catalog.listDatabases()]}")

# 2. Create a database
spark.sql("CREATE DATABASE IF NOT EXISTS my_first_db")
spark.sql("USE my_first_db")

# 3. Create a simple DataFrame
from pyspark.sql import Row

data = [
    Row(id=1, name="Alice", age=25),
    Row(id=2, name="Bob", age=30),
    Row(id=3, name="Charlie", age=35)
]

df = spark.createDataFrame(data)
df.show()

# 4. Save as Delta table
df.write.format("delta").mode("overwrite").saveAsTable("users")

# 5. Query the table
spark.sql("SELECT * FROM users").show()

# 6. Display for better visualization
display(df)
```

---

## 🏗️ LEVEL 2: LAKEHOUSE ARCHITECTURE

### 2.1 Medallion Architecture (Bronze-Silver-Gold)

**Industry Standard Pattern:**

```
┌─────────────────────────────────────────────────────────────┐
│  BRONZE LAYER (Raw Data)                                     │
│  - Raw ingestion from sources                                │
│  - No transformations                                        │
│  - Exactly as received                                       │
│  - Full history preserved                                    │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  SILVER LAYER (Cleaned & Conformed)                         │
│  - Data quality checks                                       │
│  - Standardized formats                                      │
│  - Deduplication                                             │
│  - Type casting                                              │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  GOLD LAYER (Business-Level Aggregates)                     │
│  - Feature tables                                            │
│  - Aggregated metrics                                        │
│  - Business KPIs                                             │
│  - Ready for BI/ML                                           │
└─────────────────────────────────────────────────────────────┘
```

**Implementation Example:**

```python
# BRONZE: Ingest raw data
bronze_df = (spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "json")
    .option("cloudFiles.schemaLocation", "/mnt/schema/bronze")
    .load("/mnt/raw/events/")
)

# Write to Bronze table
(bronze_df.writeStream
    .format("delta")
    .option("checkpointLocation", "/mnt/checkpoints/bronze")
    .trigger(availableNow=True)
    .table("bronze.events")
)

# SILVER: Clean and conform
silver_df = spark.read.table("bronze.events")

from pyspark.sql.functions import col, to_timestamp, current_timestamp

cleaned_df = (silver_df
    .withColumn("event_timestamp", to_timestamp(col("timestamp")))
    .withColumn("processed_at", current_timestamp())
    .filter(col("user_id").isNotNull())  # Data quality
    .dropDuplicates(["event_id"])  # Deduplication
)

cleaned_df.write.format("delta").mode("overwrite").saveAsTable("silver.events_cleaned")

# GOLD: Business aggregates
gold_df = spark.sql("""
    SELECT 
        DATE(event_timestamp) as event_date,
        event_type,
        COUNT(*) as event_count,
        COUNT(DISTINCT user_id) as unique_users
    FROM silver.events_cleaned
    GROUP BY DATE(event_timestamp), event_type
""")

gold_df.write.format("delta").mode("overwrite").saveAsTable("gold.daily_event_metrics")
```

---

### 2.2 Storage Layers in Databricks

**Mount Points vs Unity Catalog:**

**Option 1: Mount External Storage (Legacy)**
```python
# Mount Azure Blob Storage
dbutils.fs.mount(
    source = "wasbs://container@storageaccount.blob.core.windows.net",
    mount_point = "/mnt/data",
    extra_configs = {
        "fs.azure.account.key.storageaccount.blob.core.windows.net": dbutils.secrets.get("scope", "key")
    }
)

# Access mounted data
df = spark.read.parquet("/mnt/data/sales/")

# Unmount
dbutils.fs.unmount("/mnt/data")
```

**Option 2: Unity Catalog (Modern, Recommended)**
```python
# No mounting needed!
# Access directly with three-level namespace

# Read from external location
df = spark.read.parquet("s3://my-bucket/data/sales/")

# Or use external tables
spark.sql("""
    CREATE EXTERNAL TABLE sales
    LOCATION 's3://my-bucket/data/sales/'
""")

# Query
df = spark.sql("SELECT * FROM catalog.schema.sales")
```

---

## ⚡ LEVEL 3: APACHE SPARK DEEP DIVE

### 3.1 Spark Architecture

**Key Concepts:**

```
Driver Program (Your Notebook)
    ↓ Creates
SparkSession
    ↓ Manages
Cluster Manager (Databricks manages this)
    ↓ Allocates
Executors (Worker Nodes)
    ↓ Process
Partitions of Data
```

**Transformations vs Actions:**

```python
# TRANSFORMATIONS (Lazy - not executed immediately)
df = spark.read.parquet("/data/sales")  # NOT executed yet
filtered = df.filter(col("amount") > 100)  # NOT executed yet
aggregated = filtered.groupBy("category").sum("amount")  # NOT executed yet

# ACTIONS (Trigger execution)
aggregated.show()  # NOW everything executes
aggregated.count()  # Executes again
aggregated.collect()  # Executes again

# Cache to avoid re-execution
aggregated.cache()  # Or .persist()
aggregated.show()  # Executes and caches
aggregated.count()  # Uses cached data (fast!)
```

---

### 3.2 DataFrame API (Essential!)

**Creating DataFrames:**

```python
from pyspark.sql import Row
from pyspark.sql.functions import *

# From data
data = [
    Row(id=1, name="Alice", dept="Sales", salary=75000),
    Row(id=2, name="Bob", dept="Engineering", salary=85000),
    Row(id=3, name="Charlie", dept="Sales", salary=70000)
]
df = spark.createDataFrame(data)

# From file
df = spark.read.format("delta").load("/path/to/table")
df = spark.read.table("catalog.schema.table")
df = spark.read.parquet("/path/to/parquet")
df = spark.read.csv("/path/to/csv", header=True, inferSchema=True)
df = spark.read.json("/path/to/json")

# With schema
from pyspark.sql.types import *

schema = StructType([
    StructField("id", IntegerType(), False),
    StructField("name", StringType(), True),
    StructField("age", IntegerType(), True)
])

df = spark.read.schema(schema).csv("/path/to/csv")
```

---

### 3.3 Common DataFrame Operations

**Selecting & Filtering:**

```python
# Select columns
df.select("name", "salary").show()
df.select(col("name"), col("salary")).show()
df.select(df.name, df.salary).show()

# Filter rows
df.filter(col("salary") > 80000).show()
df.filter("salary > 80000").show()
df.where(col("dept") == "Sales").show()

# Multiple conditions
df.filter((col("salary") > 80000) & (col("dept") == "Engineering")).show()
df.filter((col("salary") > 100000) | (col("dept") == "Sales")).show()

# Add/rename columns
df_new = df.withColumn("bonus", col("salary") * 0.1)
df_renamed = df.withColumnRenamed("dept", "department")

# Drop columns
df_dropped = df.drop("salary", "bonus")

# Distinct
df.select("dept").distinct().show()
```

---

### 3.4 Aggregations

```python
from pyspark.sql.functions import *

# Group by
df.groupBy("dept").count().show()

df.groupBy("dept").agg(
    count("*").alias("employee_count"),
    avg("salary").alias("avg_salary"),
    max("salary").alias("max_salary"),
    min("salary").alias("min_salary")
).show()

# Multiple grouping columns
df.groupBy("dept", "location").agg(
    sum("salary").alias("total_payroll")
).show()

# Window functions
from pyspark.sql.window import Window

window_spec = Window.partitionBy("dept").orderBy(col("salary").desc())

df_with_rank = df.withColumn("rank", row_number().over(window_spec))
df_with_rank.show()

# Running totals
window_running = Window.partitionBy("dept").orderBy("id").rowsBetween(Window.unboundedPreceding, Window.currentRow)

df_with_running = df.withColumn(
    "running_total", 
    sum("salary").over(window_running)
)
df_with_running.show()
```

---

### 3.5 Joins

```python
# Sample data
employees = spark.createDataFrame([
    (1, "Alice", 101),
    (2, "Bob", 102),
    (3, "Charlie", 101)
], ["emp_id", "name", "dept_id"])

departments = spark.createDataFrame([
    (101, "Sales"),
    (102, "Engineering"),
    (103, "HR")
], ["dept_id", "dept_name"])

# Inner join
result = employees.join(departments, "dept_id", "inner")
result.show()

# Left join
result = employees.join(departments, "dept_id", "left")
result.show()

# Multiple conditions
result = employees.join(
    departments,
    (employees.dept_id == departments.dept_id) & (employees.emp_id < 3),
    "inner"
)

# Broadcast join (for small tables)
from pyspark.sql.functions import broadcast
result = employees.join(broadcast(departments), "dept_id")
```

---

### 3.6 Working with Dates & Strings

```python
from pyspark.sql.functions import *

# Date functions
df_dates = spark.createDataFrame([
    ("2024-01-15", "2024-02-20"),
    ("2024-03-10", "2024-04-05")
], ["start_date", "end_date"])

df_dates = df_dates.withColumn("start", to_date("start_date"))
df_dates = df_dates.withColumn("end", to_date("end_date"))

# Date calculations
df_dates = df_dates.withColumn("days_diff", datediff("end", "start"))
df_dates = df_dates.withColumn("year", year("start"))
df_dates = df_dates.withColumn("month", month("start"))
df_dates = df_dates.withColumn("next_month", add_months("start", 1))

df_dates.show()

# String functions
df_strings = spark.createDataFrame([
    ("  John Doe  ",),
    ("jane smith",),
    ("BOB JONES",)
], ["name"])

df_strings = df_strings.withColumn("trimmed", trim(col("name")))
df_strings = df_strings.withColumn("upper", upper(col("name")))
df_strings = df_strings.withColumn("lower", lower(col("name")))
df_strings = df_strings.withColumn("length", length(col("name")))
df_strings = df_strings.withColumn("first_word", split(col("name"), " ")[0])

df_strings.show(truncate=False)
```

---

### 3.7 UDFs (User-Defined Functions)

```python
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType, IntegerType

# Simple UDF
def categorize_salary(salary):
    if salary > 100000:
        return "High"
    elif salary > 70000:
        return "Medium"
    else:
        return "Low"

categorize_udf = udf(categorize_salary, StringType())

df_categorized = df.withColumn("salary_category", categorize_udf(col("salary")))
df_categorized.show()

# Pandas UDF (faster for vectorized operations)
from pyspark.sql.functions import pandas_udf
import pandas as pd

@pandas_udf(IntegerType())
def multiply_by_two(s: pd.Series) -> pd.Series:
    return s * 2

df_doubled = df.withColumn("salary_doubled", multiply_by_two(col("salary")))
df_doubled.show()
```

---

## 🔷 LEVEL 4: DELTA LAKE

### 4.1 What is Delta Lake?

**Delta Lake = ACID + Time Travel + Schema Evolution on Data Lake**

```
Parquet Files (Traditional)          Delta Lake
├─ No transactions                   ├─ ACID transactions ✓
├─ No schema enforcement             ├─ Schema enforcement ✓
├─ No time travel                    ├─ Time travel ✓
├─ Slow updates/deletes              ├─ Fast MERGE/UPDATE/DELETE ✓
├─ No change tracking                ├─ Change Data Capture ✓
└─ Manual optimization               └─ Auto-optimization ✓
```

---

### 4.2 Creating Delta Tables

```python
# Create Delta table from DataFrame
df = spark.read.csv("/data/sales.csv", header=True, inferSchema=True)

# Save as Delta
df.write.format("delta").mode("overwrite").save("/mnt/delta/sales")

# Or as managed table
df.write.format("delta").mode("overwrite").saveAsTable("sales")

# Create empty Delta table with schema
spark.sql("""
    CREATE TABLE IF NOT EXISTS customers (
        customer_id BIGINT,
        name STRING,
        email STRING,
        signup_date DATE,
        is_active BOOLEAN
    )
    USING DELTA
    LOCATION '/mnt/delta/customers'
""")

# Partitioned Delta table
df.write.format("delta") \
    .partitionBy("year", "month") \
    .mode("overwrite") \
    .saveAsTable("partitioned_sales")
```

---

### 4.3 Delta Lake Operations

**MERGE (UPSERT):**

```python
from delta.tables import DeltaTable

# Load Delta table
deltaTable = DeltaTable.forPath(spark, "/mnt/delta/customers")

# New/updated data
updates = spark.createDataFrame([
    (1, "Alice Updated", "alice@new.com", "2024-01-01", True),
    (4, "New Customer", "new@customer.com", "2024-02-01", True)
], ["customer_id", "name", "email", "signup_date", "is_active"])

# MERGE operation
deltaTable.alias("target").merge(
    updates.alias("source"),
    "target.customer_id = source.customer_id"
).whenMatchedUpdate(set = {
    "name": "source.name",
    "email": "source.email",
    "is_active": "source.is_active"
}).whenNotMatchedInsert(values = {
    "customer_id": "source.customer_id",
    "name": "source.name",
    "email": "source.email",
    "signup_date": "source.signup_date",
    "is_active": "source.is_active"
}).execute()

# SQL MERGE
spark.sql("""
    MERGE INTO customers AS target
    USING updates AS source
    ON target.customer_id = source.customer_id
    WHEN MATCHED THEN 
        UPDATE SET *
    WHEN NOT MATCHED THEN 
        INSERT *
""")
```

**UPDATE:**

```python
# Update using DataFrame API
deltaTable.update(
    condition = "is_active = false",
    set = {"is_active": "true"}
)

# SQL UPDATE
spark.sql("""
    UPDATE customers
    SET is_active = false
    WHERE signup_date < '2020-01-01'
""")
```

**DELETE:**

```python
# Delete using DataFrame API
deltaTable.delete("signup_date < '2019-01-01'")

# SQL DELETE
spark.sql("DELETE FROM customers WHERE is_active = false")
```

---

### 4.4 Time Travel

```python
# Read specific version
df_v1 = spark.read.format("delta").option("versionAsOf", 1).load("/mnt/delta/sales")

# Read as of timestamp
df_yesterday = spark.read.format("delta") \
    .option("timestampAsOf", "2024-02-01") \
    .load("/mnt/delta/sales")

# SQL syntax
df = spark.sql("SELECT * FROM sales VERSION AS OF 5")
df = spark.sql("SELECT * FROM sales TIMESTAMP AS OF '2024-02-01'")

# View history
deltaTable.history().show(20, truncate=False)

# Restore to previous version
spark.sql("RESTORE TABLE sales TO VERSION AS OF 5")
spark.sql("RESTORE TABLE sales TO TIMESTAMP AS OF '2024-02-01'")

# Vacuum old versions (free up storage)
# Only removes files older than retention period (default 7 days)
deltaTable.vacuum(168)  # 168 hours = 7 days

# WARNING: Vacuum removes old versions permanently!
```

---

### 4.5 Schema Evolution

```python
# Add new column (schema evolution)
new_data = spark.createDataFrame([
    (5, "Eve", "eve@example.com", "2024-03-01", True, "Premium")
], ["customer_id", "name", "email", "signup_date", "is_active", "tier"])

# Enable schema evolution
new_data.write.format("delta") \
    .mode("append") \
    .option("mergeSchema", "true") \
    .saveAsTable("customers")

# Schema enforcement (reject incompatible schemas)
spark.conf.set("spark.databricks.delta.schema.autoMerge.enabled", "false")

# View schema history
deltaTable.history().select("version", "timestamp", "operation", "operationParameters").show(truncate=False)
```

---

### 4.6 Optimization

**Optimize (Compaction):**

```python
# Compact small files into larger files
spark.sql("OPTIMIZE customers")

# With Z-ordering (co-locate related data)
spark.sql("OPTIMIZE customers ZORDER BY (customer_id)")

# Using DataFrame API
deltaTable.optimize().executeCompaction()
deltaTable.optimize().executeZOrderBy("customer_id", "signup_date")
```

**Auto-Optimize:**

```python
# Enable auto-optimize on table creation
spark.sql("""
    CREATE TABLE optimized_sales
    USING DELTA
    TBLPROPERTIES (
        delta.autoOptimize.optimizeWrite = true,
        delta.autoOptimize.autoCompact = true
    )
""")

# Enable on existing table
spark.sql("""
    ALTER TABLE sales SET TBLPROPERTIES (
        delta.autoOptimize.optimizeWrite = true,
        delta.autoOptimize.autoCompact = true
    )
""")
```

---

### 4.7 Change Data Feed (CDC)

```python
# Enable Change Data Feed
spark.sql("""
    ALTER TABLE customers SET TBLPROPERTIES (
        delta.enableChangeDataFeed = true
    )
""")

# Read changes since version
changes = spark.read.format("delta") \
    .option("readChangeFeed", "true") \
    .option("startingVersion", 5) \
    .table("customers")

changes.show()
# Columns: _change_type (insert, update_preimage, update_postimage, delete)
#          _commit_version, _commit_timestamp

# Read changes in time range
changes = spark.read.format("delta") \
    .option("readChangeFeed", "true") \
    .option("startingTimestamp", "2024-02-01") \
    .option("endingTimestamp", "2024-02-15") \
    .table("customers")
```

---

## 📓 LEVEL 5: DATABRICKS NOTEBOOKS

### 5.1 Notebook Basics

**Magic Commands:**

```python
# %md - Markdown cell
%md
# This is a heading
This is **bold** text
- Bullet point

# %sql - SQL cell
%sql
SELECT * FROM customers LIMIT 10

# %python - Python (default)
print("Hello from Python")

# %scala - Scala
%scala
val df = spark.read.table("customers")
df.show()

# %r - R
%r
library(SparkR)
df <- sql("SELECT * FROM customers")
head(df)

# %sh - Shell commands
%sh
ls -la /dbfs/mnt/data

# %fs - Filesystem commands
%fs ls /mnt/data

# %pip - Install packages
%pip install pandas matplotlib

# %run - Run another notebook
%run ./utilities/helper_functions
```

---

### 5.2 Databricks Utilities (dbutils)

**Essential dbutils commands:**

```python
# Filesystem operations
dbutils.fs.ls("/mnt/data")
dbutils.fs.cp("/source/path", "/dest/path", recurse=True)
dbutils.fs.rm("/path/to/delete", recurse=True)
dbutils.fs.mkdirs("/new/directory")
dbutils.fs.head("/path/to/file", maxBytes=1024)

# Secrets
api_key = dbutils.secrets.get(scope="my-scope", key="api-key")

# Widgets (notebook parameters)
dbutils.widgets.text("environment", "dev", "Environment")
dbutils.widgets.dropdown("region", "us-east", ["us-east", "us-west", "eu"], "Region")
env = dbutils.widgets.get("environment")

# Notebook workflow
dbutils.notebook.run(
    "/path/to/notebook",
    timeout_seconds=300,
    arguments={"param1": "value1"}
)

# Exit notebook with value
dbutils.notebook.exit("Success")
```

---

### 5.3 Visualization in Notebooks

```python
# Using display() for interactive viz
df = spark.sql("SELECT * FROM sales")
display(df)  # Interactive table with charts

# Matplotlib/Seaborn
import matplotlib.pyplot as plt
import pandas as pd

# Convert Spark DF to Pandas for plotting
pandas_df = df.toPandas()

plt.figure(figsize=(10, 6))
plt.bar(pandas_df['category'], pandas_df['total_sales'])
plt.title('Sales by Category')
plt.xlabel('Category')
plt.ylabel('Total Sales')
displayHTML(plt.gcf())  # Or just use display()

# Plotly for interactive charts
import plotly.express as px

fig = px.bar(pandas_df, x='category', y='total_sales', title='Sales by Category')
fig.show()
```

---

## 🔄 LEVEL 6: DATA ENGINEERING WORKFLOWS

### 6.1 Databricks Workflows (Jobs)

**Creating a Job (UI):**
1. Workflows → Create Job
2. Add tasks (notebooks, Python, JAR, SQL)
3. Define dependencies
4. Schedule or trigger
5. Configure cluster

**Programmatic Job Creation:**

```python
from databricks.sdk import WorkspaceClient
from databricks.sdk.service import jobs

w = WorkspaceClient()

# Create job
created_job = w.jobs.create(
    name="ETL Pipeline",
    tasks=[
        jobs.Task(
            task_key="bronze_ingestion",
            notebook_task=jobs.NotebookTask(
                notebook_path="/Workflows/Bronze",
                base_parameters={"environment": "prod"}
            ),
            new_cluster=jobs.ClusterSpec(
                spark_version="13.3.x-scala2.12",
                node_type_id="i3.xlarge",
                num_workers=2
            )
        ),
        jobs.Task(
            task_key="silver_transformation",
            depends_on=[jobs.TaskDependency(task_key="bronze_ingestion")],
            notebook_task=jobs.NotebookTask(
                notebook_path="/Workflows/Silver"
            ),
            existing_cluster_id="your-cluster-id"
        )
    ],
    schedule=jobs.CronSchedule(
        quartz_cron_expression="0 0 * * * ?",  # Daily at midnight
        timezone_id="America/New_York"
    )
)

# Trigger job run
run = w.jobs.run_now(job_id=created_job.job_id)
```

---

### 6.2 Delta Live Tables (DLT)

**What is DLT?**
- Declarative ETL framework
- Auto-scaling and optimization
- Data quality checks
- Automatic dependency management

**DLT Pipeline Example:**

```python
import dlt
from pyspark.sql.functions import *

# BRONZE: Ingest raw data
@dlt.table(
    comment="Raw customer events",
    table_properties={
        "quality": "bronze",
        "pipelines.autoOptimize.zOrderCols": "event_timestamp"
    }
)
def bronze_events():
    return (
        spark.readStream
            .format("cloudFiles")
            .option("cloudFiles.format", "json")
            .load("/mnt/raw/events/")
    )

# SILVER: Clean and validate
@dlt.table(
    comment="Cleaned customer events"
)
@dlt.expect_or_drop("valid_user_id", "user_id IS NOT NULL")
@dlt.expect_or_drop("valid_timestamp", "event_timestamp IS NOT NULL")
def silver_events():
    return (
        dlt.read_stream("bronze_events")
            .withColumn("event_timestamp", to_timestamp("timestamp"))
            .withColumn("processed_at", current_timestamp())
            .dropDuplicates(["event_id"])
    )

# GOLD: Business aggregates
@dlt.table(
    comment="Daily user activity metrics"
)
def gold_daily_metrics():
    return (
        dlt.read("silver_events")
            .groupBy(
                window("event_timestamp", "1 day").alias("date"),
                "user_id"
            )
            .agg(
                count("*").alias("event_count"),
                countDistinct("event_type").alias("unique_event_types")
            )
            .select(
                col("date.start").alias("date"),
                "user_id",
                "event_count",
                "unique_event_types"
            )
    )

# Data Quality Expectations
@dlt.table
@dlt.expect_or_fail("valid_email", "email RLIKE '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$'")
@dlt.expect_or_drop("valid_age", "age BETWEEN 18 AND 120")
@dlt.expect("valid_country", "country IS NOT NULL")  # Track violations
def validated_customers():
    return dlt.read("raw_customers")
```

**Creating DLT Pipeline (UI):**
1. Workflows → Delta Live Tables → Create Pipeline
2. Specify notebook with DLT code
3. Configure target schema
4. Set compute settings
5. Start pipeline

---

### 6.3 Streaming with Structured Streaming

**Basic Streaming:**

```python
# Read stream from Delta table
stream_df = spark.readStream \
    .format("delta") \
    .table("bronze.events")

# Transform
processed = stream_df \
    .filter(col("event_type") == "purchase") \
    .withColumn("revenue", col("price") * col("quantity"))

# Write stream
query = processed.writeStream \
    .format("delta") \
    .outputMode("append") \
    .option("checkpointLocation", "/mnt/checkpoints/purchases") \
    .trigger(processingTime="10 seconds") \
    .table("silver.purchases")

# Wait for termination
query.awaitTermination()
```

**Streaming Aggregations:**

```python
from pyspark.sql.functions import window

# Tumbling window aggregation
windowed = stream_df \
    .groupBy(
        window("event_timestamp", "5 minutes"),
        "product_id"
    ) \
    .agg(
        count("*").alias("event_count"),
        sum("revenue").alias("total_revenue")
    )

# Write to Delta
windowed.writeStream \
    .format("delta") \
    .outputMode("append") \  # or "update" or "complete"
    .option("checkpointLocation", "/mnt/checkpoints/windowed") \
    .table("gold.product_metrics_5min")
```

**Streaming Deduplication:**

```python
# Deduplicate based on watermark
deduplicated = stream_df \
    .withWatermark("event_timestamp", "1 hour") \
    .dropDuplicates(["event_id"])
```

**foreachBatch for Complex Logic:**

```python
def process_batch(batch_df, batch_id):
    # Custom processing logic
    # Can write to multiple sinks
    batch_df.write.format("delta").mode("append").saveAsTable("target_table")
    
    # Can also write to external systems
    batch_df.write.format("jdbc").save()

query = stream_df.writeStream \
    .foreachBatch(process_batch) \
    .start()
```

---

## 💼 LEVEL 7: DATABRICKS SQL

### 7.1 SQL Warehouses

**What are SQL Warehouses?**
- Serverless compute for SQL queries
- Auto-scaling
- BI tool integration
- Separate from Spark clusters

**Creating SQL Warehouse:**

```sql
-- Via UI: SQL → SQL Warehouses → Create

-- Configure:
-- - Warehouse size (2X-Small to 4X-Large)
-- - Auto-stop (minutes of inactivity)
-- - Scaling (min/max clusters)
-- - Serverless vs Pro vs Classic
```

**Querying:**

```sql
-- Standard SQL queries work!
SELECT 
    customer_id,
    COUNT(*) as order_count,
    SUM(total_amount) as total_spent,
    AVG(total_amount) as avg_order_value
FROM orders
WHERE order_date >= '2024-01-01'
GROUP BY customer_id
HAVING COUNT(*) > 5
ORDER BY total_spent DESC
LIMIT 100;

-- Use Delta features
SELECT * FROM customers VERSION AS OF 5;
SELECT * FROM customers TIMESTAMP AS OF '2024-02-01';

-- Analyze tables
ANALYZE TABLE customers COMPUTE STATISTICS;

-- Optimize
OPTIMIZE customers ZORDER BY (customer_id);
```

---

### 7.2 SQL Dashboards

**Creating Dashboards:**

1. **Save Query:**
```sql
-- Query: Daily Revenue
SELECT 
    DATE(order_date) as date,
    SUM(total_amount) as daily_revenue,
    COUNT(DISTINCT customer_id) as unique_customers
FROM orders
WHERE order_date >= CURRENT_DATE - INTERVAL 30 DAYS
GROUP BY DATE(order_date)
ORDER BY date
```

2. **Visualize:**
   - Add visualization (Line chart, Bar chart, etc.)
   - Customize colors, labels, axes
   
3. **Create Dashboard:**
   - SQL → Dashboards → Create
   - Add saved visualizations
   - Arrange layout

4. **Schedule Refresh:**
   - Set refresh schedule
   - Email reports

---

### 7.3 SQL Query Optimization

```sql
-- Use proper predicates for partition pruning
-- Bad:
SELECT * FROM large_table WHERE YEAR(date) = 2024;

-- Good:
SELECT * FROM large_table 
WHERE date >= '2024-01-01' AND date < '2025-01-01';

-- Use BROADCAST hint for small tables
SELECT /*+ BROADCAST(small_table) */
    a.*, b.*
FROM large_table a
JOIN small_table b ON a.id = b.id;

-- Create statistics
ANALYZE TABLE sales COMPUTE STATISTICS;
ANALYZE TABLE sales COMPUTE STATISTICS FOR COLUMNS product_id, customer_id;

-- Check query plan
EXPLAIN EXTENDED
SELECT * FROM sales WHERE product_id = 123;
```

---

## 🤖 LEVEL 8: MLFLOW & MACHINE LEARNING

### 8.1 MLflow Basics

**What is MLflow?**
- Experiment tracking
- Model registry
- Model deployment
- Integrated in Databricks

**Experiment Tracking:**

```python
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

# Set experiment
mlflow.set_experiment("/Users/your_email/my-ml-experiment")

# Load data
df = spark.table("features.customer_churn").toPandas()
X = df.drop("churn", axis=1)
y = df["churn"]
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# Train with MLflow
with mlflow.start_run(run_name="random_forest_v1"):
    # Log parameters
    mlflow.log_param("n_estimators", 100)
    mlflow.log_param("max_depth", 10)
    
    # Train model
    model = RandomForestClassifier(n_estimators=100, max_depth=10)
    model.fit(X_train, y_train)
    
    # Evaluate
    predictions = model.predict(X_test)
    accuracy = accuracy_score(y_test, predictions)
    
    # Log metrics
    mlflow.log_metric("accuracy", accuracy)
    
    # Log model
    mlflow.sklearn.log_model(model, "model")
    
    # Log artifacts
    import matplotlib.pyplot as plt
    plt.figure()
    # Create feature importance plot
    plt.savefig("feature_importance.png")
    mlflow.log_artifact("feature_importance.png")

# Search runs
runs = mlflow.search_runs(experiment_ids=["your_experiment_id"])
best_run = runs.sort_values("metrics.accuracy", ascending=False).iloc[0]
```

---

### 8.2 Model Registry

```python
# Register model
model_name = "customer_churn_model"

model_uri = f"runs:/{run_id}/model"
model_version = mlflow.register_model(model_uri, model_name)

# Transition model stage
from mlflow.tracking import MlflowClient
client = MlflowClient()

client.transition_model_version_stage(
    name=model_name,
    version=model_version.version,
    stage="Production"
)

# Load model for inference
model = mlflow.pyfunc.load_model(f"models:/{model_name}/Production")

# Predict
new_data = pd.DataFrame({...})
predictions = model.predict(new_data)

# Add model description
client.update_registered_model(
    name=model_name,
    description="Random Forest model for predicting customer churn"
)

# Add version description
client.update_model_version(
    name=model_name,
    version=model_version.version,
    description="Initial production model with 92% accuracy"
)
```

---

### 8.3 Feature Store

```python
from databricks.feature_store import FeatureStoreClient

fs = FeatureStoreClient()

# Create feature table
fs.create_table(
    name="ml.customer_features",
    primary_keys=["customer_id"],
    df=customer_features_df,
    description="Customer demographic and behavioral features"
)

# Write features
fs.write_table(
    name="ml.customer_features",
    df=updated_features_df,
    mode="merge"
)

# Create training set
training_set = fs.create_training_set(
    df=labels_df,  # DataFrame with labels
    feature_lookups=[
        FeatureLookup(
            table_name="ml.customer_features",
            lookup_key="customer_id"
        )
    ],
    label="churn"
)

# Load training data
training_df = training_set.load_df()

# Train with feature store
with mlflow.start_run():
    model = train_model(training_df)
    
    # Log model with feature dependencies
    fs.log_model(
        model=model,
        artifact_path="model",
        flavor=mlflow.sklearn,
        training_set=training_set
    )
```

---

### 8.4 AutoML

```python
from databricks import automl

# Run AutoML
summary = automl.classify(
    dataset=training_df,
    target_col="churn",
    primary_metric="f1",
    timeout_minutes=30,
    max_trials=20
)

# Get best model
best_trial = summary.best_trial
best_model_uri = best_trial.model_path

# Register best model
mlflow.register_model(best_model_uri, "automl_churn_model")
```

---

## ⚡ LEVEL 9: PERFORMANCE OPTIMIZATION

### 9.1 Cluster Configuration

**Cluster Types:**

```python
# All-Purpose Cluster (interactive work)
{
    "cluster_name": "Interactive Cluster",
    "spark_version": "13.3.x-scala2.12",
    "node_type_id": "i3.xlarge",
    "num_workers": 4,
    "autoscale": {
        "min_workers": 2,
        "max_workers": 8
    },
    "autotermination_minutes": 30
}

# Job Cluster (automated jobs)
{
    "cluster_name": "ETL Job Cluster",
    "spark_version": "13.3.x-scala2.12",
    "node_type_id": "r5d.xlarge",  # Memory-optimized
    "num_workers": 10,
    "spark_conf": {
        "spark.sql.adaptive.enabled": "true",
        "spark.sql.adaptive.coalescePartitions.enabled": "true"
    }
}
```

**Choosing Instance Types:**

| Workload | Instance Type | Reason |
|----------|---------------|--------|
| General ETL | Standard (m5) | Balanced CPU/memory |
| Memory-intensive | Memory-optimized (r5) | ML, aggregations |
| Storage-intensive | Storage-optimized (i3) | Large shuffles |
| Compute-intensive | Compute-optimized (c5) | Complex calculations |

---

### 9.2 Spark Configuration Tuning

```python
# Set Spark configs at cluster level or session level

# Adaptive Query Execution (AQE)
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")

# Broadcast join threshold
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", 10 * 1024 * 1024)  # 10MB

# Shuffle partitions
spark.conf.set("spark.sql.shuffle.partitions", 200)  # Default is 200

# Memory management
spark.conf.set("spark.executor.memory", "8g")
spark.conf.set("spark.driver.memory", "4g")
spark.conf.set("spark.memory.fraction", 0.8)

# Serialization
spark.conf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
```

---

### 9.3 Partitioning Strategies

```python
# Check current partitions
print(f"Number of partitions: {df.rdd.getNumPartitions()}")

# Repartition (shuffle)
df_repartitioned = df.repartition(100)
df_by_column = df.repartition("date")  # Partition by column

# Coalesce (no shuffle, only reduce)
df_coalesced = df.coalesce(10)

# Write partitioned data
df.write.format("delta") \
    .partitionBy("year", "month") \
    .mode("overwrite") \
    .saveAsTable("partitioned_data")

# Optimal partition size: 128MB - 1GB per partition
target_partitions = int(df.count() / 1000000)  # ~1M rows per partition
df = df.repartition(target_partitions)
```

---

### 9.4 Caching Strategies

```python
# Cache DataFrame in memory
df.cache()
df.count()  # Trigger caching

# Persist with storage level
from pyspark import StorageLevel
df.persist(StorageLevel.MEMORY_AND_DISK)

# Unpersist when done
df.unpersist()

# Cache Delta table (Databricks specific)
spark.sql("CACHE SELECT * FROM large_table")

# Uncache
spark.sql("UNCACHE TABLE large_table")

# When to cache:
# ✓ DataFrame used multiple times
# ✓ After expensive operations (join, aggregation)
# ✓ Iterative algorithms (ML)
# ✗ Large DataFrames that don't fit in memory
# ✗ Single-use DataFrames
```

---

### 9.5 Optimization Best Practices

```python
# 1. Filter early
df = df.filter(col("date") >= "2024-01-01")  # Do this first!

# 2. Select only needed columns
df = df.select("id", "name", "amount")  # Not SELECT *

# 3. Use broadcast for small tables
from pyspark.sql.functions import broadcast
result = large_df.join(broadcast(small_df), "id")

# 4. Avoid UDFs when possible (use built-in functions)
# Bad:
from pyspark.sql.functions import udf
def multiply(x):
    return x * 2
multiply_udf = udf(multiply)
df = df.withColumn("doubled", multiply_udf(col("value")))

# Good:
df = df.withColumn("doubled", col("value") * 2)

# 5. Use DataFrame API over RDD
# DataFrame API is optimized by Catalyst optimizer

# 6. Avoid collect() on large datasets
# Bad:
data = df.collect()  # Brings ALL data to driver!

# Good:
df.write.format("delta").save("/output")  # Distributed write

# 7. Monitor with Spark UI
# Access at: Cluster → Spark UI → Jobs/Stages/Storage
```

---

## 🔐 LEVEL 10: UNITY CATALOG

### 10.1 What is Unity Catalog?

**Unified Governance for Lakehouse:**
- Single place for data + AI assets
- Fine-grained access control
- Data lineage
- Audit logging
- Data discovery

**Three-Level Namespace:**
```
catalog.schema.table
   ↓      ↓      ↓
 prod.sales.customers
```

---

### 10.2 Creating Unity Catalog Objects

```sql
-- Create catalog
CREATE CATALOG IF NOT EXISTS prod
COMMENT 'Production data catalog';

-- Create schema
CREATE SCHEMA IF NOT EXISTS prod.sales
COMMENT 'Sales department data';

-- Create table
CREATE TABLE prod.sales.customers (
    customer_id BIGINT,
    name STRING,
    email STRING,
    created_date DATE
)
USING DELTA
LOCATION 's3://my-bucket/prod/sales/customers';

-- Create external table
CREATE EXTERNAL TABLE prod.sales.external_data
LOCATION 's3://external-bucket/data/';

-- Create view
CREATE VIEW prod.sales.active_customers AS
SELECT * FROM prod.sales.customers
WHERE is_active = true;

-- Create function
CREATE FUNCTION prod.sales.calculate_discount(price DOUBLE, rate DOUBLE)
RETURNS DOUBLE
RETURN price * (1 - rate);
```

---

### 10.3 Access Control

```sql
-- Grant privileges
GRANT USE CATALOG ON CATALOG prod TO `data-engineers`;
GRANT USE SCHEMA ON SCHEMA prod.sales TO `data-engineers`;
GRANT SELECT ON TABLE prod.sales.customers TO `analysts`;
GRANT MODIFY ON TABLE prod.sales.customers TO `data-engineers`;

-- Create custom role
GRANT SELECT ON SCHEMA prod.sales TO `sales-team`;

-- Row-level security with Dynamic Views
CREATE VIEW prod.sales.regional_sales AS
SELECT * FROM prod.sales.sales
WHERE region = current_user();  -- Filter by user

-- Column masking
CREATE FUNCTION prod.sales.mask_email(email STRING)
RETURNS STRING
RETURN 
    CASE 
        WHEN is_account_group_member('pii-access') 
        THEN email
        ELSE regexp_replace(email, '^(.)[^@]+', '$1***')
    END;

CREATE VIEW prod.sales.customers_masked AS
SELECT 
    customer_id,
    name,
    prod.sales.mask_email(email) as email
FROM prod.sales.customers;

-- Revoke privileges
REVOKE SELECT ON TABLE prod.sales.customers FROM `analysts`;
```

---

### 10.4 Data Lineage

```python
# View lineage in UI:
# Data → [Table] → Lineage tab

# Programmatic access
from databricks.sdk import WorkspaceClient

w = WorkspaceClient()

# Get table lineage
lineage = w.workspace.get_lineage(
    table_name="prod.sales.customers"
)

# Shows:
# - Upstream dependencies (what this table depends on)
# - Downstream dependencies (what depends on this table)
# - Jobs that create/modify the table
```

---

## 🏭 LEVEL 11: PRODUCTION BEST PRACTICES

### 11.1 Environment Strategy

```
Development → Staging → Production

dev.schema.table
staging.schema.table
prod.schema.table
```

**Using Widgets for Environment:**

```python
# In notebook
dbutils.widgets.dropdown("environment", "dev", ["dev", "staging", "prod"])
env = dbutils.widgets.get("environment")

# Use in code
catalog = env
schema = "sales"
table = f"{catalog}.{schema}.customers"

df = spark.read.table(table)
```

---

### 11.2 CI/CD Pipeline

**Git Integration:**

1. **Repos → Add Repo:**
   - Connect GitHub/GitLab/Bitbucket
   - Clone repository
   - Work on branches

2. **Databricks Asset Bundles (DAB):**

```yaml
# databricks.yml
bundle:
  name: my-etl-pipeline

resources:
  jobs:
    daily_etl:
      name: Daily ETL Job
      tasks:
        - task_key: bronze
          notebook_task:
            notebook_path: /notebooks/bronze_ingestion
          new_cluster:
            spark_version: 13.3.x-scala2.12
            node_type_id: i3.xlarge
            num_workers: 2
        
        - task_key: silver
          depends_on:
            - task_key: bronze
          notebook_task:
            notebook_path: /notebooks/silver_transformation
```

**Deploy:**
```bash
# Install Databricks CLI
pip install databricks-cli

# Configure
databricks configure --token

# Deploy to dev
databricks bundle deploy --target dev

# Deploy to prod
databricks bundle deploy --target prod

# Run job
databricks bundle run daily_etl --target prod
```

---

### 11.3 Error Handling & Logging

```python
import logging

# Setup logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

# Error handling in ETL
def process_data():
    try:
        logger.info("Starting data processing")
        
        df = spark.read.table("source_table")
        
        # Validation
        if df.count() == 0:
            raise ValueError("Source table is empty!")
        
        # Process
        result = df.filter(col("amount") > 0)
        
        # Write
        result.write.format("delta").mode("overwrite").saveAsTable("target_table")
        
        logger.info(f"Successfully processed {result.count()} records")
        
        return "Success"
        
    except Exception as e:
        logger.error(f"Error processing data: {str(e)}")
        
        # Write to error table
        error_df = spark.createDataFrame(
            [(str(e), current_timestamp())],
            ["error_message", "error_timestamp"]
        )
        error_df.write.format("delta").mode("append").saveAsTable("error_log")
        
        raise  # Re-raise for job failure

# Call function
result = process_data()
dbutils.notebook.exit(result)
```

---

### 11.4 Monitoring & Alerting

```python
# Create monitoring table
spark.sql("""
    CREATE TABLE IF NOT EXISTS monitoring.job_metrics (
        job_name STRING,
        run_id STRING,
        start_time TIMESTAMP,
        end_time TIMESTAMP,
        status STRING,
        records_processed BIGINT,
        error_message STRING
    )
    USING DELTA
""")

# Log metrics
from datetime import datetime

def log_job_metrics(job_name, status, records_processed=0, error_message=None):
    metrics = spark.createDataFrame([{
        "job_name": job_name,
        "run_id": spark.sparkContext.applicationId,
        "start_time": start_time,
        "end_time": datetime.now(),
        "status": status,
        "records_processed": records_processed,
        "error_message": error_message
    }])
    
    metrics.write.format("delta").mode("append").saveAsTable("monitoring.job_metrics")

# Use in pipeline
start_time = datetime.now()
try:
    df = process_data()
    log_job_metrics("ETL Pipeline", "SUCCESS", df.count())
except Exception as e:
    log_job_metrics("ETL Pipeline", "FAILED", 0, str(e))
    raise

# Query metrics
spark.sql("""
    SELECT 
        job_name,
        DATE(end_time) as date,
        COUNT(*) as runs,
        SUM(CASE WHEN status = 'SUCCESS' THEN 1 ELSE 0 END) as successful,
        SUM(CASE WHEN status = 'FAILED' THEN 1 ELSE 0 END) as failed,
        AVG(records_processed) as avg_records
    FROM monitoring.job_metrics
    WHERE end_time >= CURRENT_DATE - INTERVAL 7 DAYS
    GROUP BY job_name, DATE(end_time)
""").display()
```

---

### 11.5 Cost Optimization

**Strategies:**

```python
# 1. Use appropriate cluster sizes
# Start small, scale up only when needed

# 2. Auto-termination
# Set aggressive timeouts (10-30 minutes)

# 3. Use Spot instances (for fault-tolerant workloads)
{
    "aws_attributes": {
        "first_on_demand": 1,
        "availability": "SPOT_WITH_FALLBACK",
        "spot_bid_price_percent": 100
    }
}

# 4. Use Photon (Databricks' vectorized engine)
{
    "runtime_engine": "PHOTON"
}

# 5. Monitor DBU usage
dbutils.notebook.run(
    "/Utils/cost_monitoring",
    timeout_seconds=300
)

# 6. Use Delta table features
spark.sql("OPTIMIZE customers ZORDER BY (customer_id)")
spark.sql("VACUUM customers RETAIN 168 HOURS")  # Clean old files
```

---

## 🏗️ PRACTICE PROJECTS

### Project 1: End-to-End Lakehouse

**Build medallion architecture:**

1. **Bronze Layer:**
```python
# Auto-loader for continuous ingestion
(spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "json")
    .option("cloudFiles.schemaLocation", "/checkpoints/schema")
    .load("s3://raw-data/events/")
    .writeStream
    .format("delta")
    .option("checkpointLocation", "/checkpoints/bronze")
    .table("bronze.events")
)
```

2. **Silver Layer:**
```python
# DLT for transformations
@dlt.table
@dlt.expect_or_drop("valid_data", "user_id IS NOT NULL")
def silver_events():
    return (
        dlt.read_stream("bronze.events")
            .withColumn("event_date", to_date("timestamp"))
            .dropDuplicates(["event_id"])
    )
```

3. **Gold Layer:**
```python
# Business metrics
@dlt.table
def gold_daily_metrics():
    return (
        dlt.read("silver.events")
            .groupBy("event_date", "event_type")
            .agg(count("*").alias("count"))
    )
```

4. **ML Pipeline:**
```python
# Feature engineering
# Model training with MLflow
# Model deployment to registry
```

---

### Project 2: Real-Time Analytics

1. Kafka → Databricks streaming
2. Real-time aggregations
3. Dashboard updates
4. Alerting on anomalies

---

### Project 3: ML Pipeline

1. Feature store setup
2. Training pipeline
3. Model registry
4. Batch inference
5. Model monitoring

---

## 🎤 INTERVIEW PREP

### Top Databricks Interview Questions:

**1. What is the Lakehouse architecture?**
- Combines data lake + data warehouse
- ACID transactions with Delta Lake
- Single copy of data
- Supports BI, ML, streaming

**2. Delta Lake vs Parquet?**
- Delta = ACID, time travel, MERGE operations
- Parquet = just a file format
- Delta built on top of Parquet

**3. Explain Databricks cluster types**
- All-purpose: Interactive work
- Job clusters: Automated jobs (cheaper)
- SQL warehouses: BI queries

**4. What is Unity Catalog?**
- Unified governance
- catalog.schema.table namespace
- Fine-grained access control
- Data lineage

**5. How do you optimize Spark jobs?**
- Proper partitioning
- Broadcast small tables
- Cache reused data
- Use built-in functions over UDFs
- Enable AQE

**6. Explain streaming in Databricks**
- Structured Streaming
- Exactly-once semantics
- Auto Loader for cloud files
- Checkpointing for fault tolerance

**7. What is MLflow?**
- Experiment tracking
- Model registry
- Model deployment
- Integrated in Databricks

**8. How do you handle slowly changing dimensions?**
- SCD Type 1: Overwrite
- SCD Type 2: Add new row with versioning
- Use Delta MERGE for updates

**9. Cost optimization strategies?**
- Right-size clusters
- Use Spot instances
- Photon engine
- Auto-termination
- Optimize Delta tables

**10. CI/CD in Databricks?**
- Git integration (Repos)
- Databricks Asset Bundles
- Automated testing
- Multi-environment deployment

---

## ✅ DATABRICKS MASTERY CHECKLIST

### Beginner ✓
- [ ] Understand Lakehouse concept
- [ ] Create notebooks, run Spark code
- [ ] Read/write Delta tables
- [ ] Use basic DataFrame operations
- [ ] Create simple jobs

### Intermediate ✓
- [ ] Implement medallion architecture
- [ ] Use Delta Lake features (MERGE, time travel)
- [ ] Build streaming pipelines
- [ ] Use Delta Live Tables
- [ ] Basic MLflow usage

### Advanced ✓
- [ ] Optimize Spark performance
- [ ] Use Unity Catalog
- [ ] Implement CI/CD
- [ ] Build production ML pipelines
- [ ] Cost optimization

### Expert ✓
- [ ] Design complex data architectures
- [ ] Tune cluster configurations
- [ ] Implement governance at scale
- [ ] Build AutoML solutions
- [ ] Architect multi-cloud solutions

---

## 📚 LEARNING RESOURCES

**Official:**
- Databricks Academy (free courses)
- Documentation: docs.databricks.com
- Community Edition (free tier)

**Certifications:**
- Databricks Certified Data Engineer Associate
- Databricks Certified Data Engineer Professional
- Databricks Certified Machine Learning Professional

**Practice:**
- Community Edition signup
- Hands-on tutorials
- Sample notebooks

---

## 🎯 30-DAY LEARNING PLAN

**Week 1: Foundations**
- Days 1-2: Lakehouse concept, Spark basics
- Days 3-4: DataFrame API practice
- Days 5-7: Delta Lake fundamentals

**Week 2: Data Engineering**
- Days 8-10: Medallion architecture
- Days 11-12: Streaming pipelines
- Days 13-14: Delta Live Tables

**Week 3: ML & Advanced**
- Days 15-17: MLflow basics
- Days 18-19: Feature Store
- Days 20-21: Performance optimization

**Week 4: Production**
- Days 22-24: Unity Catalog
- Days 25-27: CI/CD, monitoring
- Days 28-30: End-to-end project

---

**Ready for next topic?** 

Choose:
1. **Python Deep Dive** - Advanced pandas, data manipulation
2. **Apache Airflow** - Workflow orchestration
3. **Azure Data Factory** - Cloud ETL
4. **Power BI** - Data visualization
5. **dbt** - Data transformation

Which one? 🚀
