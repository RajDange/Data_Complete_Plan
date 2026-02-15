# 🌬️ APACHE AIRFLOW MASTERY - Complete Learning Guide

## 📋 Table of Contents
1. [Airflow Fundamentals](#fundamentals)
2. [Architecture Deep Dive](#architecture)
3. [DAGs - Directed Acyclic Graphs](#dags)
4. [Operators & Tasks](#operators)
5. [Task Dependencies & Execution](#dependencies)
6. [XComs & Data Passing](#xcoms)
7. [Sensors & Triggers](#sensors)
8. [Scheduling & Backfilling](#scheduling)
9. [Production Best Practices](#production)
10. [Advanced Patterns](#advanced)
11. [Monitoring & Debugging](#monitoring)
12. [Real-World Projects](#projects)

---

## 🎯 LEVEL 1: AIRFLOW FUNDAMENTALS

### 1.1 What is Apache Airflow?

**Definition:**
Apache Airflow is an **orchestration platform** to programmatically author, schedule, and monitor workflows.

**Key Concepts:**
- **DAG** (Directed Acyclic Graph) - A workflow definition
- **Task** - A unit of work (run a Python function, execute SQL, etc.)
- **Operator** - Template for a task
- **Scheduler** - Triggers tasks based on schedule
- **Executor** - Determines how tasks run (local, parallel, distributed)
- **Worker** - Process that executes tasks

**Why Airflow?**
✓ **Programmatic** - Python code (not drag-and-drop)
✓ **Dynamic** - Generate pipelines programmatically
✓ **Extensible** - Custom operators, hooks, sensors
✓ **Scalable** - Run thousands of tasks
✓ **Rich UI** - Monitor, troubleshoot, backfill
✓ **Battle-tested** - Used by Airbnb, Spotify, Twitter, etc.

---

### 1.2 Airflow Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        WEB SERVER                            │
│                    (Flask UI - Port 8080)                    │
└─────────────────────────────────────────────────────────────┘
                            ▲
                            │
┌─────────────────────────────────────────────────────────────┐
│                        SCHEDULER                             │
│         (Reads DAGs, schedules tasks, sends to queue)        │
└─────────────────────────────────────────────────────────────┘
                            ▲
                            │
┌─────────────────────────────────────────────────────────────┐
│                    METADATA DATABASE                         │
│              (PostgreSQL/MySQL - Stores state)               │
└─────────────────────────────────────────────────────────────┘
                            ▲
                            │
┌─────────────────────────────────────────────────────────────┐
│                        EXECUTOR                              │
│         (LocalExecutor, CeleryExecutor, KubernetesExecutor)  │
└─────────────────────────────────────────────────────────────┘
                            ▲
                            │
┌─────────────────────────────────────────────────────────────┐
│                        WORKERS                               │
│              (Execute tasks, send results back)              │
└─────────────────────────────────────────────────────────────┘
                            ▲
                            │
┌─────────────────────────────────────────────────────────────┐
│                      DAG FILES                               │
│                  (/dags folder - Python)                     │
└─────────────────────────────────────────────────────────────┘
```

**Components Explained:**

1. **Web Server** - UI for monitoring, triggering, debugging
2. **Scheduler** - Heart of Airflow, triggers task instances
3. **Metadata Database** - Stores DAG runs, task instances, variables
4. **Executor** - Defines how tasks are executed
5. **Workers** - Execute the actual tasks
6. **DAG Files** - Python files defining workflows

---

### 1.3 Installation & Setup

**Quick Start (Local Development):**

```bash
# Install Airflow
pip install apache-airflow==2.8.1

# Or with constraints for compatibility
AIRFLOW_VERSION=2.8.1
PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"
pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"

# Initialize database
airflow db init

# Create admin user
airflow users create \
    --username admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com \
    --password admin

# Start web server (terminal 1)
airflow webserver --port 8080

# Start scheduler (terminal 2)
airflow scheduler
```

**Docker Compose (Recommended for Development):**

```yaml
# docker-compose.yaml
version: '3'
services:
  postgres:
    image: postgres:13
    environment:
      POSTGRES_USER: airflow
      POSTGRES_PASSWORD: airflow
      POSTGRES_DB: airflow
    volumes:
      - postgres-db-volume:/var/lib/postgresql/data

  airflow-webserver:
    image: apache/airflow:2.8.1
    depends_on:
      - postgres
    environment:
      AIRFLOW__CORE__EXECUTOR: LocalExecutor
      AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
      AIRFLOW__CORE__FERNET_KEY: ''
      AIRFLOW__CORE__DAGS_ARE_PAUSED_AT_CREATION: 'true'
      AIRFLOW__CORE__LOAD_EXAMPLES: 'false'
    volumes:
      - ./dags:/opt/airflow/dags
      - ./logs:/opt/airflow/logs
      - ./plugins:/opt/airflow/plugins
    ports:
      - "8080:8080"
    command: webserver

  airflow-scheduler:
    image: apache/airflow:2.8.1
    depends_on:
      - postgres
    environment:
      AIRFLOW__CORE__EXECUTOR: LocalExecutor
      AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
      AIRFLOW__CORE__FERNET_KEY: ''
    volumes:
      - ./dags:/opt/airflow/dags
      - ./logs:/opt/airflow/logs
      - ./plugins:/opt/airflow/plugins
    command: scheduler

volumes:
  postgres-db-volume:
```

```bash
# Start Airflow with Docker
docker-compose up -d

# Access UI: http://localhost:8080
# Username: airflow, Password: airflow
```

---

## 📊 LEVEL 2: DAGs (Directed Acyclic Graphs)

### 2.1 Your First DAG

**Basic DAG Structure:**

```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.operators.python import PythonOperator

# Default arguments for all tasks
default_args = {
    'owner': 'data_team',
    'depends_on_past': False,
    'email': ['alerts@company.com'],
    'email_on_failure': True,
    'email_on_retry': False,
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
    'execution_timeout': timedelta(hours=1),
}

# Define the DAG
dag = DAG(
    dag_id='my_first_dag',
    default_args=default_args,
    description='A simple tutorial DAG',
    schedule_interval='@daily',  # Run once per day
    start_date=datetime(2024, 1, 1),
    catchup=False,  # Don't run for past dates
    tags=['tutorial', 'example'],
)

# Define tasks
task1 = BashOperator(
    task_id='print_date',
    bash_command='date',
    dag=dag,
)

def print_hello():
    print("Hello from Airflow!")
    return "Success!"

task2 = PythonOperator(
    task_id='print_hello',
    python_callable=print_hello,
    dag=dag,
)

# Set dependencies
task1 >> task2  # task1 runs before task2
```

**Alternative Syntax (Recommended - TaskFlow API):**

```python
from datetime import datetime, timedelta
from airflow.decorators import dag, task

default_args = {
    'owner': 'data_team',
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
}

@dag(
    dag_id='my_first_dag_taskflow',
    default_args=default_args,
    description='DAG using TaskFlow API',
    schedule='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
    tags=['tutorial', 'taskflow'],
)
def my_etl_pipeline():
    
    @task
    def extract():
        """Extract data from source"""
        data = {'id': 1, 'name': 'John', 'value': 100}
        print(f"Extracted: {data}")
        return data
    
    @task
    def transform(data: dict):
        """Transform the data"""
        data['value'] = data['value'] * 2
        print(f"Transformed: {data}")
        return data
    
    @task
    def load(data: dict):
        """Load data to destination"""
        print(f"Loading: {data}")
        print("Data loaded successfully!")
    
    # Define pipeline
    data = extract()
    transformed_data = transform(data)
    load(transformed_data)

# Instantiate the DAG
my_etl_pipeline()
```

---

### 2.2 DAG Parameters Explained

**Essential DAG Parameters:**

```python
from airflow import DAG
from datetime import datetime, timedelta

dag = DAG(
    # REQUIRED
    dag_id='example_dag',                      # Unique identifier
    start_date=datetime(2024, 1, 1),           # When DAG can start running
    
    # SCHEDULING
    schedule_interval='@daily',                 # How often to run
    # Options: '@hourly', '@daily', '@weekly', '@monthly', 
    #          '0 0 * * *' (cron), timedelta(hours=1), None (manual only)
    
    catchup=False,                              # Backfill past runs?
    max_active_runs=1,                          # Max concurrent DAG runs
    
    # TASK DEFAULTS
    default_args={
        'owner': 'data_team',
        'depends_on_past': False,               # Wait for previous run?
        'email': ['team@company.com'],
        'email_on_failure': True,
        'email_on_retry': False,
        'retries': 3,                           # Retry failed tasks
        'retry_delay': timedelta(minutes=5),
        'retry_exponential_backoff': True,      # Increase delay each retry
        'max_retry_delay': timedelta(hours=1),
        'execution_timeout': timedelta(hours=2), # Task timeout
    },
    
    # DOCUMENTATION
    description='Example DAG for learning',
    doc_md="""
    # My DAG Documentation
    This DAG does XYZ...
    """,
    tags=['example', 'tutorial'],
    
    # ADVANCED
    dagrun_timeout=timedelta(hours=4),         # Total DAG timeout
    sla_miss_callback=None,                    # Function to call on SLA miss
    on_failure_callback=None,                  # Function on DAG failure
    on_success_callback=None,                  # Function on DAG success
    
    # CONCURRENCY
    concurrency=16,                            # Max tasks running concurrently
    max_active_tasks=16,                       # Same as above (newer name)
)
```

---

### 2.3 Schedule Intervals

**Common Schedules:**

```python
# Preset schedules
'@once'       # Run once, then never again
'@hourly'     # 0 * * * * (every hour at minute 0)
'@daily'      # 0 0 * * * (midnight every day)
'@weekly'     # 0 0 * * 0 (midnight on Sunday)
'@monthly'    # 0 0 1 * * (midnight on 1st of month)
'@yearly'     # 0 0 1 1 * (midnight on Jan 1)

# Cron expressions
'0 * * * *'       # Every hour
'*/15 * * * *'    # Every 15 minutes
'0 9 * * 1-5'     # 9 AM on weekdays
'0 0 * * 0'       # Midnight every Sunday
'0 2 1 * *'       # 2 AM on 1st of month

# Timedelta (relative scheduling)
from datetime import timedelta
schedule_interval=timedelta(hours=6)    # Every 6 hours
schedule_interval=timedelta(minutes=30) # Every 30 minutes

# No schedule (manual only)
schedule_interval=None

# Data-driven scheduling (new in Airflow 2.4+)
from airflow.timetables.interval import CronDataIntervalTimetable
schedule=CronDataIntervalTimetable("0 0 * * *", timezone="UTC")
```

**Understanding Execution Date:**

```python
# If schedule_interval='@daily' and start_date=2024-01-01:
# - First run: execution_date=2024-01-01, actually runs on 2024-01-02
# - Second run: execution_date=2024-01-02, actually runs on 2024-01-03

# Why? Airflow runs at END of period
# This ensures all data for the period is available

# Access dates in tasks:
from airflow.operators.python import PythonOperator

def my_task(**context):
    execution_date = context['execution_date']
    next_execution_date = context['next_execution_date']
    print(f"Processing data for {execution_date}")
```

---

## 🔧 LEVEL 3: OPERATORS & TASKS

### 3.1 Core Operators

**1. BashOperator - Run Shell Commands:**

```python
from airflow.operators.bash import BashOperator

# Simple command
run_script = BashOperator(
    task_id='run_data_script',
    bash_command='python /scripts/process_data.py',
)

# Multi-line script
complex_bash = BashOperator(
    task_id='complex_processing',
    bash_command="""
        cd /data
        echo "Starting processing..."
        python process.py --date {{ ds }}
        echo "Processing complete!"
    """,
)

# With environment variables
bash_with_env = BashOperator(
    task_id='bash_with_env',
    bash_command='echo "Processing $DATA_PATH"',
    env={'DATA_PATH': '/data/raw'},
)

# Conditional execution
conditional_bash = BashOperator(
    task_id='conditional',
    bash_command='test -f /data/file.csv && echo "File exists" || exit 1',
)
```

**2. PythonOperator - Run Python Functions:**

```python
from airflow.operators.python import PythonOperator

def process_data(param1, param2, **context):
    """Your Python function"""
    execution_date = context['execution_date']
    print(f"Processing with {param1}, {param2} for {execution_date}")
    return {'status': 'success', 'records': 1000}

python_task = PythonOperator(
    task_id='process_data',
    python_callable=process_data,
    op_kwargs={'param1': 'value1', 'param2': 'value2'},
    provide_context=True,  # Pass Airflow context
)

# Using TaskFlow API (cleaner)
from airflow.decorators import task

@task
def process_data_v2(param1: str, param2: str):
    print(f"Processing with {param1}, {param2}")
    return {'status': 'success', 'records': 1000}

result = process_data_v2('value1', 'value2')
```

**3. BranchPythonOperator - Conditional Execution:**

```python
from airflow.operators.python import BranchPythonOperator

def choose_branch(**context):
    """Decide which branch to take"""
    execution_date = context['execution_date']
    
    # Example: different processing on weekends
    if execution_date.weekday() in (5, 6):  # Sat, Sun
        return 'weekend_processing'
    else:
        return 'weekday_processing'

branch = BranchPythonOperator(
    task_id='branching',
    python_callable=choose_branch,
)

weekend_task = BashOperator(
    task_id='weekend_processing',
    bash_command='echo "Weekend process"',
)

weekday_task = BashOperator(
    task_id='weekday_processing',
    bash_command='echo "Weekday process"',
)

branch >> [weekend_task, weekday_task]
```

**4. EmailOperator - Send Emails:**

```python
from airflow.operators.email import EmailOperator

send_email = EmailOperator(
    task_id='send_report',
    to=['team@company.com'],
    subject='Daily Report - {{ ds }}',
    html_content="""
        <h3>Daily Report</h3>
        <p>Date: {{ ds }}</p>
        <p>Records processed: {{ ti.xcom_pull(task_ids='process_data')['records'] }}</p>
    """,
)
```

---

### 3.2 Database Operators

**PostgresOperator:**

```python
from airflow.providers.postgres.operators.postgres import PostgresOperator

create_table = PostgresOperator(
    task_id='create_table',
    postgres_conn_id='postgres_default',  # Connection ID from Airflow UI
    sql="""
        CREATE TABLE IF NOT EXISTS sales (
            id SERIAL PRIMARY KEY,
            date DATE,
            amount DECIMAL(10,2)
        );
    """,
)

insert_data = PostgresOperator(
    task_id='insert_data',
    postgres_conn_id='postgres_default',
    sql="""
        INSERT INTO sales (date, amount)
        VALUES ('{{ ds }}', 1000.00);
    """,
)

# From SQL file
run_sql_file = PostgresOperator(
    task_id='run_sql_file',
    postgres_conn_id='postgres_default',
    sql='queries/transform_data.sql',
)
```

**SnowflakeOperator:**

```python
from airflow.providers.snowflake.operators.snowflake import SnowflakeOperator

snowflake_task = SnowflakeOperator(
    task_id='run_snowflake_query',
    snowflake_conn_id='snowflake_default',
    sql="""
        INSERT INTO analytics.daily_summary
        SELECT 
            '{{ ds }}' AS date,
            COUNT(*) AS orders,
            SUM(amount) AS revenue
        FROM raw.orders
        WHERE DATE(order_timestamp) = '{{ ds }}';
    """,
    warehouse='COMPUTE_WH',
    database='ANALYTICS',
    schema='PUBLIC',
    role='TRANSFORMER',
)
```

**MySQLOperator, MsSqlOperator, etc.:**

```python
from airflow.providers.mysql.operators.mysql import MySqlOperator
from airflow.providers.microsoft.mssql.operators.mssql import MsSqlOperator

mysql_task = MySqlOperator(
    task_id='mysql_query',
    mysql_conn_id='mysql_default',
    sql='SELECT * FROM users WHERE created_date = "{{ ds }}"',
)

mssql_task = MsSqlOperator(
    task_id='mssql_query',
    mssql_conn_id='mssql_default',
    sql='EXEC sp_process_daily_data @date = "{{ ds }}"',
)
```

---

### 3.3 Cloud Operators

**AWS S3 Operators:**

```python
from airflow.providers.amazon.aws.operators.s3 import S3CreateBucketOperator
from airflow.providers.amazon.aws.transfers.local_to_s3 import LocalFilesystemToS3Operator

create_bucket = S3CreateBucketOperator(
    task_id='create_s3_bucket',
    bucket_name='my-data-bucket-{{ ds_nodash }}',
    aws_conn_id='aws_default',
)

upload_to_s3 = LocalFilesystemToS3Operator(
    task_id='upload_to_s3',
    filename='/tmp/data.csv',
    dest_key='raw/data_{{ ds }}.csv',
    dest_bucket='my-data-bucket',
    aws_conn_id='aws_default',
    replace=True,
)
```

**Azure Blob Operators:**

```python
from airflow.providers.microsoft.azure.operators.wasb_delete_blob import WasbDeleteBlobOperator
from airflow.providers.microsoft.azure.transfers.local_to_wasb import LocalFilesystemToWasbOperator

upload_to_blob = LocalFilesystemToWasbOperator(
    task_id='upload_to_blob',
    file_path='/tmp/data.csv',
    container_name='raw-data',
    blob_name='data_{{ ds }}.csv',
    wasb_conn_id='azure_default',
)
```

**Google Cloud Operators:**

```python
from airflow.providers.google.cloud.operators.bigquery import BigQueryInsertJobOperator
from airflow.providers.google.cloud.transfers.gcs_to_bigquery import GCSToBigQueryOperator

bigquery_task = BigQueryInsertJobOperator(
    task_id='run_bigquery',
    configuration={
        'query': {
            'query': 'SELECT COUNT(*) FROM `project.dataset.table` WHERE date = "{{ ds }}"',
            'useLegacySql': False,
        }
    },
    gcp_conn_id='google_cloud_default',
)
```

---

### 3.4 Transfer Operators

**Move data between systems:**

```python
from airflow.providers.amazon.aws.transfers.s3_to_redshift import S3ToRedshiftOperator

s3_to_redshift = S3ToRedshiftOperator(
    task_id='load_s3_to_redshift',
    s3_bucket='my-bucket',
    s3_key='data/{{ ds }}/data.csv',
    schema='public',
    table='staging_table',
    copy_options=['CSV', 'IGNOREHEADER 1'],
    redshift_conn_id='redshift_default',
    aws_conn_id='aws_default',
)

from airflow.providers.mysql.transfers.mysql_to_s3 import MySQLToS3Operator

mysql_to_s3 = MySQLToS3Operator(
    task_id='export_mysql_to_s3',
    query='SELECT * FROM users WHERE created_date = "{{ ds }}"',
    s3_bucket='my-bucket',
    s3_key='exports/users_{{ ds }}.csv',
    mysql_conn_id='mysql_default',
    aws_conn_id='aws_default',
    pd_csv_kwargs={'index': False, 'header': True},
)
```

---

## 🔗 LEVEL 4: TASK DEPENDENCIES & EXECUTION

### 4.1 Setting Dependencies

**Basic Dependencies:**

```python
# Method 1: Bitshift operators (recommended)
task1 >> task2  # task1 before task2
task1 >> task2 >> task3  # Sequential

# Method 2: set_upstream/set_downstream
task2.set_upstream(task1)
task2.set_downstream(task3)

# Multiple dependencies
task1 >> [task2, task3]  # task1 runs, then task2 AND task3 in parallel
[task2, task3] >> task4  # task2 AND task3 complete, then task4

# Complex dependencies
task1 >> task2 >> [task3, task4]
[task3, task4] >> task5

# Visual representation:
#     task1
#       |
#     task2
#      / \
#  task3 task4
#      \ /
#     task5
```

**Dependency Patterns:**

```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime

with DAG('dependency_patterns', start_date=datetime(2024, 1, 1)) as dag:
    
    # Pattern 1: Linear pipeline
    extract = BashOperator(task_id='extract', bash_command='echo extract')
    transform = BashOperator(task_id='transform', bash_command='echo transform')
    load = BashOperator(task_id='load', bash_command='echo load')
    
    extract >> transform >> load
    
    # Pattern 2: Fan-out (parallel processing)
    prepare = BashOperator(task_id='prepare', bash_command='echo prepare')
    process_a = BashOperator(task_id='process_a', bash_command='echo a')
    process_b = BashOperator(task_id='process_b', bash_command='echo b')
    process_c = BashOperator(task_id='process_c', bash_command='echo c')
    combine = BashOperator(task_id='combine', bash_command='echo combine')
    
    prepare >> [process_a, process_b, process_c] >> combine
    
    # Pattern 3: Branching (conditional)
    from airflow.operators.python import BranchPythonOperator
    
    def choose_branch():
        import random
        return 'path_a' if random.random() > 0.5 else 'path_b'
    
    branch = BranchPythonOperator(task_id='branch', python_callable=choose_branch)
    path_a = BashOperator(task_id='path_a', bash_command='echo path A')
    path_b = BashOperator(task_id='path_b', bash_command='echo path B')
    
    branch >> [path_a, path_b]
```

---

### 4.2 Trigger Rules

**Control when tasks execute:**

```python
from airflow.utils.trigger_rule import TriggerRule

# Default: all_success
task = BashOperator(
    task_id='default_task',
    bash_command='echo hello',
    trigger_rule=TriggerRule.ALL_SUCCESS,  # Run only if all parents succeed
)

# Common trigger rules:
task_all_success = BashOperator(
    task_id='all_success',
    bash_command='echo "All parents succeeded"',
    trigger_rule=TriggerRule.ALL_SUCCESS,  # Default
)

task_all_failed = BashOperator(
    task_id='all_failed',
    bash_command='echo "All parents failed"',
    trigger_rule=TriggerRule.ALL_FAILED,  # Run if all parents fail
)

task_one_success = BashOperator(
    task_id='one_success',
    bash_command='echo "At least one parent succeeded"',
    trigger_rule=TriggerRule.ONE_SUCCESS,
)

task_one_failed = BashOperator(
    task_id='one_failed',
    bash_command='echo "At least one parent failed"',
    trigger_rule=TriggerRule.ONE_FAILED,
)

task_none_failed = BashOperator(
    task_id='none_failed',
    bash_command='echo "No parents failed (some may have been skipped)"',
    trigger_rule=TriggerRule.NONE_FAILED,
)

task_always = BashOperator(
    task_id='always_run',
    bash_command='echo "Always runs regardless of parent status"',
    trigger_rule=TriggerRule.ALL_DONE,  # Runs when all parents are done
)

# Example: Cleanup task that always runs
[task1, task2, task3] >> cleanup_task
cleanup_task.trigger_rule = TriggerRule.ALL_DONE  # Run even if tasks fail
```

**Practical Example - Error Handling:**

```python
from airflow.decorators import dag, task
from datetime import datetime

@dag(start_date=datetime(2024, 1, 1), schedule='@daily', catchup=False)
def error_handling_dag():
    
    @task
    def risky_task():
        import random
        if random.random() > 0.5:
            raise Exception("Task failed!")
        return "Success"
    
    @task(trigger_rule=TriggerRule.ALL_FAILED)
    def handle_failure():
        print("Handling failure...")
        # Send alert, log error, etc.
    
    @task(trigger_rule=TriggerRule.ALL_SUCCESS)
    def handle_success():
        print("Everything worked!")
    
    @task(trigger_rule=TriggerRule.ALL_DONE)
    def cleanup():
        print("Cleanup runs regardless of success/failure")
    
    result = risky_task()
    result >> [handle_failure(), handle_success()] >> cleanup()

error_handling_dag()
```

---

### 4.3 Task Groups

**Organize related tasks:**

```python
from airflow.utils.task_group import TaskGroup
from airflow.decorators import dag, task
from datetime import datetime

@dag(start_date=datetime(2024, 1, 1), schedule='@daily', catchup=False)
def task_group_example():
    
    @task
    def start():
        print("Starting pipeline")
    
    # Task Group 1: Data Ingestion
    with TaskGroup('ingestion', tooltip='Ingest data from sources') as ingestion:
        @task
        def extract_db():
            print("Extract from database")
        
        @task
        def extract_api():
            print("Extract from API")
        
        @task
        def extract_files():
            print("Extract from files")
        
        [extract_db(), extract_api(), extract_files()]
    
    # Task Group 2: Transformation
    with TaskGroup('transformation', tooltip='Transform data') as transformation:
        @task
        def clean_data():
            print("Clean data")
        
        @task
        def aggregate_data():
            print("Aggregate data")
        
        clean_data() >> aggregate_data()
    
    # Task Group 3: Loading
    with TaskGroup('loading', tooltip='Load to destinations') as loading:
        @task
        def load_warehouse():
            print("Load to warehouse")
        
        @task
        def load_data_lake():
            print("Load to data lake")
        
        [load_warehouse(), load_data_lake()]
    
    @task
    def finish():
        print("Pipeline complete")
    
    # Set dependencies between groups
    start() >> ingestion >> transformation >> loading >> finish()

task_group_example()

# UI shows groups as collapsible boxes - cleaner visualization!
```

---

## 📦 LEVEL 5: XCOMS & DATA PASSING

### 5.1 Understanding XComs

**XCom = Cross-Communication**
- Share data between tasks
- Stored in metadata database
- Limited size (check your DB limits - typically ~48KB for SQLite, larger for PostgreSQL)
- Use for small data only (task IDs, counts, status, not large datasets)

**Basic XCom Usage:**

```python
from airflow.decorators import dag, task
from datetime import datetime

@dag(start_date=datetime(2024, 1, 1), schedule='@daily', catchup=False)
def xcom_example():
    
    # Method 1: TaskFlow API (automatic XCom)
    @task
    def get_data():
        data = {'count': 100, 'status': 'success'}
        return data  # Automatically pushed to XCom
    
    @task
    def process_data(data: dict):  # Automatically pulls from XCom
        print(f"Received data: {data}")
        count = data['count']
        return count * 2
    
    @task
    def display_result(result: int):
        print(f"Final result: {result}")
    
    data = get_data()
    result = process_data(data)
    display_result(result)

xcom_example()
```

**Manual XCom Push/Pull:**

```python
from airflow.operators.python import PythonOperator

def push_function(**context):
    # Manual push
    context['ti'].xcom_push(key='my_key', value='my_value')
    # or return (pushes to 'return_value' key)
    return {'data': [1, 2, 3]}

def pull_function(**context):
    # Pull by task_id (gets 'return_value' key)
    data = context['ti'].xcom_pull(task_ids='push_task')
    print(f"Pulled data: {data}")
    
    # Pull specific key
    value = context['ti'].xcom_pull(task_ids='push_task', key='my_key')
    print(f"Pulled value: {value}")
    
    # Pull from multiple tasks
    results = context['ti'].xcom_pull(task_ids=['task1', 'task2'])
    print(f"Multiple results: {results}")

push_task = PythonOperator(
    task_id='push_task',
    python_callable=push_function,
)

pull_task = PythonOperator(
    task_id='pull_task',
    python_callable=pull_function,
)

push_task >> pull_task
```

---

### 5.2 XCom Best Practices

**❌ DON'T: Pass Large Data**

```python
# BAD - Don't do this!
@task
def bad_example():
    # Don't pass DataFrames, large lists, files
    import pandas as pd
    df = pd.read_csv('huge_file.csv')  # Millions of rows
    return df  # ❌ Will bloat database, cause errors

# GOOD - Use external storage
@task
def good_example():
    import pandas as pd
    df = pd.read_csv('huge_file.csv')
    
    # Save to S3/GCS/Azure
    df.to_parquet('s3://bucket/data/output.parquet')
    
    # Return metadata only
    return {
        'file_path': 's3://bucket/data/output.parquet',
        'row_count': len(df),
        'columns': list(df.columns)
    }
```

**✅ DO: Pass Metadata**

```python
@task
def extract_data():
    # Process data, save to storage
    # Return metadata
    return {
        'file_location': 's3://bucket/raw/data.parquet',
        'record_count': 10000,
        'extraction_timestamp': datetime.now().isoformat(),
        'status': 'success'
    }

@task
def transform_data(metadata: dict):
    import pandas as pd
    
    # Read from location
    df = pd.read_parquet(metadata['file_location'])
    
    # Transform
    df_transformed = df.groupby('category').sum()
    
    # Save
    output_path = 's3://bucket/transformed/data.parquet'
    df_transformed.to_parquet(output_path)
    
    return {
        'file_location': output_path,
        'record_count': len(df_transformed),
        'status': 'success'
    }
```

---

### 5.3 Custom XCom Backends

**Store XComs in S3 instead of database:**

```python
# airflow/config/custom_xcom_backend.py
from airflow.models.xcom import BaseXCom
import boto3
import pickle

class S3XComBackend(BaseXCom):
    PREFIX = "xcom_s3://"
    BUCKET_NAME = "my-airflow-xcom-bucket"
    
    @staticmethod
    def serialize_value(value):
        if isinstance(value, (dict, list)) and len(str(value)) > 1000:
            # Large data - store in S3
            s3 = boto3.client('s3')
            key = f"xcom/{datetime.now().isoformat()}"
            s3.put_object(
                Bucket=S3XComBackend.BUCKET_NAME,
                Key=key,
                Body=pickle.dumps(value)
            )
            return f"{S3XComBackend.PREFIX}{key}"
        # Small data - store in DB
        return BaseXCom.serialize_value(value)
    
    @staticmethod
    def deserialize_value(result):
        if isinstance(result, str) and result.startswith(S3XComBackend.PREFIX):
            # Retrieve from S3
            key = result.replace(S3XComBackend.PREFIX, "")
            s3 = boto3.client('s3')
            response = s3.get_object(Bucket=S3XComBackend.BUCKET_NAME, Key=key)
            return pickle.loads(response['Body'].read())
        return BaseXCom.deserialize_value(result)

# In airflow.cfg:
# [core]
# xcom_backend = custom_xcom_backend.S3XComBackend
```

---

## 🎯 LEVEL 6: SENSORS & TRIGGERS

### 6.1 Sensors

**Wait for conditions before proceeding:**

**FileSensor - Wait for File:**

```python
from airflow.sensors.filesystem import FileSensor

wait_for_file = FileSensor(
    task_id='wait_for_data_file',
    filepath='/data/input/{{ ds }}.csv',
    poke_interval=30,  # Check every 30 seconds
    timeout=600,       # Timeout after 10 minutes
    mode='poke',       # 'poke' or 'reschedule'
)
```

**S3KeySensor - Wait for S3 File:**

```python
from airflow.providers.amazon.aws.sensors.s3 import S3KeySensor

wait_for_s3 = S3KeySensor(
    task_id='wait_for_s3_file',
    bucket_name='my-bucket',
    bucket_key='raw/data_{{ ds }}.csv',
    aws_conn_id='aws_default',
    timeout=1800,      # 30 minutes
    poke_interval=60,  # Check every minute
)
```

**DateTimeSensor - Wait for Specific Time:**

```python
from airflow.sensors.date_time import DateTimeSensorAsync

wait_until_midnight = DateTimeSensorAsync(
    task_id='wait_until_midnight',
    target_time='{{ execution_date.replace(hour=0, minute=0) }}',
)
```

**ExternalTaskSensor - Wait for Another DAG:**

```python
from airflow.sensors.external_task import ExternalTaskSensor

wait_for_upstream_dag = ExternalTaskSensor(
    task_id='wait_for_upstream',
    external_dag_id='upstream_dag',
    external_task_id='final_task',
    timeout=3600,
)
```

**SqlSensor - Wait for SQL Condition:**

```python
from airflow.providers.common.sql.sensors.sql import SqlSensor

wait_for_data = SqlSensor(
    task_id='wait_for_new_records',
    conn_id='postgres_default',
    sql="SELECT COUNT(*) FROM orders WHERE date = '{{ ds }}'",
    success=lambda x: x[0][0] > 0,  # Wait until count > 0
    poke_interval=300,  # Check every 5 minutes
)
```

---

### 6.2 Sensor Modes

**Poke vs Reschedule:**

```python
# Mode 1: POKE (default)
# - Occupies a worker slot continuously
# - Checks at intervals (poke_interval)
# - Use for short waits (< 5 minutes)

sensor_poke = FileSensor(
    task_id='poke_sensor',
    filepath='/data/file.csv',
    mode='poke',
    poke_interval=30,
)

# Mode 2: RESCHEDULE
# - Frees worker slot between checks
# - More resource-efficient
# - Use for long waits (hours)

sensor_reschedule = FileSensor(
    task_id='reschedule_sensor',
    filepath='/data/file.csv',
    mode='reschedule',
    poke_interval=300,  # 5 minutes
)
```

---

### 6.3 Deferrable Operators (Async)

**More efficient - uses triggers:**

```python
from airflow.providers.amazon.aws.sensors.s3 import S3KeySensorAsync

# Async sensor - doesn't occupy worker slot!
async_sensor = S3KeySensorAsync(
    task_id='async_s3_sensor',
    bucket_name='my-bucket',
    bucket_key='data/{{ ds }}.csv',
    timeout=3600,
)

# Regular sensor for comparison
regular_sensor = S3KeySensor(
    task_id='regular_s3_sensor',
    bucket_name='my-bucket',
    bucket_key='data/{{ ds }}.csv',
    mode='reschedule',
    timeout=3600,
)

# Async is better:
# - Doesn't occupy worker
# - Handled by triggerer process
# - More scalable
```

---

## 📅 LEVEL 7: SCHEDULING & BACKFILLING

### 7.1 Understanding Catchup

**Catchup = Run all missed DAG runs**

```python
from datetime import datetime, timedelta
from airflow import DAG

# Scenario 1: catchup=True (backfill)
# start_date=2024-01-01, today=2024-01-10
# Will create 9 DAG runs (Jan 1-9)
dag_with_catchup = DAG(
    'catchup_true',
    start_date=datetime(2024, 1, 1),
    schedule_interval='@daily',
    catchup=True,  # Run all past dates
)

# Scenario 2: catchup=False (no backfill)
# Will only run from today forward
dag_no_catchup = DAG(
    'catchup_false',
    start_date=datetime(2024, 1, 1),
    schedule_interval='@daily',
    catchup=False,  # Only run current + future
)
```

**When to use catchup:**
- ✅ Historical data processing
- ✅ Rebuilding metrics from scratch
- ✅ Data consistency checks
- ❌ Real-time dashboards
- ❌ External API calls (rate limits)
- ❌ Non-idempotent operations

---

### 7.2 Manual Backfilling

**Backfill specific date range:**

```bash
# Backfill from command line
airflow dags backfill \
    --start-date 2024-01-01 \
    --end-date 2024-01-31 \
    my_dag_id

# Backfill with options
airflow dags backfill \
    --start-date 2024-01-01 \
    --end-date 2024-01-31 \
    --rerun-failed-tasks \
    --reset-dagruns \
    my_dag_id
```

**Programmatic backfill:**

```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime

with DAG(
    'backfill_example',
    start_date=datetime(2024, 1, 1),
    end_date=datetime(2024, 1, 31),  # Optional: auto-stop after this date
    schedule_interval='@daily',
    catchup=True,
) as dag:
    
    process_data = BashOperator(
        task_id='process',
        bash_command='echo "Processing {{ ds }}"',
    )
```

---

### 7.3 Data Interval

**Understanding execution_date vs data_interval:**

```python
# Airflow 2.2+: data_interval concept
# If schedule='@daily' and execution_date='2024-01-15':
# - data_interval_start = 2024-01-15 00:00:00
# - data_interval_end = 2024-01-16 00:00:00
# - logical_date = 2024-01-15 (same as old execution_date)

@task
def process_data(**context):
    # New way (Airflow 2.2+)
    data_interval_start = context['data_interval_start']
    data_interval_end = context['data_interval_end']
    
    # Old way (still works)
    execution_date = context['execution_date']
    
    print(f"Processing data from {data_interval_start} to {data_interval_end}")
```

---

## 🎨 LEVEL 8: PRODUCTION BEST PRACTICES

### 8.1 Error Handling

**Retries and Alerts:**

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

def send_failure_alert(context):
    """Custom failure callback"""
    task_instance = context['task_instance']
    dag_run = context['dag_run']
    
    # Send to Slack, email, PagerDuty, etc.
    print(f"ALERT: Task {task_instance.task_id} failed!")
    print(f"DAG: {dag_run.dag_id}, Run: {dag_run.run_id}")
    
    # Could integrate with monitoring tools:
    # - Send Slack message
    # - Create PagerDuty incident
    # - Log to monitoring system

default_args = {
    'owner': 'data_team',
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
    'retry_exponential_backoff': True,
    'max_retry_delay': timedelta(hours=1),
    'on_failure_callback': send_failure_alert,
}

with DAG(
    'production_dag',
    default_args=default_args,
    start_date=datetime(2024, 1, 1),
    schedule_interval='@daily',
    catchup=False,
) as dag:
    
    @task(retries=5)  # Override DAG-level retries
    def critical_task():
        # Critical operation
        pass
    
    @task(retries=0)  # No retries for this task
    def send_notification():
        # Send final notification
        pass
```

**Try-Except in Tasks:**

```python
@task
def robust_task():
    try:
        # Risky operation
        result = perform_operation()
        return {'status': 'success', 'result': result}
    except SpecificError as e:
        # Handle specific error
        log.error(f"Specific error occurred: {e}")
        # Maybe retry manually or handle gracefully
        return {'status': 'handled_error', 'error': str(e)}
    except Exception as e:
        # Unexpected error - let Airflow retry
        log.error(f"Unexpected error: {e}")
        raise  # Re-raise to trigger retry
    finally:
        # Cleanup code
        cleanup_resources()
```

---

### 8.2 Idempotency

**Make tasks rerunnable:**

```python
# ❌ BAD - Not idempotent
@task
def bad_insert():
    # Will create duplicates if rerun!
    conn.execute("INSERT INTO table VALUES (1, 'data')")

# ✅ GOOD - Idempotent
@task
def good_insert():
    # Delete then insert (or use MERGE/UPSERT)
    conn.execute("DELETE FROM table WHERE id = 1")
    conn.execute("INSERT INTO table VALUES (1, 'data')")

# ✅ BETTER - Use MERGE/UPSERT
@task
def best_insert():
    conn.execute("""
        MERGE INTO table t
        USING (SELECT 1 as id, 'data' as value) s
        ON t.id = s.id
        WHEN MATCHED THEN UPDATE SET value = s.value
        WHEN NOT MATCHED THEN INSERT VALUES (s.id, s.value)
    """)

# ✅ GOOD - Partition-based (Snowflake example)
@task
def partition_insert(**context):
    date = context['ds']
    
    # Delete partition
    conn.execute(f"DELETE FROM table WHERE date = '{date}'")
    
    # Insert new data
    conn.execute(f"""
        INSERT INTO table
        SELECT * FROM staging WHERE date = '{date}'
    """)
```

---

### 8.3 Configuration Management

**Use Variables and Connections:**

```python
from airflow.models import Variable
from airflow.hooks.base import BaseHook

# Set variables in UI or CLI
# airflow variables set my_var "my_value"
# airflow variables set api_key "secret123" --json

@task
def use_variables():
    # Get variable
    api_key = Variable.get("api_key")
    
    # With default
    threshold = Variable.get("threshold", default_var=100)
    
    # JSON variable
    config = Variable.get("config", deserialize_json=True)
    
    # Connection
    conn = BaseHook.get_connection('my_postgres')
    host = conn.host
    user = conn.login
    password = conn.password
```

**Environment-Specific Config:**

```python
from airflow.models import Variable

@task
def environment_aware_task():
    env = Variable.get("environment", default_var="dev")
    
    if env == "prod":
        bucket = "prod-data-bucket"
        warehouse = "PROD_WH"
    elif env == "staging":
        bucket = "staging-data-bucket"
        warehouse = "STAGING_WH"
    else:
        bucket = "dev-data-bucket"
        warehouse = "DEV_WH"
    
    print(f"Using bucket: {bucket}, warehouse: {warehouse}")
```

---

### 8.4 Testing DAGs

**Unit Testing:**

```python
# tests/dags/test_my_dag.py
import pytest
from airflow.models import DagBag
from datetime import datetime

def test_dag_loaded():
    """Test DAG loads without errors"""
    dag_bag = DagBag(dag_folder='dags/', include_examples=False)
    assert dag_bag.import_errors == {}
    assert 'my_dag' in dag_bag.dags

def test_dag_structure():
    """Test DAG has expected structure"""
    dag_bag = DagBag(dag_folder='dags/')
    dag = dag_bag.get_dag('my_dag')
    
    assert len(dag.tasks) == 5
    assert 'extract' in dag.task_dict
    assert 'transform' in dag.task_dict
    assert 'load' in dag.task_dict

def test_task_dependencies():
    """Test tasks have correct dependencies"""
    dag_bag = DagBag(dag_folder='dags/')
    dag = dag_bag.get_dag('my_dag')
    
    extract = dag.task_dict['extract']
    transform = dag.task_dict['transform']
    
    assert transform in extract.downstream_list

def test_task_execution():
    """Test individual task execution"""
    from dags.my_dag import process_data
    
    result = process_data(param1='test')
    assert result['status'] == 'success'
```

**Integration Testing:**

```python
# Test with actual DAG run
from airflow.models import DagBag
from airflow.utils.state import State

def test_dag_run():
    """Test full DAG execution"""
    dag_bag = DagBag(dag_folder='dags/')
    dag = dag_bag.get_dag('my_dag')
    
    # Create DAG run
    dag.clear()
    dag.run(
        start_date=datetime(2024, 1, 1),
        end_date=datetime(2024, 1, 1),
    )
    
    # Check all tasks succeeded
    for task in dag.tasks:
        task_instance = task.get_task_instance(datetime(2024, 1, 1))
        assert task_instance.state == State.SUCCESS
```

---

### 8.5 Logging Best Practices

**Structured Logging:**

```python
import logging
from airflow.decorators import task

logger = logging.getLogger(__name__)

@task
def well_logged_task(**context):
    execution_date = context['execution_date']
    
    logger.info(f"Starting task for {execution_date}")
    logger.info(f"Configuration: {config}")
    
    try:
        result = process_data()
        logger.info(f"Processed {result['row_count']} rows")
        logger.info(f"Output: {result['output_path']}")
        return result
    except Exception as e:
        logger.error(f"Error processing data: {e}", exc_info=True)
        raise

# Logs available in:
# - Airflow UI (per task)
# - Log files: $AIRFLOW_HOME/logs/dag_id/task_id/execution_date/
```

---

### 8.6 Resource Management

**Pool Configuration:**

```python
# Create pools in UI or CLI
# airflow pools set default_pool 128 "Default pool"
# airflow pools set etl_pool 5 "Limited ETL resources"

from airflow.operators.python import PythonOperator

# Use pool to limit concurrent tasks
task = PythonOperator(
    task_id='limited_task',
    python_callable=my_function,
    pool='etl_pool',  # Max 5 tasks from this pool run concurrently
    priority_weight=10,  # Higher priority tasks run first
)
```

**Task Concurrency:**

```python
with DAG(
    'resource_managed_dag',
    start_date=datetime(2024, 1, 1),
    max_active_tasks=5,  # Max 5 tasks running at once
    max_active_runs=1,   # Only 1 DAG run at a time
) as dag:
    
    # Generate 100 tasks but only 5 run concurrently
    for i in range(100):
        PythonOperator(
            task_id=f'task_{i}',
            python_callable=lambda: print(f"Processing {i}"),
        )
```

---

## 🚀 LEVEL 9: ADVANCED PATTERNS

### 9.1 Dynamic DAG Generation

**Generate tasks programmatically:**

```python
from airflow.decorators import dag, task
from datetime import datetime

# Generate DAGs for multiple customers
CUSTOMERS = ['customer_a', 'customer_b', 'customer_c']

for customer in CUSTOMERS:
    @dag(
        dag_id=f'process_{customer}_data',
        start_date=datetime(2024, 1, 1),
        schedule='@daily',
        catchup=False,
    )
    def customer_pipeline():
        @task
        def extract():
            print(f"Extracting data for {customer}")
        
        @task
        def transform():
            print(f"Transforming data for {customer}")
        
        @task
        def load():
            print(f"Loading data for {customer}")
        
        extract() >> transform() >> load()
    
    # Create DAG instance
    globals()[f'process_{customer}_data'] = customer_pipeline()
```

**Dynamic Task Generation:**

```python
from airflow.decorators import dag, task
from datetime import datetime

@dag(start_date=datetime(2024, 1, 1), schedule='@daily', catchup=False)
def dynamic_tasks_dag():
    
    @task
    def get_table_list():
        # Fetch list of tables to process
        return ['users', 'orders', 'products', 'inventory', 'reviews']
    
    @task
    def process_table(table_name: str):
        print(f"Processing table: {table_name}")
        # Process the table
        return f"{table_name}_complete"
    
    @task
    def combine_results(results: list):
        print(f"All tables processed: {results}")
    
    # Dynamic pipeline
    tables = get_table_list()
    results = process_table.expand(table_name=tables)  # Create task for each table
    combine_results(results)

dynamic_tasks_dag()
```

---

### 9.2 SubDAGs (Legacy - Prefer TaskGroups)

**Reusable sub-workflows:**

```python
# Better to use TaskGroups in Airflow 2.0+
# But here's how SubDAGs work:

from airflow.models import DAG
from airflow.operators.subdag import SubDagOperator
from airflow.operators.bash import BashOperator
from datetime import datetime

def create_subdag(parent_dag_id, subdag_id, start_date, schedule_interval):
    """Factory function for subdag"""
    subdag = DAG(
        dag_id=f'{parent_dag_id}.{subdag_id}',
        start_date=start_date,
        schedule_interval=schedule_interval,
    )
    
    for i in range(3):
        BashOperator(
            task_id=f'task_{i}',
            bash_command=f'echo "SubDAG task {i}"',
            dag=subdag,
        )
    
    return subdag

# Main DAG
with DAG('main_dag', start_date=datetime(2024, 1, 1), schedule_interval='@daily') as dag:
    
    start = BashOperator(task_id='start', bash_command='echo start')
    
    subdag_task = SubDagOperator(
        task_id='sub_workflow',
        subdag=create_subdag('main_dag', 'sub_workflow', datetime(2024, 1, 1), '@daily'),
    )
    
    end = BashOperator(task_id='end', bash_command='echo end')
    
    start >> subdag_task >> end
```

---

### 9.3 Datasets (Airflow 2.4+)

**Data-aware scheduling:**

```python
from airflow import Dataset
from airflow.decorators import dag, task
from datetime import datetime

# Define datasets
customer_data = Dataset("s3://bucket/customers/")
order_data = Dataset("s3://bucket/orders/")
analytics_data = Dataset("s3://bucket/analytics/")

# Producer DAG - creates datasets
@dag(
    start_date=datetime(2024, 1, 1),
    schedule='@hourly',
    catchup=False,
)
def producer_dag():
    @task(outlets=[customer_data])  # Declares it produces this dataset
    def update_customers():
        # Update customer data
        print("Updating customer data...")
    
    @task(outlets=[order_data])
    def update_orders():
        # Update order data
        print("Updating order data...")
    
    update_customers()
    update_orders()

# Consumer DAG - triggered when datasets updated
@dag(
    start_date=datetime(2024, 1, 1),
    schedule=[customer_data, order_data],  # Runs when BOTH are updated
    catchup=False,
)
def consumer_dag():
    @task(outlets=[analytics_data])
    def create_analytics():
        # Create analytics from customer + order data
        print("Creating analytics...")
    
    create_analytics()

producer_dag()
consumer_dag()

# consumer_dag automatically runs when both datasets are updated!
```

---

### 9.4 Custom Operators

**Create reusable operators:**

```python
# plugins/operators/snowflake_to_s3_operator.py
from airflow.models import BaseOperator
from airflow.providers.snowflake.hooks.snowflake import SnowflakeHook
from airflow.providers.amazon.aws.hooks.s3 import S3Hook
import pandas as pd

class SnowflakeToS3Operator(BaseOperator):
    """
    Extract data from Snowflake and load to S3
    """
    template_fields = ('sql', 's3_key')
    
    def __init__(
        self,
        sql: str,
        s3_bucket: str,
        s3_key: str,
        snowflake_conn_id: str = 'snowflake_default',
        aws_conn_id: str = 'aws_default',
        **kwargs
    ):
        super().__init__(**kwargs)
        self.sql = sql
        self.s3_bucket = s3_bucket
        self.s3_key = s3_key
        self.snowflake_conn_id = snowflake_conn_id
        self.aws_conn_id = aws_conn_id
    
    def execute(self, context):
        # Extract from Snowflake
        sf_hook = SnowflakeHook(snowflake_conn_id=self.snowflake_conn_id)
        df = sf_hook.get_pandas_df(self.sql)
        
        self.log.info(f"Extracted {len(df)} rows from Snowflake")
        
        # Save to S3
        s3_hook = S3Hook(aws_conn_id=self.aws_conn_id)
        
        # Convert to parquet
        parquet_buffer = df.to_parquet(index=False)
        
        s3_hook.load_bytes(
            bytes_data=parquet_buffer,
            bucket_name=self.s3_bucket,
            key=self.s3_key,
            replace=True,
        )
        
        self.log.info(f"Loaded data to s3://{self.s3_bucket}/{self.s3_key}")
        
        return {
            'row_count': len(df),
            's3_location': f's3://{self.s3_bucket}/{self.s3_key}'
        }

# Use in DAG:
from operators.snowflake_to_s3_operator import SnowflakeToS3Operator

export_task = SnowflakeToS3Operator(
    task_id='export_to_s3',
    sql="SELECT * FROM customers WHERE date = '{{ ds }}'",
    s3_bucket='my-bucket',
    s3_key='exports/customers_{{ ds }}.parquet',
)
```

---

## 📊 LEVEL 10: MONITORING & DEBUGGING

### 10.1 Airflow UI Navigation

**Key UI Features:**

1. **DAGs Page** - View all DAGs, statuses
2. **Graph View** - Visual DAG structure
3. **Tree View** - Historical runs
4. **Gantt View** - Task duration timeline
5. **Task Duration** - Performance over time
6. **Calendar View** - Run history calendar
7. **Code View** - View DAG source code
8. **Logs** - Per-task execution logs

**Useful Filters:**

```python
# In UI, use tags to organize DAGs
DAG(
    'my_dag',
    tags=['production', 'finance', 'hourly'],
)

# Filter in UI by:
# - Owner
# - Tags
# - Status (running, success, failed)
```

---

### 10.2 Debugging Failed Tasks

**Steps to debug:**

1. **Check Task Logs**
   - Click on failed task → Logs
   - Look for error messages, stack traces

2. **Check XCom Values**
   - Click task → XCom
   - Verify data passed between tasks

3. **Inspect Context Variables**
```python
@task
def debug_task(**context):
    # Print all available context
    for key, value in context.items():
        print(f"{key}: {value}")
```

4. **Test Locally**
```bash
# Test single task
airflow tasks test my_dag my_task 2024-01-01
```

5. **Use PyCharm/VSCode Debugging**
```python
@task
def breakpoint_task():
    import pdb; pdb.set_trace()  # Python debugger
    # Code to debug
```

---

### 10.3 Metrics & Monitoring

**Airflow Metrics (StatsD/Prometheus):**

```python
# airflow.cfg
[metrics]
statsd_on = True
statsd_host = localhost
statsd_port = 8125
statsd_prefix = airflow

# Metrics emitted:
# - dag.{dag_id}.{task_id}.duration
# - dag.{dag_id}.{task_id}.success/failure
# - scheduler.tasks.running/queued
# - executor.open_slots
```

**Health Checks:**

```bash
# Check scheduler health
airflow jobs check --job-type SchedulerJob --hostname $(hostname)

# Check webserver
curl http://localhost:8080/health

# Database health
airflow db check
```

**Custom Metrics:**

```python
from airflow.stats import Stats

@task
def monitored_task():
    # Increment counter
    Stats.incr('custom.tasks.processed')
    
    # Time operation
    with Stats.timer('custom.processing.duration'):
        result = expensive_operation()
    
    # Gauge metric
    Stats.gauge('custom.records.count', len(result))
    
    return result
```

---

## 🏗️ PRACTICE PROJECTS

### Project 1: E-Commerce ETL Pipeline

**Build a complete data pipeline:**

```python
from airflow.decorators import dag, task
from airflow.providers.postgres.operators.postgres import PostgresOperator
from airflow.providers.amazon.aws.transfers.s3_to_sql import S3ToSqlOperator
from datetime import datetime

@dag(
    start_date=datetime(2024, 1, 1),
    schedule='@daily',
    catchup=False,
    tags=['ecommerce', 'production']
)
def ecommerce_pipeline():
    """
    Daily ETL for e-commerce data
    1. Extract from API
    2. Load raw to S3
    3. Stage in Postgres
    4. Transform
    5. Load to DWH
    6. Create aggregates
    """
    
    @task
    def extract_orders_api(**context):
        """Extract orders from API"""
        import requests
        date = context['ds']
        response = requests.get(f'https://api.example.com/orders?date={date}')
        orders = response.json()
        # Save to S3
        return {'count': len(orders), 's3_path': f's3://bucket/raw/orders_{date}.json'}
    
    create_staging_table = PostgresOperator(
        task_id='create_staging',
        sql="""
            CREATE TABLE IF NOT EXISTS staging_orders (
                order_id INT,
                customer_id INT,
                amount DECIMAL(10,2),
                order_date DATE
            );
            TRUNCATE TABLE staging_orders;
        """
    )
    
    @task
    def transform_orders():
        """Transform staging to production"""
        # Complex transformation logic
        pass
    
    @task
    def create_daily_aggregates():
        """Create daily summary"""
        pass
    
    @task
    def send_completion_email(**context):
        """Send pipeline completion notification"""
        from airflow.operators.email import EmailOperator
        # Send email with stats
        pass
    
    # Pipeline
    raw_data = extract_orders_api()
    create_staging_table >> transform_orders() >> create_daily_aggregates() >> send_completion_email()

ecommerce_pipeline()
```

---

### Project 2: Data Quality Framework

**Implement data quality checks:**

```python
from airflow.decorators import dag, task
from datetime import datetime

@dag(
    start_date=datetime(2024, 1, 1),
    schedule='@daily',
    catchup=False,
)
def data_quality_dag():
    
    @task
    def check_row_count(**context):
        """Ensure minimum row count"""
        # Query database
        count = query_db("SELECT COUNT(*) FROM orders WHERE date = '{{ ds }}'")
        
        if count < 100:
            raise Exception(f"Too few rows: {count}")
        
        return {'row_count': count}
    
    @task
    def check_null_values():
        """Check for unexpected nulls"""
        null_count = query_db("""
            SELECT COUNT(*) FROM orders 
            WHERE customer_id IS NULL 
            AND date = '{{ ds }}'
        """)
        
        if null_count > 0:
            raise Exception(f"Found {null_count} null customer_ids")
    
    @task
    def check_duplicates():
        """Check for duplicate records"""
        dup_count = query_db("""
            SELECT COUNT(*) FROM (
                SELECT order_id, COUNT(*) 
                FROM orders 
                WHERE date = '{{ ds }}'
                GROUP BY order_id 
                HAVING COUNT(*) > 1
            )
        """)
        
        if dup_count > 0:
            raise Exception(f"Found {dup_count} duplicate orders")
    
    @task
    def check_referential_integrity():
        """Ensure foreign keys exist"""
        orphan_count = query_db("""
            SELECT COUNT(*) FROM orders o
            LEFT JOIN customers c ON o.customer_id = c.customer_id
            WHERE c.customer_id IS NULL
            AND o.date = '{{ ds }}'
        """)
        
        if orphan_count > 0:
            raise Exception(f"Found {orphan_count} orders with invalid customer_id")
    
    # Run all checks in parallel
    [check_row_count(), check_null_values(), check_duplicates(), check_referential_integrity()]

data_quality_dag()
```

---

### Project 3: Multi-Source Data Integration

**Combine data from various sources:**

```python
from airflow.decorators import dag, task, task_group
from datetime import datetime

@dag(start_date=datetime(2024, 1, 1), schedule='@daily', catchup=False)
def multi_source_integration():
    
    @task_group
    def extract_sources():
        @task
        def extract_mysql():
            # Extract from MySQL
            return 's3://bucket/mysql_data.parquet'
        
        @task
        def extract_api():
            # Extract from API
            return 's3://bucket/api_data.parquet'
        
        @task
        def extract_files():
            # Extract from FTP/SFTP
            return 's3://bucket/file_data.parquet'
        
        return [extract_mysql(), extract_api(), extract_files()]
    
    @task
    def combine_sources(file_paths: list):
        import pandas as pd
        
        # Read all sources
        dfs = [pd.read_parquet(path) for path in file_paths]
        
        # Combine
        combined = pd.concat(dfs, ignore_index=True)
        
        # Save
        output_path = 's3://bucket/combined_data.parquet'
        combined.to_parquet(output_path)
        
        return output_path
    
    @task
    def load_to_warehouse(file_path: str):
        # Load to Snowflake/Redshift/BigQuery
        pass
    
    sources = extract_sources()
    combined = combine_sources(sources)
    load_to_warehouse(combined)

multi_source_integration()
```

---

## ✅ AIRFLOW MASTERY CHECKLIST

### Beginner ✓
- [ ] Install and run Airflow locally
- [ ] Create basic DAG with BashOperator
- [ ] Use PythonOperator
- [ ] Set task dependencies
- [ ] Understand schedule_interval

### Intermediate ✓
- [ ] Use XComs for data passing
- [ ] Implement branching logic
- [ ] Work with database operators
- [ ] Use sensors
- [ ] Configure connections and variables

### Advanced ✓
- [ ] Build production DAGs with error handling
- [ ] Implement dynamic task generation
- [ ] Create custom operators
- [ ] Use task groups effectively
- [ ] Implement data quality checks

### Expert ✓
- [ ] Design scalable DAG architectures
- [ ] Optimize resource usage
- [ ] Implement monitoring and alerting
- [ ] Use Datasets for data-aware scheduling
- [ ] Deploy Airflow in production (Docker/Kubernetes)

---

## 📚 LEARNING RESOURCES

**Official:**
- Airflow Documentation: https://airflow.apache.org/docs/
- Airflow GitHub: https://github.com/apache/airflow
- Astronomer Registry: https://registry.astronomer.io/

**Courses:**
- Astronomer Academy (free)
- Udemy Airflow courses
- Coursera/DataCamp

**Practice:**
- Astronomer Certified Apache Airflow
- Build real projects
- Contribute to open source

---

## 🎯 30-DAY LEARNING PLAN

**Week 1: Foundations**
- Days 1-2: Install, understand architecture
- Days 3-4: Create basic DAGs
- Days 5-7: Operators and dependencies

**Week 2: Intermediate**
- Days 8-10: XComs, sensors, branching
- Days 11-12: Database operators
- Days 13-14: Cloud operators (AWS/Azure/GCP)

**Week 3: Advanced**
- Days 15-17: Error handling, retries
- Days 18-19: Dynamic DAGs
- Days 20-21: Custom operators

**Week 4: Production**
- Days 22-24: Build complete ETL project
- Days 25-27: Monitoring, testing
- Days 28-30: Deploy to production (Docker)

---

**You've mastered Airflow when:**
✓ You can design complex data pipelines
✓ You understand when to use each operator
✓ You can debug failed tasks efficiently
✓ You know production best practices
✓ You can optimize resource usage
✓ You're comfortable with advanced patterns

**Next up - what should we deep dive into?**
1. Python for Data Engineering (pandas, APIs)
2. dbt (data build tool)
3. Docker & Kubernetes for data pipelines
4. Azure Data Factory (alternative to Airflow)
5. Real-time streaming (Kafka, Spark Streaming)

Ready? 🚀
