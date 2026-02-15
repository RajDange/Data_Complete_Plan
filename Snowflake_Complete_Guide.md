# ❄️ SNOWFLAKE MASTERY - Complete Learning Guide

## 📋 Table of Contents
1. [Snowflake Fundamentals](#fundamentals)
2. [Architecture Deep Dive](#architecture)
3. [Virtual Warehouses](#warehouses)
4. [Storage & File Formats](#storage)
5. [Data Loading](#loading)
6. [Semi-Structured Data](#semi-structured)
7. [Advanced Features](#advanced)
8. [Performance Optimization](#optimization)
9. [Security & Governance](#security)
10. [Cost Optimization](#cost)
11. [Practice Projects](#projects)
12. [Interview Prep](#interview)

---

## 🎯 LEVEL 1: SNOWFLAKE FUNDAMENTALS

### 1.1 What Makes Snowflake Different?

**Traditional Warehouse vs Snowflake:**

| Feature | Traditional | Snowflake |
|---------|-------------|-----------|
| **Storage** | Coupled with compute | Separated from compute |
| **Scaling** | Manual, downtime | Instant, automatic |
| **Pricing** | Fixed costs | Pay-per-use (per-second) |
| **Maintenance** | You manage | Fully managed |
| **Data Sharing** | Complex ETL | Native, instant |
| **Concurrency** | Limited | Unlimited (multi-cluster) |

**Key Differentiators:**
✓ **Separation of storage and compute** - Scale independently
✓ **Zero-copy cloning** - Instant copies without storage cost
✓ **Time Travel** - Query historical data (up to 90 days)
✓ **Data Sharing** - Share live data without copying
✓ **Semi-structured data** - Native JSON/Avro/Parquet support
✓ **No indexes or tuning** - Automatic optimization

---

### 1.2 Snowflake Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    CLOUD SERVICES LAYER                      │
│  (Authentication, Metadata, Query Optimization, Security)    │
└─────────────────────────────────────────────────────────────┘
                            ▲
                            │
┌─────────────────────────────────────────────────────────────┐
│                    COMPUTE LAYER                             │
│                 (Virtual Warehouses)                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │   VW 1   │  │   VW 2   │  │   VW 3   │  │   VW N   │   │
│  │ X-Small  │  │  Medium  │  │  Large   │  │  X-Large │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
                            ▲
                            │
┌─────────────────────────────────────────────────────────────┐
│                    STORAGE LAYER                             │
│         (Micro-partitions, Columnar Storage)                 │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Database 1  │  Database 2  │  Database 3  │  ...   │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

**Three Layers Explained:**

1. **Cloud Services Layer**
   - Query parsing and optimization
   - Metadata management
   - Authentication and access control
   - Infrastructure management

2. **Compute Layer (Virtual Warehouses)**
   - Execute queries
   - Load/unload data
   - Scale up/down/out independently
   - Multiple warehouses for different workloads

3. **Storage Layer**
   - Compressed, columnar storage
   - Micro-partitions (automatic)
   - Scalable, durable storage
   - Cross-region replication

---

### 1.3 Snowflake Objects Hierarchy

```
ORGANIZATION
    └── ACCOUNT
        └── DATABASE
            └── SCHEMA
                ├── TABLES
                ├── VIEWS
                ├── STAGES
                ├── FILE FORMATS
                ├── SEQUENCES
                ├── PIPES
                ├── STREAMS
                ├── TASKS
                └── STORED PROCEDURES
```

**Key Concepts:**
- **Account**: Your Snowflake instance (e.g., xy12345.us-east-1)
- **Database**: Top-level container for schemas
- **Schema**: Container for database objects
- **Objects**: Tables, views, stages, etc.

---

### 1.4 Getting Started - First Steps

**Creating Your First Objects:**

```sql
-- 1. Create a Database
CREATE DATABASE MY_FIRST_DB;

-- 2. Use the database
USE DATABASE MY_FIRST_DB;

-- 3. Create a Schema
CREATE SCHEMA MY_SCHEMA;
USE SCHEMA MY_SCHEMA;

-- 4. Create a simple table
CREATE TABLE CUSTOMERS (
    CUSTOMER_ID INT AUTOINCREMENT,
    FIRST_NAME VARCHAR(50),
    LAST_NAME VARCHAR(50),
    EMAIL VARCHAR(100),
    SIGNUP_DATE DATE,
    IS_ACTIVE BOOLEAN DEFAULT TRUE,
    METADATA VARIANT  -- For JSON data
);

-- 5. Insert data
INSERT INTO CUSTOMERS (FIRST_NAME, LAST_NAME, EMAIL, SIGNUP_DATE)
VALUES 
    ('John', 'Doe', 'john@example.com', CURRENT_DATE()),
    ('Jane', 'Smith', 'jane@example.com', CURRENT_DATE());

-- 6. Query data
SELECT * FROM CUSTOMERS;

-- 7. Show current context
SELECT CURRENT_DATABASE(), CURRENT_SCHEMA(), CURRENT_WAREHOUSE();
```

---

## 🏗️ LEVEL 2: VIRTUAL WAREHOUSES

### 2.1 Understanding Virtual Warehouses

**What is a Virtual Warehouse?**
- Cluster of compute resources (CPU, memory, temp storage)
- Used to execute queries and load data
- Scale up = bigger warehouse (more resources per cluster)
- Scale out = multi-cluster (more clusters)

**Warehouse Sizes:**

| Size | Credits/Hour | Relative Power | Use Case |
|------|--------------|----------------|----------|
| X-Small | 1 | 1x | Dev, testing, small queries |
| Small | 2 | 2x | Light production workloads |
| Medium | 4 | 4x | Standard analytics |
| Large | 8 | 8x | Heavy ETL, complex queries |
| X-Large | 16 | 16x | Very large data processing |
| 2X-Large | 32 | 32x | Massive datasets |
| 3X-Large | 64 | 64x | Extreme workloads |
| 4X-Large | 128 | 128x | Largest available |

**Creating and Managing Warehouses:**

```sql
-- Create a warehouse
CREATE WAREHOUSE ANALYTICS_WH
    WAREHOUSE_SIZE = 'MEDIUM'
    AUTO_SUSPEND = 60          -- Suspend after 60 seconds of inactivity
    AUTO_RESUME = TRUE         -- Auto-resume when query submitted
    INITIALLY_SUSPENDED = TRUE
    COMMENT = 'Warehouse for analytics team';

-- Use the warehouse
USE WAREHOUSE ANALYTICS_WH;

-- Resize warehouse (instant!)
ALTER WAREHOUSE ANALYTICS_WH SET WAREHOUSE_SIZE = 'LARGE';

-- Suspend manually
ALTER WAREHOUSE ANALYTICS_WH SUSPEND;

-- Resume manually
ALTER WAREHOUSE ANALYTICS_WH RESUME;

-- Multi-cluster warehouse (Enterprise Edition)
CREATE WAREHOUSE ETL_WH
    WAREHOUSE_SIZE = 'LARGE'
    MIN_CLUSTER_COUNT = 1
    MAX_CLUSTER_COUNT = 5      -- Scale out to 5 clusters
    SCALING_POLICY = 'STANDARD'  -- or 'ECONOMY'
    AUTO_SUSPEND = 60
    AUTO_RESUME = TRUE;

-- Show warehouses
SHOW WAREHOUSES;

-- Get warehouse usage
SELECT * FROM TABLE(INFORMATION_SCHEMA.WAREHOUSE_METERING_HISTORY(
    DATE_RANGE_START => DATEADD('days', -7, CURRENT_DATE())
));
```

**Best Practices:**
✓ **Use separate warehouses** for different workloads (ETL vs BI vs Data Science)
✓ **Auto-suspend** to save costs (60-300 seconds)
✓ **Start small** - scale up only when needed
✓ **Multi-cluster** for concurrent users
✓ **Monitor usage** regularly

---

### 2.2 Warehouse Sizing Strategy

**When to Scale UP (bigger warehouse):**
- Large, complex queries
- Heavy data transformations
- Fast query execution needed
- Single-user heavy workload

**When to Scale OUT (multi-cluster):**
- Many concurrent users
- Queuing issues
- BI tool with many users
- Consistent high load

**Example Scenarios:**

```sql
-- Scenario 1: Heavy ETL job
-- Use Large warehouse for faster processing
ALTER WAREHOUSE ETL_WH SET WAREHOUSE_SIZE = 'LARGE';

-- Run ETL
INSERT INTO fact_sales
SELECT ... -- Complex transformation
FROM staging_sales;

-- Scale back down after
ALTER WAREHOUSE ETL_WH SET WAREHOUSE_SIZE = 'MEDIUM';

-- Scenario 2: BI Dashboard (many users)
-- Use multi-cluster to handle concurrency
CREATE WAREHOUSE BI_WH
    WAREHOUSE_SIZE = 'MEDIUM'
    MIN_CLUSTER_COUNT = 1
    MAX_CLUSTER_COUNT = 10;
```

---

## 💾 LEVEL 3: STORAGE & FILE FORMATS

### 3.1 Micro-Partitions (Automatic!)

**What are Micro-Partitions?**
- Snowflake automatically partitions data
- 50-500 MB compressed size
- Columnar format
- Immutable (never modified, only replaced)
- Metadata tracked automatically

**How it works:**
```
Table SALES (1 billion rows)
    ↓
Automatically split into ~2000 micro-partitions
    ↓
Each partition: metadata (min/max values, distinct counts, NULL counts)
    ↓
Query uses metadata to SKIP irrelevant partitions (pruning)
```

**Clustering Keys (Manual Optimization):**

```sql
-- Create table with clustering key
CREATE TABLE SALES (
    SALE_DATE DATE,
    PRODUCT_ID INT,
    CUSTOMER_ID INT,
    AMOUNT DECIMAL(10,2)
)
CLUSTER BY (SALE_DATE);  -- Physically organize by date

-- Check clustering quality
SELECT SYSTEM$CLUSTERING_INFORMATION('SALES', '(SALE_DATE)');

-- Re-cluster if needed (automatic with AUTO_CLUSTERING)
ALTER TABLE SALES RECLUSTER;

-- Enable automatic clustering
ALTER TABLE SALES RESUME RECLUSTER;
```

**When to use Clustering:**
✓ Very large tables (multi-TB)
✓ Queries always filter on same columns
✓ Performance issues with pruning
✓ Cost-benefit analysis needed (clustering costs credits!)

---

### 3.2 Supported File Formats

**Structured Formats:**
- CSV (with options for delimiters, compression, etc.)
- TSV (tab-separated)
- Delimited files (custom delimiters)

**Semi-Structured Formats:**
- JSON (native VARIANT type)
- Avro
- ORC
- Parquet
- XML

**Creating File Formats:**

```sql
-- CSV format
CREATE FILE FORMAT CSV_FORMAT
    TYPE = 'CSV'
    FIELD_DELIMITER = ','
    SKIP_HEADER = 1
    NULL_IF = ('NULL', 'null', '')
    EMPTY_FIELD_AS_NULL = TRUE
    COMPRESSION = 'GZIP'
    ERROR_ON_COLUMN_COUNT_MISMATCH = FALSE;

-- JSON format
CREATE FILE FORMAT JSON_FORMAT
    TYPE = 'JSON'
    COMPRESSION = 'AUTO'
    STRIP_OUTER_ARRAY = TRUE;

-- Parquet format
CREATE FILE FORMAT PARQUET_FORMAT
    TYPE = 'PARQUET'
    COMPRESSION = 'SNAPPY';

-- Show file formats
SHOW FILE FORMATS;
```

---

### 3.3 Stages (Where Files Live)

**Types of Stages:**

1. **Internal Stages** (Snowflake-managed)
   - User stage (private per user)
   - Table stage (private per table)
   - Named stage (shared)

2. **External Stages** (Cloud storage)
   - AWS S3
   - Azure Blob Storage
   - Google Cloud Storage

**Creating Stages:**

```sql
-- Internal named stage
CREATE STAGE MY_STAGE
    FILE_FORMAT = CSV_FORMAT
    COMMENT = 'Stage for CSV files';

-- External stage (AWS S3)
CREATE STAGE S3_STAGE
    URL = 's3://my-bucket/data/'
    STORAGE_INTEGRATION = MY_S3_INTEGRATION  -- Setup separately
    FILE_FORMAT = CSV_FORMAT;

-- External stage (Azure)
CREATE STAGE AZURE_STAGE
    URL = 'azure://myaccount.blob.core.windows.net/mycontainer/path/'
    STORAGE_INTEGRATION = MY_AZURE_INTEGRATION
    FILE_FORMAT = PARQUET_FORMAT;

-- List files in stage
LIST @MY_STAGE;

-- Put files into stage (from local)
PUT file:///tmp/data.csv @MY_STAGE;

-- Get files from stage (to local)
GET @MY_STAGE/data.csv file:///tmp/;

-- Remove files from stage
REMOVE @MY_STAGE/data.csv;
```

---

## 📥 LEVEL 4: DATA LOADING

### 4.1 COPY INTO (Bulk Loading)

**Basic COPY INTO:**

```sql
-- Load from internal stage
COPY INTO CUSTOMERS
FROM @MY_STAGE/customers.csv
FILE_FORMAT = CSV_FORMAT
ON_ERROR = 'CONTINUE'  -- or 'ABORT_STATEMENT', 'SKIP_FILE'
PURGE = TRUE;          -- Delete files after loading

-- Load from external S3
COPY INTO SALES
FROM @S3_STAGE/sales/
FILE_FORMAT = (TYPE = 'PARQUET')
PATTERN = '.*sales_2024.*[.]parquet'  -- Regex pattern
VALIDATION_MODE = 'RETURN_ERRORS';    -- Check for errors without loading

-- Load with transformations
COPY INTO SALES (SALE_DATE, PRODUCT_ID, AMOUNT)
FROM (
    SELECT 
        $1::DATE,           -- Column 1 as DATE
        $2::INT,            -- Column 2 as INT
        $3::DECIMAL(10,2)   -- Column 3 as DECIMAL
    FROM @MY_STAGE/sales.csv
)
FILE_FORMAT = CSV_FORMAT;

-- Load JSON data
COPY INTO JSON_TABLE
FROM @MY_STAGE/data.json
FILE_FORMAT = JSON_FORMAT
MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE;
```

**Validation Before Loading:**

```sql
-- Validate data (shows errors without loading)
COPY INTO CUSTOMERS
FROM @MY_STAGE/customers.csv
FILE_FORMAT = CSV_FORMAT
VALIDATION_MODE = 'RETURN_ERRORS';

-- Get detailed errors
SELECT * FROM TABLE(VALIDATE(CUSTOMERS, JOB_ID => '_last'));

-- Check copy history
SELECT * FROM TABLE(INFORMATION_SCHEMA.COPY_HISTORY(
    TABLE_NAME => 'CUSTOMERS',
    START_TIME => DATEADD(hours, -1, CURRENT_TIMESTAMP())
));
```

---

### 4.2 Snowpipe (Continuous Loading)

**What is Snowpipe?**
- Automated, continuous data ingestion
- Serverless (uses Snowflake-managed compute)
- Triggered by file arrival
- Low latency (typically < 1 minute)

**Setting up Snowpipe:**

```sql
-- Create pipe
CREATE PIPE SALES_PIPE
    AUTO_INGEST = TRUE
    AS
    COPY INTO SALES
    FROM @S3_STAGE/incoming/
    FILE_FORMAT = CSV_FORMAT;

-- Show pipes
SHOW PIPES;

-- Get pipe status
SELECT SYSTEM$PIPE_STATUS('SALES_PIPE');

-- Manually trigger pipe
ALTER PIPE SALES_PIPE REFRESH;

-- Pause pipe
ALTER PIPE SALES_PIPE SET PIPE_EXECUTION_PAUSED = TRUE;

-- Resume pipe
ALTER PIPE SALES_PIPE SET PIPE_EXECUTION_PAUSED = FALSE;

-- Check pipe history
SELECT * FROM TABLE(INFORMATION_SCHEMA.PIPE_USAGE_HISTORY(
    DATE_RANGE_START => DATEADD('days', -1, CURRENT_DATE()),
    PIPE_NAME => 'SALES_PIPE'
));
```

**Auto-Ingest Setup (AWS S3):**
1. Create SQS queue in AWS
2. Configure S3 bucket event notification
3. Create Snowpipe with AUTO_INGEST
4. Snowflake listens to SQS queue
5. Files auto-load when added to S3

---

### 4.3 External Tables

**Query external data without loading:**

```sql
-- Create external table
CREATE EXTERNAL TABLE EXT_SALES
    WITH LOCATION = @S3_STAGE/sales/
    FILE_FORMAT = PARQUET_FORMAT
    AUTO_REFRESH = TRUE
    PATTERN = '.*[.]parquet';

-- Query external table (Snowflake reads directly from S3)
SELECT * FROM EXT_SALES
WHERE SALE_DATE >= '2024-01-01';

-- Refresh metadata
ALTER EXTERNAL TABLE EXT_SALES REFRESH;

-- Add partition columns for performance
ALTER EXTERNAL TABLE EXT_SALES
    ADD PARTITION COLUMN YEAR INT AS SPLIT_PART(METADATA$FILENAME, '/', 2);
```

**When to use External Tables:**
✓ Query data rarely
✓ Data owned by another team/system
✓ Avoid storage costs in Snowflake
✓ Data governance requires external storage

---

## 🗂️ LEVEL 5: SEMI-STRUCTURED DATA

### 5.1 VARIANT Data Type

**What is VARIANT?**
- Universal data type for JSON, Avro, ORC, Parquet, XML
- Stores data in optimized format
- Query with dot notation or bracket notation
- Up to 16 MB per value

**Working with JSON:**

```sql
-- Create table with VARIANT column
CREATE TABLE EVENTS (
    EVENT_ID INT,
    EVENT_DATA VARIANT,
    CREATED_AT TIMESTAMP
);

-- Insert JSON data
INSERT INTO EVENTS (EVENT_ID, EVENT_DATA, CREATED_AT)
SELECT 
    1,
    PARSE_JSON('{
        "user_id": 123,
        "action": "purchase",
        "items": [
            {"product_id": 456, "quantity": 2},
            {"product_id": 789, "quantity": 1}
        ],
        "total": 99.99
    }'),
    CURRENT_TIMESTAMP();

-- Query JSON with dot notation
SELECT 
    EVENT_ID,
    EVENT_DATA:user_id::INT AS user_id,
    EVENT_DATA:action::STRING AS action,
    EVENT_DATA:total::FLOAT AS total,
    EVENT_DATA:items[0]:product_id::INT AS first_product
FROM EVENTS;

-- Flatten nested arrays
SELECT 
    e.EVENT_ID,
    f.value:product_id::INT AS product_id,
    f.value:quantity::INT AS quantity
FROM EVENTS e,
LATERAL FLATTEN(input => e.EVENT_DATA:items) f;
```

---

### 5.2 Advanced JSON Functions

```sql
-- FLATTEN for arrays/objects
SELECT 
    f.value:name::STRING AS feature_name,
    f.value:enabled::BOOLEAN AS is_enabled
FROM EVENTS,
LATERAL FLATTEN(input => EVENT_DATA:features) f;

-- Nested FLATTEN
SELECT 
    f1.value:category::STRING AS category,
    f2.value::STRING AS tag
FROM EVENTS,
LATERAL FLATTEN(input => EVENT_DATA:categories) f1,
LATERAL FLATTEN(input => f1.value:tags) f2;

-- Check if key exists
SELECT 
    EVENT_DATA,
    IFF(EVENT_DATA:metadata IS NOT NULL, 'Has metadata', 'No metadata') AS status
FROM EVENTS;

-- Get all keys
SELECT OBJECT_KEYS(EVENT_DATA) FROM EVENTS;

-- Array functions
SELECT 
    ARRAY_SIZE(EVENT_DATA:items) AS item_count,
    ARRAY_CONTAINS('purchase'::VARIANT, EVENT_DATA:tags) AS has_purchase_tag
FROM EVENTS;
```

---

### 5.3 Creating Structured Views from JSON

```sql
-- Create a view to make JSON queryable
CREATE VIEW EVENTS_STRUCTURED AS
SELECT 
    EVENT_ID,
    EVENT_DATA:user_id::INT AS user_id,
    EVENT_DATA:action::STRING AS action,
    EVENT_DATA:timestamp::TIMESTAMP AS event_timestamp,
    EVENT_DATA:total::DECIMAL(10,2) AS total,
    EVENT_DATA:metadata:device::STRING AS device,
    EVENT_DATA:metadata:location:city::STRING AS city
FROM EVENTS;

-- Now query like a normal table
SELECT 
    action,
    COUNT(*) AS event_count,
    SUM(total) AS total_revenue
FROM EVENTS_STRUCTURED
GROUP BY action;
```

---

## 🚀 LEVEL 6: ADVANCED FEATURES

### 6.1 Time Travel

**Query historical data:**

```sql
-- Query table as it was 1 hour ago
SELECT * FROM CUSTOMERS
AT(OFFSET => -3600);  -- 3600 seconds = 1 hour

-- Query specific timestamp
SELECT * FROM CUSTOMERS
AT(TIMESTAMP => '2024-02-01 10:00:00'::TIMESTAMP);

-- Query before specific statement
SELECT * FROM CUSTOMERS
BEFORE(STATEMENT => '01a12b34-5678-90cd-ef12-3456789abcde');

-- Compare current vs 1 day ago
SELECT 
    current.customer_id,
    current.status AS current_status,
    historical.status AS status_yesterday
FROM CUSTOMERS current
LEFT JOIN CUSTOMERS AT(OFFSET => -86400) historical
    ON current.customer_id = historical.customer_id
WHERE current.status != historical.status;
```

**Undrop objects:**

```sql
-- Undrop table
DROP TABLE CUSTOMERS;
UNDROP TABLE CUSTOMERS;

-- Undrop database
DROP DATABASE MY_DB;
UNDROP DATABASE MY_DB;

-- Undrop schema
DROP SCHEMA MY_SCHEMA;
UNDROP SCHEMA MY_SCHEMA;

-- Show dropped objects
SHOW TABLES HISTORY;
```

**Set retention period:**

```sql
-- Account level (max 90 days for Enterprise)
ALTER ACCOUNT SET DATA_RETENTION_TIME_IN_DAYS = 90;

-- Database level
ALTER DATABASE MY_DB SET DATA_RETENTION_TIME_IN_DAYS = 30;

-- Table level
ALTER TABLE CUSTOMERS SET DATA_RETENTION_TIME_IN_DAYS = 7;
```

---

### 6.2 Zero-Copy Cloning

**Instant, free copies:**

```sql
-- Clone a table
CREATE TABLE CUSTOMERS_BACKUP CLONE CUSTOMERS;

-- Clone with time travel
CREATE TABLE CUSTOMERS_YESTERDAY CLONE CUSTOMERS
AT(OFFSET => -86400);

-- Clone a schema
CREATE SCHEMA ANALYTICS_DEV CLONE ANALYTICS_PROD;

-- Clone entire database
CREATE DATABASE PROD_BACKUP CLONE PRODUCTION;

-- Clones are independent after creation
-- Changes to clone don't affect original
INSERT INTO CUSTOMERS_BACKUP VALUES (...);  -- Only in backup
```

**Use Cases:**
✓ Instant dev/test environments
✓ Zero-cost backups
✓ Experimentation without risk
✓ Data versioning

---

### 6.3 Streams (Change Data Capture)

**Track DML changes:**

```sql
-- Create stream on table
CREATE STREAM CUSTOMERS_STREAM ON TABLE CUSTOMERS;

-- View changes
SELECT * FROM CUSTOMERS_STREAM;

-- Columns in stream:
-- METADATA$ACTION: INSERT, DELETE, UPDATE
-- METADATA$ISUPDATE: TRUE for updates
-- METADATA$ROW_ID: Unique row identifier

-- Process changes
BEGIN;
    INSERT INTO CUSTOMER_HISTORY
    SELECT * FROM CUSTOMERS_STREAM;
    
    -- Stream advances after consumption
COMMIT;

-- Stream is now empty (changes consumed)
SELECT * FROM CUSTOMERS_STREAM;  -- 0 rows

-- Stream types
CREATE STREAM S1 ON TABLE T1;  -- Standard
CREATE STREAM S2 ON TABLE T2 APPEND_ONLY = TRUE;  -- Only inserts
CREATE STREAM S3 ON VIEW V1;  -- On view
CREATE STREAM S4 ON EXTERNAL TABLE ET1;  -- On external table
```

---

### 6.4 Tasks (Scheduling)

**Automate SQL execution:**

```sql
-- Simple task (runs every hour)
CREATE TASK HOURLY_AGGREGATION
    WAREHOUSE = COMPUTE_WH
    SCHEDULE = '60 MINUTE'
AS
    INSERT INTO DAILY_SUMMARY
    SELECT 
        CURRENT_DATE() AS summary_date,
        COUNT(*) AS total_orders,
        SUM(amount) AS total_revenue
    FROM ORDERS
    WHERE order_date = CURRENT_DATE();

-- Task with cron syntax
CREATE TASK DAILY_CLEANUP
    WAREHOUSE = COMPUTE_WH
    SCHEDULE = 'USING CRON 0 2 * * * America/Los_Angeles'  -- 2 AM daily
AS
    DELETE FROM LOGS WHERE log_date < DATEADD(days, -30, CURRENT_DATE());

-- Task triggered by stream (when data available)
CREATE TASK PROCESS_NEW_CUSTOMERS
    WAREHOUSE = COMPUTE_WH
    SCHEDULE = '5 MINUTE'
    WHEN SYSTEM$STREAM_HAS_DATA('CUSTOMERS_STREAM')
AS
    INSERT INTO CUSTOMER_PROCESSED
    SELECT * FROM CUSTOMERS_STREAM;

-- Resume/suspend tasks
ALTER TASK HOURLY_AGGREGATION RESUME;
ALTER TASK HOURLY_AGGREGATION SUSPEND;

-- Show tasks
SHOW TASKS;

-- Task history
SELECT * FROM TABLE(INFORMATION_SCHEMA.TASK_HISTORY(
    SCHEDULED_TIME_RANGE_START => DATEADD('hours', -24, CURRENT_TIMESTAMP())
));
```

**Task Trees (Dependencies):**

```sql
-- Parent task
CREATE TASK PARENT_TASK
    WAREHOUSE = COMPUTE_WH
    SCHEDULE = '60 MINUTE'
AS
    INSERT INTO STAGE_1 SELECT ...;

-- Child task (runs after parent)
CREATE TASK CHILD_TASK_1
    WAREHOUSE = COMPUTE_WH
    AFTER PARENT_TASK
AS
    INSERT INTO STAGE_2 SELECT * FROM STAGE_1;

CREATE TASK CHILD_TASK_2
    WAREHOUSE = COMPUTE_WH
    AFTER PARENT_TASK
AS
    INSERT INTO STAGE_3 SELECT * FROM STAGE_1;

-- Resume in order (child first!)
ALTER TASK CHILD_TASK_1 RESUME;
ALTER TASK CHILD_TASK_2 RESUME;
ALTER TASK PARENT_TASK RESUME;
```

---

### 6.5 Stored Procedures (JavaScript/Python/SQL)

**JavaScript Procedure:**

```sql
CREATE OR REPLACE PROCEDURE UPDATE_CUSTOMER_TIER()
RETURNS VARCHAR
LANGUAGE JAVASCRIPT
AS
$$
    var sql_command = `
        UPDATE CUSTOMERS
        SET tier = CASE
            WHEN total_purchases > 10000 THEN 'PLATINUM'
            WHEN total_purchases > 5000 THEN 'GOLD'
            WHEN total_purchases > 1000 THEN 'SILVER'
            ELSE 'BRONZE'
        END
    `;
    
    var stmt = snowflake.createStatement({sqlText: sql_command});
    var result = stmt.execute();
    
    return 'Customer tiers updated successfully';
$$;

-- Call procedure
CALL UPDATE_CUSTOMER_TIER();
```

**Python Procedure (Snowpark):**

```sql
CREATE OR REPLACE PROCEDURE CALCULATE_METRICS(START_DATE DATE, END_DATE DATE)
RETURNS TABLE(metric_name VARCHAR, metric_value FLOAT)
LANGUAGE PYTHON
RUNTIME_VERSION = '3.8'
PACKAGES = ('snowflake-snowpark-python', 'pandas')
HANDLER = 'calculate'
AS
$$
def calculate(session, start_date, end_date):
    # Use Snowpark DataFrame API
    df = session.table('SALES')
    filtered = df.filter((df['SALE_DATE'] >= start_date) & (df['SALE_DATE'] <= end_date))
    
    metrics = [
        ('total_sales', filtered.count()),
        ('total_revenue', filtered.agg({'AMOUNT': 'sum'}).collect()[0][0])
    ]
    
    return session.create_dataframe(metrics, schema=['metric_name', 'metric_value'])
$$;

-- Call
CALL CALCULATE_METRICS('2024-01-01', '2024-01-31');
```

---

### 6.6 User-Defined Functions (UDFs)

**SQL UDF:**

```sql
-- Simple scalar function
CREATE FUNCTION CALCULATE_TAX(AMOUNT FLOAT, TAX_RATE FLOAT)
RETURNS FLOAT
AS
$$
    AMOUNT * TAX_RATE
$$;

-- Use it
SELECT 
    product_name,
    price,
    CALCULATE_TAX(price, 0.08) AS tax
FROM PRODUCTS;

-- Table function
CREATE FUNCTION GET_EMPLOYEE_HIERARCHY(EMP_ID INT)
RETURNS TABLE (employee_id INT, level INT)
AS
$$
    WITH RECURSIVE hierarchy AS (
        SELECT employee_id, 1 as level
        FROM employees
        WHERE employee_id = EMP_ID
        
        UNION ALL
        
        SELECT e.employee_id, h.level + 1
        FROM employees e
        JOIN hierarchy h ON e.manager_id = h.employee_id
    )
    SELECT * FROM hierarchy
$$;

-- Use it
SELECT * FROM TABLE(GET_EMPLOYEE_HIERARCHY(100));
```

**JavaScript UDF:**

```sql
CREATE OR REPLACE FUNCTION PARSE_USER_AGENT(UA VARCHAR)
RETURNS VARIANT
LANGUAGE JAVASCRIPT
AS
$$
    var parser = {
        browser: null,
        os: null
    };
    
    if (UA.includes('Chrome')) parser.browser = 'Chrome';
    else if (UA.includes('Firefox')) parser.browser = 'Firefox';
    else if (UA.includes('Safari')) parser.browser = 'Safari';
    
    if (UA.includes('Windows')) parser.os = 'Windows';
    else if (UA.includes('Mac')) parser.os = 'Mac';
    else if (UA.includes('Linux')) parser.os = 'Linux';
    
    return parser;
$$;

-- Use it
SELECT 
    user_agent,
    PARSE_USER_AGENT(user_agent):browser::STRING AS browser,
    PARSE_USER_AGENT(user_agent):os::STRING AS os
FROM WEB_LOGS;
```

---

## ⚡ LEVEL 7: PERFORMANCE OPTIMIZATION

### 7.1 Query Profiling

**Using Query Profile:**

```sql
-- Run a query
SELECT 
    customer_id,
    COUNT(*) AS order_count,
    SUM(amount) AS total_spent
FROM ORDERS
GROUP BY customer_id;

-- View query profile in Snowflake UI:
-- History → Select Query → Query Profile

-- Key metrics to check:
-- 1. Bytes scanned (lower is better - shows pruning effectiveness)
-- 2. Partitions scanned vs total (fewer is better)
-- 3. Spilling to disk (should be zero or minimal)
-- 4. Queued time (indicates warehouse too small)
```

**Query Tags for Tracking:**

```sql
-- Set query tag
ALTER SESSION SET QUERY_TAG = 'daily_etl_job';

-- Run queries
INSERT INTO ...;
UPDATE ...;

-- View queries by tag
SELECT *
FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY())
WHERE QUERY_TAG = 'daily_etl_job';

-- Unset tag
ALTER SESSION UNSET QUERY_TAG;
```

---

### 7.2 Optimization Techniques

**1. Partition Pruning:**

```sql
-- Bad: Function on column prevents pruning
SELECT * FROM SALES
WHERE YEAR(SALE_DATE) = 2024;

-- Good: Direct comparison enables pruning
SELECT * FROM SALES
WHERE SALE_DATE >= '2024-01-01' AND SALE_DATE < '2025-01-01';

-- Check pruning effectiveness
-- In Query Profile, look at "Partitions scanned" vs "Partitions total"
```

**2. Projection Pushdown:**

```sql
-- Bad: SELECT *
SELECT * FROM LARGE_TABLE WHERE id = 123;

-- Good: Only needed columns
SELECT id, name, amount FROM LARGE_TABLE WHERE id = 123;
```

**3. Predicate Pushdown:**

```sql
-- Bad: Filter after join
SELECT *
FROM ORDERS o
JOIN CUSTOMERS c ON o.customer_id = c.customer_id
WHERE o.order_date >= '2024-01-01';

-- Good: Filter before join (when possible)
SELECT *
FROM (SELECT * FROM ORDERS WHERE order_date >= '2024-01-01') o
JOIN CUSTOMERS c ON o.customer_id = c.customer_id;
```

**4. Result Caching:**

```sql
-- Snowflake caches query results for 24 hours
-- Identical queries return results instantly (free!)

-- Use result cache
SELECT COUNT(*) FROM LARGE_TABLE;  -- Takes 30 seconds

-- Run again immediately
SELECT COUNT(*) FROM LARGE_TABLE;  -- Returns instantly from cache

-- Bypass cache if needed
ALTER SESSION SET USE_CACHED_RESULT = FALSE;
```

**5. Clustering for Large Tables:**

```sql
-- Analyze current clustering
SELECT SYSTEM$CLUSTERING_INFORMATION('SALES', '(SALE_DATE, REGION)');

-- Add clustering key
ALTER TABLE SALES CLUSTER BY (SALE_DATE, REGION);

-- Enable automatic reclustering
ALTER TABLE SALES RESUME RECLUSTER;

-- Monitor clustering depth (0-4 is good, >100 needs reclustering)
SELECT SYSTEM$CLUSTERING_DEPTH('SALES', '(SALE_DATE)');
```

---

### 7.3 Materialized Views

**Pre-computed query results:**

```sql
-- Create materialized view
CREATE MATERIALIZED VIEW DAILY_SALES_SUMMARY AS
SELECT 
    DATE_TRUNC('day', order_date) AS sale_date,
    product_id,
    COUNT(*) AS order_count,
    SUM(amount) AS total_revenue
FROM ORDERS
GROUP BY DATE_TRUNC('day', order_date), product_id;

-- Query materialized view (instant results!)
SELECT * FROM DAILY_SALES_SUMMARY
WHERE sale_date = '2024-02-01';

-- Snowflake auto-maintains materialized views
-- When base table updates, view updates automatically

-- Show materialized views
SHOW MATERIALIZED VIEWS;

-- Suspend/resume background maintenance
ALTER MATERIALIZED VIEW DAILY_SALES_SUMMARY SUSPEND;
ALTER MATERIALIZED VIEW DAILY_SALES_SUMMARY RESUME;
```

**When to use:**
✓ Complex aggregations queried frequently
✓ Joins of large tables
✓ Views queried by many users
✓ Dashboard queries

**When NOT to use:**
✗ Base table changes constantly
✗ View rarely queried
✗ Simple queries (not worth maintenance cost)

---

### 7.4 Search Optimization Service

**Speed up point lookups and substring searches:**

```sql
-- Enable search optimization
ALTER TABLE CUSTOMERS ADD SEARCH OPTIMIZATION;

-- Check status
SELECT PARSE_JSON(SYSTEM$GET_SEARCH_OPTIMIZATION_STATUS('CUSTOMERS'));

-- Queries that benefit:
SELECT * FROM CUSTOMERS WHERE email = 'john@example.com';  -- Point lookup
SELECT * FROM CUSTOMERS WHERE name LIKE '%Smith%';         -- Substring
SELECT * FROM CUSTOMERS WHERE id IN (1, 2, 3, 100);       -- IN list

-- Remove search optimization
ALTER TABLE CUSTOMERS DROP SEARCH OPTIMIZATION;
```

**Cost vs Benefit:**
- Costs credits to build/maintain index
- Dramatically speeds up selective queries
- Best for tables with selective predicates
- Not for full table scans

---

## 🔒 LEVEL 8: SECURITY & GOVERNANCE

### 8.1 Role-Based Access Control (RBAC)

**Snowflake Role Hierarchy:**

```
ACCOUNTADMIN (highest privilege)
    ↓
SECURITYADMIN (manage users/roles)
    ↓
SYSADMIN (create objects)
    ↓
PUBLIC (default, all users)
    ↓
Custom Roles
```

**Creating Roles and Users:**

```sql
-- Create roles
CREATE ROLE DATA_ANALYST;
CREATE ROLE DATA_ENGINEER;
CREATE ROLE DATA_SCIENTIST;

-- Grant privileges to roles
GRANT USAGE ON WAREHOUSE ANALYTICS_WH TO ROLE DATA_ANALYST;
GRANT USAGE ON DATABASE ANALYTICS_DB TO ROLE DATA_ANALYST;
GRANT USAGE ON SCHEMA ANALYTICS_DB.PUBLIC TO ROLE DATA_ANALYST;
GRANT SELECT ON ALL TABLES IN SCHEMA ANALYTICS_DB.PUBLIC TO ROLE DATA_ANALYST;

-- Future grants (for new objects)
GRANT SELECT ON FUTURE TABLES IN SCHEMA ANALYTICS_DB.PUBLIC TO ROLE DATA_ANALYST;

-- Create user
CREATE USER john_doe
    PASSWORD = 'SecurePassword123!'
    DEFAULT_ROLE = DATA_ANALYST
    DEFAULT_WAREHOUSE = ANALYTICS_WH
    MUST_CHANGE_PASSWORD = TRUE;

-- Grant role to user
GRANT ROLE DATA_ANALYST TO USER john_doe;

-- Role hierarchy
GRANT ROLE DATA_ANALYST TO ROLE DATA_ENGINEER;  -- Engineer can use Analyst role

-- Switch role
USE ROLE DATA_ANALYST;

-- Show current role
SELECT CURRENT_ROLE();
```

---

### 8.2 Row-Level Security

**Secure views with row filtering:**

```sql
-- Create mapping table
CREATE TABLE USER_REGION_ACCESS (
    username VARCHAR,
    region VARCHAR
);

INSERT INTO USER_REGION_ACCESS VALUES
    ('john_doe', 'US_EAST'),
    ('jane_smith', 'US_WEST'),
    ('admin_user', 'ALL');

-- Create secure view with row-level security
CREATE SECURE VIEW SALES_SECURE AS
SELECT s.*
FROM SALES s
JOIN USER_REGION_ACCESS ura 
    ON s.region = ura.region 
    OR ura.region = 'ALL'
WHERE ura.username = CURRENT_USER();

-- Users only see their allowed rows
-- john_doe: only US_EAST sales
-- jane_smith: only US_WEST sales
-- admin_user: all sales

-- Grant access
GRANT SELECT ON VIEW SALES_SECURE TO ROLE DATA_ANALYST;
```

---

### 8.3 Column-Level Security

**Dynamic Data Masking:**

```sql
-- Create masking policy
CREATE MASKING POLICY EMAIL_MASK AS (VAL STRING) 
RETURNS STRING ->
  CASE
    WHEN CURRENT_ROLE() IN ('ADMIN', 'COMPLIANCE') THEN VAL  -- Full access
    ELSE REGEXP_REPLACE(VAL, '.+@', '****@')  -- Mask for others
  END;

-- Apply to column
ALTER TABLE CUSTOMERS MODIFY COLUMN EMAIL 
SET MASKING POLICY EMAIL_MASK;

-- Test
USE ROLE DATA_ANALYST;
SELECT email FROM CUSTOMERS;  -- Shows: ****@example.com

USE ROLE ADMIN;
SELECT email FROM CUSTOMERS;  -- Shows: john@example.com

-- Remove masking policy
ALTER TABLE CUSTOMERS MODIFY COLUMN EMAIL 
UNSET MASKING POLICY;
```

**Row Access Policies:**

```sql
-- Create row access policy
CREATE ROW ACCESS POLICY REGION_POLICY AS (REGION VARCHAR) 
RETURNS BOOLEAN ->
  CASE
    WHEN CURRENT_ROLE() = 'ADMIN' THEN TRUE
    WHEN CURRENT_ROLE() = 'US_ANALYST' AND REGION = 'US' THEN TRUE
    WHEN CURRENT_ROLE() = 'EU_ANALYST' AND REGION = 'EU' THEN TRUE
    ELSE FALSE
  END;

-- Apply to table
ALTER TABLE SALES ADD ROW ACCESS POLICY REGION_POLICY ON (REGION);
```

---

### 8.4 Data Sharing

**Share data without copying:**

```sql
-- Create share (as data provider)
CREATE SHARE SALES_SHARE;

-- Grant usage on database
GRANT USAGE ON DATABASE ANALYTICS_DB TO SHARE SALES_SHARE;

-- Grant usage on schema
GRANT USAGE ON SCHEMA ANALYTICS_DB.PUBLIC TO SHARE SALES_SHARE;

-- Grant select on specific tables
GRANT SELECT ON TABLE ANALYTICS_DB.PUBLIC.SALES TO SHARE SALES_SHARE;
GRANT SELECT ON TABLE ANALYTICS_DB.PUBLIC.PRODUCTS TO SHARE SALES_SHARE;

-- Add accounts to share
ALTER SHARE SALES_SHARE ADD ACCOUNTS = xy12345, ab67890;

-- Show shares
SHOW SHARES;

-- As data consumer:
-- View available shares
SHOW SHARES;

-- Create database from share
CREATE DATABASE SHARED_SALES_DATA FROM SHARE provider_account.SALES_SHARE;

-- Query shared data
SELECT * FROM SHARED_SALES_DATA.PUBLIC.SALES;
```

**Secure Data Sharing:**
- No data copying
- Real-time access
- Provider controls access
- Consumer pays for compute only
- No ETL pipelines needed

---

## 💰 LEVEL 9: COST OPTIMIZATION

### 9.1 Understanding Snowflake Pricing

**What You Pay For:**

1. **Compute (Virtual Warehouses)**
   - Charged per-second (60 second minimum)
   - Based on warehouse size
   - Different rates for different cloud platforms

2. **Storage**
   - Compressed data storage
   - Time Travel storage
   - Fail-Safe storage (7 days automatic)

3. **Cloud Services**
   - Metadata operations
   - Authentication
   - Query optimization
   - Free up to 10% of compute spend

**Credit Consumption:**

| Warehouse Size | Credits/Hour |
|----------------|--------------|
| X-Small | 1 |
| Small | 2 |
| Medium | 4 |
| Large | 8 |
| X-Large | 16 |
| 2X-Large | 32 |

---

### 9.2 Cost Optimization Strategies

**1. Warehouse Auto-Suspend:**

```sql
-- Set aggressive auto-suspend
ALTER WAREHOUSE ANALYTICS_WH SET AUTO_SUSPEND = 60;  -- 1 minute

-- For BI tools with sporadic queries
ALTER WAREHOUSE BI_WH SET AUTO_SUSPEND = 300;  -- 5 minutes

-- For continuous ETL
ALTER WAREHOUSE ETL_WH SET AUTO_SUSPEND = 600;  -- 10 minutes
```

**2. Right-Size Warehouses:**

```sql
-- Start small, scale up if needed
CREATE WAREHOUSE EXPERIMENTAL_WH
    WAREHOUSE_SIZE = 'XSMALL'
    AUTO_SUSPEND = 60;

-- Monitor query performance
SELECT 
    warehouse_name,
    AVG(execution_time) / 1000 AS avg_seconds,
    AVG(queued_overload_time) / 1000 AS avg_queue_seconds
FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY(
    DATE_RANGE_START => DATEADD('days', -7, CURRENT_DATE())
))
GROUP BY warehouse_name;

-- If queries are queuing, scale up
-- If queries run fast with warehouse mostly idle, scale down
```

**3. Use Multi-Cluster Wisely:**

```sql
-- Economy mode: slower to scale, saves money
CREATE WAREHOUSE BI_WH
    WAREHOUSE_SIZE = 'MEDIUM'
    MIN_CLUSTER_COUNT = 1
    MAX_CLUSTER_COUNT = 5
    SCALING_POLICY = 'ECONOMY'  -- Wait before adding clusters
    AUTO_SUSPEND = 300;

-- Standard mode: faster scaling, higher cost
ALTER WAREHOUSE BI_WH SET SCALING_POLICY = 'STANDARD';
```

**4. Optimize Storage:**

```sql
-- Reduce time travel retention for temporary tables
ALTER TABLE STAGING_TABLE SET DATA_RETENTION_TIME_IN_DAYS = 0;

-- Drop unused tables/databases
DROP TABLE IF EXISTS OLD_BACKUP;

-- Monitor storage costs
SELECT 
    table_catalog AS database_name,
    table_schema AS schema_name,
    table_name,
    ROUND(active_bytes / (1024*1024*1024), 2) AS active_gb,
    ROUND(time_travel_bytes / (1024*1024*1024), 2) AS time_travel_gb,
    ROUND(failsafe_bytes / (1024*1024*1024), 2) AS failsafe_gb
FROM INFORMATION_SCHEMA.TABLE_STORAGE_METRICS
WHERE table_catalog = 'MY_DATABASE'
ORDER BY active_bytes DESC;
```

**5. Monitor Credit Usage:**

```sql
-- Warehouse credit usage (last 7 days)
SELECT 
    warehouse_name,
    SUM(credits_used) AS total_credits
FROM TABLE(INFORMATION_SCHEMA.WAREHOUSE_METERING_HISTORY(
    DATE_RANGE_START => DATEADD('days', -7, CURRENT_DATE())
))
GROUP BY warehouse_name
ORDER BY total_credits DESC;

-- Daily credit usage trend
SELECT 
    DATE(start_time) AS usage_date,
    warehouse_name,
    SUM(credits_used) AS daily_credits
FROM TABLE(INFORMATION_SCHEMA.WAREHOUSE_METERING_HISTORY(
    DATE_RANGE_START => DATEADD('days', -30, CURRENT_DATE())
))
GROUP BY DATE(start_time), warehouse_name
ORDER BY usage_date DESC, daily_credits DESC;

-- Storage costs
SELECT 
    DATE_TRUNC('month', usage_date) AS month,
    AVG(storage_bytes + stage_bytes + failsafe_bytes) / (1024*1024*1024*1024) AS avg_tb,
    AVG(storage_bytes + stage_bytes + failsafe_bytes) / (1024*1024*1024*1024) * 23 AS estimated_monthly_cost_usd
FROM TABLE(INFORMATION_SCHEMA.STORAGE_USAGE(
    DATE_RANGE_START => DATEADD('months', -3, CURRENT_DATE())
))
GROUP BY DATE_TRUNC('month', usage_date);
```

**6. Resource Monitors:**

```sql
-- Create resource monitor
CREATE RESOURCE MONITOR MONTHLY_LIMIT
    WITH CREDIT_QUOTA = 1000          -- 1000 credits per month
    FREQUENCY = MONTHLY
    START_TIMESTAMP = '2024-02-01 00:00'
    TRIGGERS
        ON 75 PERCENT DO NOTIFY         -- Alert at 75%
        ON 90 PERCENT DO SUSPEND        -- Suspend at 90%
        ON 100 PERCENT DO SUSPEND_IMMEDIATE;  -- Hard stop at 100%

-- Assign to account
ALTER ACCOUNT SET RESOURCE_MONITOR = MONTHLY_LIMIT;

-- Assign to specific warehouse
ALTER WAREHOUSE ANALYTICS_WH SET RESOURCE_MONITOR = MONTHLY_LIMIT;
```

---

### 9.3 Cost Optimization Best Practices

✓ **Separate warehouses** by workload (ETL, BI, Data Science)
✓ **Auto-suspend** aggressively (60-300 seconds)
✓ **Start small** (X-Small/Small), scale up only when needed
✓ **Use multi-cluster** only when concurrent users > 1 warehouse can handle
✓ **Cluster tables** only when proven benefit
✓ **Reduce time travel** for non-critical tables
✓ **Drop unused objects** regularly
✓ **Use transient/temporary** tables for staging
✓ **Monitor daily** - catch runaway queries early
✓ **Educate users** - developers can waste credits easily

---

## 🏗️ PRACTICE PROJECTS

### Project 1: E-Commerce Data Warehouse

**Objective:** Build complete DWH with all Snowflake features

**Steps:**

1. **Setup Structure:**
```sql
-- Create databases
CREATE DATABASE ECOM_RAW;
CREATE DATABASE ECOM_TRANSFORMED;
CREATE DATABASE ECOM_ANALYTICS;

-- Create warehouses
CREATE WAREHOUSE LOADING_WH SIZE = SMALL AUTO_SUSPEND = 60;
CREATE WAREHOUSE TRANSFORM_WH SIZE = MEDIUM AUTO_SUSPEND = 300;
CREATE WAREHOUSE ANALYTICS_WH SIZE = SMALL AUTO_SUSPEND = 60;
```

2. **Data Loading:**
```sql
-- Create stages for raw data
CREATE STAGE S3_ORDERS URL = 's3://my-bucket/orders/';
CREATE STAGE S3_CUSTOMERS URL = 's3://my-bucket/customers/';

-- Setup Snowpipe for continuous loading
CREATE PIPE ORDERS_PIPE AUTO_INGEST = TRUE AS
COPY INTO ECOM_RAW.PUBLIC.ORDERS FROM @S3_ORDERS;
```

3. **Transformations:**
```sql
-- Create streams for CDC
CREATE STREAM ORDERS_STREAM ON TABLE ECOM_RAW.PUBLIC.ORDERS;

-- Create tasks for transformations
CREATE TASK TRANSFORM_ORDERS
    WAREHOUSE = TRANSFORM_WH
    SCHEDULE = '5 MINUTE'
    WHEN SYSTEM$STREAM_HAS_DATA('ORDERS_STREAM')
AS
    MERGE INTO ECOM_TRANSFORMED.PUBLIC.ORDERS_CLEAN target
    USING ORDERS_STREAM source
    ON target.order_id = source.order_id
    WHEN MATCHED THEN UPDATE SET ...
    WHEN NOT MATCHED THEN INSERT ...;
```

4. **Analytics Layer:**
```sql
-- Create materialized views for dashboards
CREATE MATERIALIZED VIEW DAILY_REVENUE AS
SELECT 
    DATE_TRUNC('day', order_date) AS day,
    SUM(amount) AS revenue,
    COUNT(DISTINCT customer_id) AS customers
FROM ECOM_TRANSFORMED.PUBLIC.ORDERS_CLEAN
GROUP BY DATE_TRUNC('day', order_date);
```

5. **Security:**
```sql
-- Create roles
CREATE ROLE ANALYST;
CREATE ROLE DATA_ENGINEER;

-- Grant appropriate access
GRANT USAGE ON WAREHOUSE ANALYTICS_WH TO ROLE ANALYST;
GRANT SELECT ON ALL VIEWS IN DATABASE ECOM_ANALYTICS TO ROLE ANALYST;
```

---

### Project 2: Real-Time Event Processing

**Build streaming analytics pipeline:**

1. Ingest JSON events via Snowpipe
2. Parse semi-structured data
3. Create real-time aggregations
4. Build dashboard views

---

### Project 3: Data Sharing Platform

**Setup data sharing:**

1. Prepare clean, governed datasets
2. Create secure shares
3. Implement row-level security
4. Share with external accounts

---

## 🎤 INTERVIEW PREPARATION

### Common Snowflake Interview Questions:

**1. Explain Snowflake's architecture**
- Three-layer architecture
- Separation of storage and compute
- Shared-disk and shared-nothing benefits

**2. What is Time Travel? How does it work?**
- Query historical data
- Retention periods (0-90 days)
- Use cases: recover data, audit changes

**3. Difference between Snowflake and traditional data warehouses?**
- No indexes/tuning needed
- Instant elasticity
- Zero-copy cloning
- Native semi-structured support

**4. How does Snowflake pricing work?**
- Compute credits (per-second billing)
- Storage (compressed, monthly)
- Cloud services (usually free)

**5. What are micro-partitions?**
- Automatic partitioning (50-500MB)
- Immutable, columnar
- Metadata for pruning

**6. When would you use clustering?**
- Very large tables (TB+)
- Frequent queries on same columns
- Pruning not effective

**7. Explain virtual warehouses**
- Compute clusters
- Scale up vs scale out
- Auto-suspend/resume

**8. What is zero-copy cloning?**
- Instant table/schema/DB copies
- No storage cost initially
- Use cases: dev/test, backups

**9. How do you optimize Snowflake costs?**
- Right-size warehouses
- Auto-suspend
- Monitor usage
- Resource monitors

**10. What are Snowflake Stages?**
- Internal vs external
- User/table/named stages
- Used for loading data

---

## ✅ SNOWFLAKE MASTERY CHECKLIST

### Beginner ✓
- [ ] Understand architecture (3 layers)
- [ ] Create databases, schemas, tables
- [ ] Load data via COPY INTO
- [ ] Write basic SQL queries
- [ ] Use virtual warehouses

### Intermediate ✓
- [ ] Setup Snowpipe for continuous loading
- [ ] Work with semi-structured data (JSON)
- [ ] Use Time Travel and cloning
- [ ] Create and use stages
- [ ] Understand file formats

### Advanced ✓
- [ ] Implement Streams and Tasks
- [ ] Create stored procedures (JS/Python)
- [ ] Use materialized views
- [ ] Setup data sharing
- [ ] Implement row/column security

### Expert ✓
- [ ] Optimize query performance
- [ ] Design cost-effective solutions
- [ ] Implement clustering strategies
- [ ] Build end-to-end pipelines
- [ ] Architect multi-environment setups

---

## 📚 LEARNING RESOURCES

**Official Snowflake:**
- Snowflake Documentation: https://docs.snowflake.com
- Hands-On Lab: https://quickstarts.snowflake.com
- Snowflake University (free courses)
- Community Forums

**Practice:**
- 30-day free trial (full features!)
- Sample datasets available
- Hands-on labs

**Certifications:**
- SnowPro Core (foundational)
- SnowPro Advanced: Architect
- SnowPro Advanced: Data Engineer

---

## 🎯 30-DAY LEARNING PLAN

**Week 1: Foundations**
- Days 1-2: Architecture, setup trial account
- Days 3-4: Create objects, load data (COPY INTO)
- Days 5-7: SQL practice, virtual warehouses

**Week 2: Advanced Loading**
- Days 8-10: Stages, file formats, Snowpipe
- Days 11-12: Semi-structured data (JSON)
- Days 13-14: Time Travel, cloning

**Week 3: Automation & Advanced**
- Days 15-17: Streams and Tasks
- Days 18-19: Stored procedures
- Days 20-21: Materialized views

**Week 4: Production Skills**
- Days 22-24: Security (RBAC, masking)
- Days 25-27: Performance optimization
- Days 28-30: Cost optimization, monitoring

---

**You've mastered Snowflake when:**
✓ You can design a complete data warehouse architecture
✓ You understand when to use each feature
✓ You can optimize for performance AND cost
✓ You can implement security best practices
✓ You can troubleshoot production issues
✓ You're comfortable with semi-structured data

**Next Steps:**
- Get SnowPro Core certified
- Build a real project
- Practice cost optimization
- Learn Snowpark (Python/Java/Scala)

Ready to move to the next topic? 🚀
