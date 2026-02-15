# ☁️ AZURE DATA FACTORY (ADF) MASTERY - Complete Learning Guide

## 📋 Table of Contents
1. [ADF Fundamentals](#fundamentals)
2. [Architecture & Components](#architecture)
3. [Linked Services & Datasets](#linked-services)
4. [Pipelines & Activities](#pipelines)
5. [Data Flows](#data-flows)
6. [Triggers & Scheduling](#triggers)
7. [Parameters & Variables](#parameters)
8. [Control Flow Activities](#control-flow)
9. [Monitoring & Debugging](#monitoring)
10. [Integration Runtime](#integration-runtime)
11. [CI/CD & DevOps](#cicd)
12. [Real-World Projects](#projects)

---

## 🎯 LEVEL 1: ADF FUNDAMENTALS

### 1.1 What is Azure Data Factory?

**Definition:**
Azure Data Factory (ADF) is a **cloud-based ETL and data integration service** that allows you to create data-driven workflows for orchestrating data movement and transforming data at scale.

**Key Differentiators:**

| Feature | ADF | Apache Airflow |
|---------|-----|----------------|
| **Interface** | GUI + Code (JSON) | Code-first (Python) |
| **Serverless** | Fully managed | Self-managed |
| **Pricing** | Pay-per-activity | Infrastructure costs |
| **Learning Curve** | Lower (GUI) | Steeper (coding) |
| **Flexibility** | Medium | High |
| **Azure Integration** | Native | Via providers |

**When to Use ADF:**
✓ **Azure-centric** data stack
✓ **Low-code** preference
✓ **Serverless** requirement
✓ **Built-in connectors** to 90+ data sources
✓ **Visual design** preferred
✓ **Enterprise compliance** (Azure security)

---

### 1.2 ADF Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    AZURE DATA FACTORY                        │
│                    (Control Plane)                           │
└─────────────────────────────────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
┌───────▼────────┐  ┌──────▼───────┐  ┌────────▼──────┐
│   PIPELINES    │  │   DATA FLOWS │  │   TRIGGERS    │
│  (Orchestration│  │ (ETL Logic)  │  │  (Scheduling) │
└────────────────┘  └──────────────┘  └───────────────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
┌─────────────────────────────────────────────────────────────┐
│              INTEGRATION RUNTIME (IR)                        │
│              (Execution Engine)                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Azure IR    │  │ Self-Hosted  │  │Azure-SSIS IR │      │
│  │  (Cloud)     │  │ IR (On-prem) │  │(SSIS packages│      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
┌───────▼────────┐  ┌──────▼───────┐  ┌────────▼──────┐
│LINKED SERVICES │  │   DATASETS   │  │  DATA STORES  │
│ (Connections)  │  │(Data Schemas)│  │ (Sources/Sinks│
└────────────────┘  └──────────────┘  └───────────────┘
```

**Core Components:**

1. **Pipelines** - Logical grouping of activities
2. **Activities** - Processing steps (Copy, Transform, Control Flow)
3. **Datasets** - Data structures (tables, files, etc.)
4. **Linked Services** - Connection strings to data stores
5. **Triggers** - Schedulers and event triggers
6. **Integration Runtime** - Compute infrastructure
7. **Data Flows** - Visual data transformation designer

---

### 1.3 Creating Your First Data Factory

**Via Azure Portal:**

```
1. Azure Portal → Create a resource
2. Search "Data Factory"
3. Create Data Factory:
   - Subscription: Your subscription
   - Resource Group: rg-adf-demo
   - Region: East US
   - Name: adf-demo-dev
   - Version: V2
   - Git Configuration: Configure later
   
4. Click "Review + Create"
5. Launch "Author & Monitor"
```

**Via Azure CLI:**

```bash
# Login to Azure
az login

# Create resource group
az group create --name rg-adf-demo --location eastus

# Create Data Factory
az datafactory create \
  --resource-group rg-adf-demo \
  --factory-name adf-demo-dev \
  --location eastus

# List Data Factories
az datafactory list --resource-group rg-adf-demo
```

**Via PowerShell:**

```powershell
# Login
Connect-AzAccount

# Create Data Factory
$DataFactory = Set-AzDataFactoryV2 `
  -ResourceGroupName "rg-adf-demo" `
  -Location "EastUS" `
  -Name "adf-demo-dev"
```

---

## 🔗 LEVEL 2: LINKED SERVICES & DATASETS

### 2.1 Linked Services (Connection Strings)

**What are Linked Services?**
- Connection information to external resources
- Like "connections" in Airflow or "connection strings" in code
- Stores credentials, endpoints, authentication

**Common Linked Services:**

**1. Azure Blob Storage:**

```json
{
    "name": "AzureBlobStorage_LS",
    "type": "Microsoft.DataFactory/factories/linkedservices",
    "properties": {
        "annotations": [],
        "type": "AzureBlobStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=mystorageaccount;AccountKey=<key>;EndpointSuffix=core.windows.net",
            "encryptedCredential": null
        }
    }
}
```

**2. Azure SQL Database:**

```json
{
    "name": "AzureSqlDatabase_LS",
    "type": "Microsoft.DataFactory/factories/linkedservices",
    "properties": {
        "type": "AzureSqlDatabase",
        "typeProperties": {
            "connectionString": "Server=tcp:myserver.database.windows.net,1433;Database=mydb;User ID=myuser;Password=<password>;Encrypt=True;",
            "encryptedCredential": null
        }
    }
}
```

**3. Azure Data Lake Storage Gen2:**

```json
{
    "name": "AzureDataLakeStorage_LS",
    "type": "Microsoft.DataFactory/factories/linkedservices",
    "properties": {
        "type": "AzureBlobFS",
        "typeProperties": {
            "url": "https://mydatalake.dfs.core.windows.net",
            "accountKey": {
                "type": "SecureString",
                "value": "<account-key>"
            }
        }
    }
}
```

**4. Snowflake:**

```json
{
    "name": "Snowflake_LS",
    "type": "Microsoft.DataFactory/factories/linkedservices",
    "properties": {
        "type": "Snowflake",
        "typeProperties": {
            "connectionString": "jdbc:snowflake://myaccount.snowflakecomputing.com/?user=myuser&db=mydb&warehouse=COMPUTE_WH",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        }
    }
}
```

**5. REST API:**

```json
{
    "name": "RestService_LS",
    "type": "Microsoft.DataFactory/factories/linkedservices",
    "properties": {
        "type": "RestService",
        "typeProperties": {
            "url": "https://api.example.com",
            "enableServerCertificateValidation": true,
            "authenticationType": "Basic",
            "userName": "apiuser",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        }
    }
}
```

**6. On-Premises SQL Server (via Self-Hosted IR):**

```json
{
    "name": "OnPremisesSqlServer_LS",
    "type": "Microsoft.DataFactory/factories/linkedservices",
    "properties": {
        "type": "SqlServer",
        "typeProperties": {
            "connectionString": "Server=myserver;Database=mydb;User ID=myuser;Password=<password>;",
            "encryptedCredential": null
        },
        "connectVia": {
            "referenceName": "SelfHostedIR",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

---

### 2.2 Using Azure Key Vault for Secrets

**Best Practice: Store credentials in Key Vault**

**1. Create Key Vault Linked Service:**

```json
{
    "name": "AzureKeyVault_LS",
    "type": "Microsoft.DataFactory/factories/linkedservices",
    "properties": {
        "type": "AzureKeyVault",
        "typeProperties": {
            "baseUrl": "https://mykeyvault.vault.azure.net/"
        }
    }
}
```

**2. Reference Secrets from Key Vault:**

```json
{
    "name": "AzureSqlDatabase_LS",
    "properties": {
        "type": "AzureSqlDatabase",
        "typeProperties": {
            "connectionString": "Server=tcp:myserver.database.windows.net,1433;Database=mydb;",
            "password": {
                "type": "AzureKeyVaultSecret",
                "store": {
                    "referenceName": "AzureKeyVault_LS",
                    "type": "LinkedServiceReference"
                },
                "secretName": "SqlDatabasePassword"
            }
        }
    }
}
```

---

### 2.3 Datasets

**What are Datasets?**
- Represent data structures within data stores
- Point to or reference the data you want to use
- Define schema, location, format

**Common Dataset Types:**

**1. Delimited Text (CSV) Dataset:**

```json
{
    "name": "CSV_Dataset",
    "properties": {
        "linkedServiceName": {
            "referenceName": "AzureBlobStorage_LS",
            "type": "LinkedServiceReference"
        },
        "type": "DelimitedText",
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "fileName": "data.csv",
                "folderPath": "raw",
                "container": "datacontainer"
            },
            "columnDelimiter": ",",
            "escapeChar": "\\",
            "firstRowAsHeader": true,
            "quoteChar": "\""
        },
        "schema": [
            {
                "name": "customer_id",
                "type": "String"
            },
            {
                "name": "name",
                "type": "String"
            },
            {
                "name": "email",
                "type": "String"
            }
        ]
    }
}
```

**2. Parquet Dataset:**

```json
{
    "name": "Parquet_Dataset",
    "properties": {
        "linkedServiceName": {
            "referenceName": "AzureDataLakeStorage_LS",
            "type": "LinkedServiceReference"
        },
        "type": "Parquet",
        "typeProperties": {
            "location": {
                "type": "AzureBlobFSLocation",
                "fileName": "data.parquet",
                "folderPath": "processed",
                "fileSystem": "datalake"
            },
            "compressionCodec": "snappy"
        }
    }
}
```

**3. Azure SQL Table Dataset:**

```json
{
    "name": "AzureSqlTable_Dataset",
    "properties": {
        "linkedServiceName": {
            "referenceName": "AzureSqlDatabase_LS",
            "type": "LinkedServiceReference"
        },
        "type": "AzureSqlTable",
        "typeProperties": {
            "schema": "dbo",
            "table": "customers"
        }
    }
}
```

**4. Parameterized Dataset:**

```json
{
    "name": "Parameterized_CSV_Dataset",
    "properties": {
        "linkedServiceName": {
            "referenceName": "AzureBlobStorage_LS",
            "type": "LinkedServiceReference"
        },
        "parameters": {
            "fileName": {
                "type": "string"
            },
            "folderPath": {
                "type": "string"
            }
        },
        "type": "DelimitedText",
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "fileName": {
                    "value": "@dataset().fileName",
                    "type": "Expression"
                },
                "folderPath": {
                    "value": "@dataset().folderPath",
                    "type": "Expression"
                },
                "container": "datacontainer"
            },
            "columnDelimiter": ",",
            "firstRowAsHeader": true
        }
    }
}
```

---

## 🔄 LEVEL 3: PIPELINES & ACTIVITIES

### 3.1 Pipeline Basics

**What is a Pipeline?**
- Logical grouping of activities
- Defines a workflow
- Can be scheduled or triggered

**Your First Pipeline - Copy Activity:**

```json
{
    "name": "CopyPipeline",
    "properties": {
        "activities": [
            {
                "name": "CopyFromBlobToSQL",
                "type": "Copy",
                "inputs": [
                    {
                        "referenceName": "CSV_Dataset",
                        "type": "DatasetReference"
                    }
                ],
                "outputs": [
                    {
                        "referenceName": "AzureSqlTable_Dataset",
                        "type": "DatasetReference"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "DelimitedTextSource",
                        "storeSettings": {
                            "type": "AzureBlobStorageReadSettings"
                        }
                    },
                    "sink": {
                        "type": "AzureSqlSink",
                        "writeBehavior": "insert",
                        "sqlWriterUseTableLock": false
                    },
                    "enableStaging": false,
                    "translator": {
                        "type": "TabularTranslator",
                        "mappings": [
                            {
                                "source": {"name": "customer_id"},
                                "sink": {"name": "CustomerID"}
                            },
                            {
                                "source": {"name": "name"},
                                "sink": {"name": "CustomerName"}
                            }
                        ]
                    }
                }
            }
        ]
    }
}
```

---

### 3.2 Copy Activity Deep Dive

**Copy Activity is the workhorse of ADF**

**Source Options:**

```json
{
    "source": {
        "type": "AzureSqlSource",
        "sqlReaderQuery": "SELECT * FROM customers WHERE created_date >= '@{formatDateTime(pipeline().parameters.startDate, 'yyyy-MM-dd')}'",
        "queryTimeout": "02:00:00",
        "partitionOption": "None"
    }
}
```

**Sink Options:**

```json
{
    "sink": {
        "type": "AzureSqlSink",
        "writeBehavior": "upsert",
        "upsertSettings": {
            "useTempDB": true,
            "keys": ["CustomerID"]
        },
        "sqlWriterStoredProcedureName": "spUpsertCustomers",
        "sqlWriterTableType": "CustomerTableType",
        "preCopyScript": "TRUNCATE TABLE staging_customers",
        "tableOption": "autoCreate"
    }
}
```

**Performance Tuning:**

```json
{
    "typeProperties": {
        "source": {...},
        "sink": {...},
        "enableStaging": true,
        "stagingSettings": {
            "linkedServiceName": {
                "referenceName": "AzureBlobStorage_LS",
                "type": "LinkedServiceReference"
            },
            "path": "staging"
        },
        "parallelCopies": 32,
        "dataIntegrationUnits": 16,
        "enableSkipIncompatibleRow": true,
        "logSettings": {
            "enableCopyActivityLog": true,
            "copyActivityLogSettings": {
                "logLevel": "Warning",
                "enableReliableLogging": false
            }
        }
    }
}
```

**Data Integration Units (DIU):**
- Azure IR uses DIUs for cloud-to-cloud copies
- Range: 2-256 DIUs
- Higher DIUs = more parallelism = faster copies
- Auto-tuning available

---

### 3.3 Other Common Activities

**1. Lookup Activity - Get Metadata:**

```json
{
    "name": "LookupActivity",
    "type": "Lookup",
    "typeProperties": {
        "source": {
            "type": "AzureSqlSource",
            "sqlReaderQuery": "SELECT MAX(last_modified) as max_date FROM customers"
        },
        "dataset": {
            "referenceName": "AzureSqlTable_Dataset",
            "type": "DatasetReference"
        },
        "firstRowOnly": true
    }
}
```

**2. Execute Pipeline Activity - Call Another Pipeline:**

```json
{
    "name": "ExecuteChildPipeline",
    "type": "ExecutePipeline",
    "typeProperties": {
        "pipeline": {
            "referenceName": "ChildPipeline",
            "type": "PipelineReference"
        },
        "waitOnCompletion": true,
        "parameters": {
            "sourceFolder": "@pipeline().parameters.folder",
            "targetTable": "destination_table"
        }
    }
}
```

**3. Stored Procedure Activity:**

```json
{
    "name": "ExecuteStoredProcedure",
    "type": "SqlServerStoredProcedure",
    "typeProperties": {
        "storedProcedureName": "sp_ProcessDailyData",
        "storedProcedureParameters": {
            "date": {
                "value": "@formatDateTime(pipeline().TriggerTime, 'yyyy-MM-dd')",
                "type": "String"
            },
            "batchSize": {
                "value": "1000",
                "type": "Int32"
            }
        }
    },
    "linkedServiceName": {
        "referenceName": "AzureSqlDatabase_LS",
        "type": "LinkedServiceReference"
    }
}
```

**4. Get Metadata Activity:**

```json
{
    "name": "GetFileMetadata",
    "type": "GetMetadata",
    "typeProperties": {
        "dataset": {
            "referenceName": "CSV_Dataset",
            "type": "DatasetReference"
        },
        "fieldList": [
            "exists",
            "itemName",
            "lastModified",
            "size"
        ]
    }
}
```

**5. Delete Activity:**

```json
{
    "name": "DeleteOldFiles",
    "type": "Delete",
    "typeProperties": {
        "dataset": {
            "referenceName": "BlobDataset",
            "type": "DatasetReference",
            "parameters": {
                "folderPath": "archive"
            }
        },
        "enableLogging": true,
        "logStorageSettings": {
            "linkedServiceName": {
                "referenceName": "AzureBlobStorage_LS",
                "type": "LinkedServiceReference"
            },
            "path": "logs/delete"
        }
    }
}
```

**6. Web Activity - Call REST API:**

```json
{
    "name": "CallRestAPI",
    "type": "WebActivity",
    "typeProperties": {
        "url": "https://api.example.com/process",
        "method": "POST",
        "headers": {
            "Content-Type": "application/json",
            "Authorization": "Bearer @{pipeline().parameters.apiToken}"
        },
        "body": {
            "date": "@{formatDateTime(pipeline().TriggerTime, 'yyyy-MM-dd')}",
            "action": "process"
        }
    }
}
```

---

## 💧 LEVEL 4: DATA FLOWS

### 4.1 What are Data Flows?

**Mapping Data Flows:**
- Visual, code-free data transformation
- Built on Apache Spark (runs on Databricks clusters)
- Similar to SSIS Data Flow or Informatica mappings
- Drag-and-drop transformations

**When to Use Data Flows:**
✓ Complex transformations (joins, aggregations, pivots)
✓ Large datasets (GB to TB)
✓ No-code/low-code preference
✓ Reusable transformation logic

**When NOT to Use:**
✗ Simple copies (use Copy Activity)
✗ Small datasets (overhead not worth it)
✗ Need specific Python/Scala logic (use Databricks directly)

---

### 4.2 Data Flow Components

**Common Transformations:**

1. **Source** - Read data
2. **Filter** - Filter rows
3. **Select** - Select/rename columns
4. **Derived Column** - Add calculated columns
5. **Aggregate** - Group and aggregate
6. **Join** - Join datasets
7. **Lookup** - Enrich data
8. **Pivot/Unpivot** - Reshape data
9. **Sort** - Order rows
10. **Sink** - Write output

---

### 4.3 Example Data Flow

**Scenario: Sales Data Transformation**

```json
{
    "name": "SalesTransformationDataFlow",
    "properties": {
        "type": "MappingDataFlow",
        "typeProperties": {
            "sources": [
                {
                    "dataset": {
                        "referenceName": "SalesCSV",
                        "type": "DatasetReference"
                    },
                    "name": "SalesSource"
                },
                {
                    "dataset": {
                        "referenceName": "CustomersSQL",
                        "type": "DatasetReference"
                    },
                    "name": "CustomerSource"
                }
            ],
            "sinks": [
                {
                    "dataset": {
                        "referenceName": "SalesFactTable",
                        "type": "DatasetReference"
                    },
                    "name": "SalesSink"
                }
            ],
            "transformations": [
                {
                    "name": "FilterActive",
                    "type": "Filter",
                    "expression": "status == 'active'"
                },
                {
                    "name": "DerivedColumns",
                    "type": "DerivedColumn",
                    "columns": [
                        {
                            "name": "total_amount",
                            "expression": "quantity * unit_price"
                        },
                        {
                            "name": "sale_year",
                            "expression": "year(sale_date)"
                        }
                    ]
                },
                {
                    "name": "JoinCustomers",
                    "type": "Join",
                    "leftStream": "SalesSource",
                    "rightStream": "CustomerSource",
                    "joinType": "inner",
                    "joinCondition": "customer_id == cust_id"
                },
                {
                    "name": "AggregateByCustomer",
                    "type": "Aggregate",
                    "groupBy": ["customer_id", "customer_name"],
                    "aggregates": [
                        {
                            "name": "total_sales",
                            "expression": "sum(total_amount)"
                        },
                        {
                            "name": "order_count",
                            "expression": "count()"
                        }
                    ]
                }
            ]
        }
    }
}
```

**Data Flow Activity in Pipeline:**

```json
{
    "name": "ExecuteDataFlow",
    "type": "ExecuteDataFlow",
    "typeProperties": {
        "dataFlow": {
            "referenceName": "SalesTransformationDataFlow",
            "type": "DataFlowReference"
        },
        "compute": {
            "coreCount": 8,
            "computeType": "General"
        },
        "traceLevel": "Fine"
    }
}
```

---

### 4.4 Data Flow Expressions

**Common Expression Functions:**

```javascript
// String functions
upper(name)                          // JOHN DOE
lower(email)                         // john@example.com
concat(first_name, ' ', last_name)   // John Doe
substring(phone, 1, 3)               // Extract area code
trim(description)                    // Remove whitespace

// Date functions
currentDate()                        // Today's date
addDays(sale_date, 30)              // Add 30 days
year(sale_date)                      // Extract year
month(sale_date)                     // Extract month
datediff(end_date, start_date)      // Days between dates

// Math functions
round(price, 2)                      // Round to 2 decimals
abs(balance)                         // Absolute value
toInteger(string_value)              // Convert to int
toDecimal(price, 10, 2)             // Convert to decimal

// Conditional
iif(amount > 1000, 'High', 'Low')   // If-then-else
case(status == 'A', 'Active', 
     status == 'I', 'Inactive', 
     'Unknown')                      // Case statement

// Null handling
isNull(column)                       // Check if null
coalesce(price, 0)                  // Replace null with 0

// Aggregations (in Aggregate transformation)
sum(amount)
avg(amount)
count()
min(date)
max(date)
first(value)
last(value)
```

---

## ⏰ LEVEL 5: TRIGGERS & SCHEDULING

### 5.1 Trigger Types

**1. Schedule Trigger:**

```json
{
    "name": "DailySchedule",
    "properties": {
        "type": "ScheduleTrigger",
        "typeProperties": {
            "recurrence": {
                "frequency": "Day",
                "interval": 1,
                "startTime": "2024-01-01T00:00:00Z",
                "timeZone": "UTC",
                "schedule": {
                    "hours": [2],
                    "minutes": [0]
                }
            }
        },
        "pipelines": [
            {
                "pipelineReference": {
                    "referenceName": "DailyETLPipeline",
                    "type": "PipelineReference"
                },
                "parameters": {
                    "triggerTime": "@trigger().scheduledTime"
                }
            }
        ]
    }
}
```

**Schedule Patterns:**

```json
// Every hour
{
    "frequency": "Hour",
    "interval": 1
}

// Every 15 minutes
{
    "frequency": "Minute",
    "interval": 15
}

// Weekly on Monday at 9 AM
{
    "frequency": "Week",
    "interval": 1,
    "schedule": {
        "hours": [9],
        "minutes": [0],
        "weekDays": ["Monday"]
    }
}

// Monthly on 1st at midnight
{
    "frequency": "Month",
    "interval": 1,
    "schedule": {
        "hours": [0],
        "minutes": [0],
        "monthDays": [1]
    }
}

// Complex: Every weekday at 8 AM and 6 PM
{
    "frequency": "Day",
    "interval": 1,
    "schedule": {
        "hours": [8, 18],
        "minutes": [0],
        "weekDays": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"]
    }
}
```

---

### 5.2 Tumbling Window Trigger

**For backfilling and catching up:**

```json
{
    "name": "HourlyTumblingWindow",
    "properties": {
        "type": "TumblingWindowTrigger",
        "typeProperties": {
            "frequency": "Hour",
            "interval": 1,
            "startTime": "2024-01-01T00:00:00Z",
            "delay": "00:15:00",
            "maxConcurrency": 5,
            "retryPolicy": {
                "count": 3,
                "intervalInSeconds": 30
            }
        },
        "pipeline": {
            "pipelineReference": {
                "referenceName": "HourlyProcessingPipeline",
                "type": "PipelineReference"
            },
            "parameters": {
                "windowStart": "@trigger().outputs.windowStartTime",
                "windowEnd": "@trigger().outputs.windowEndTime"
            }
        }
    }
}
```

**Tumbling Window Features:**
- ✓ **Automatic backfilling** - Processes historical windows
- ✓ **Dependency tracking** - Wait for other pipelines
- ✓ **Guaranteed execution** - Each window runs exactly once
- ✓ **Retry logic** - Built-in retry on failure

**Self-Dependency (Wait for previous window):**

```json
{
    "typeProperties": {
        "dependsOn": [
            {
                "type": "SelfDependencyTumblingWindowTriggerReference",
                "offset": "-01:00:00",
                "size": "01:00:00"
            }
        ]
    }
}
```

---

### 5.3 Event-Based Trigger (Storage Event)

**Trigger on blob creation:**

```json
{
    "name": "BlobCreatedTrigger",
    "properties": {
        "type": "BlobEventsTrigger",
        "typeProperties": {
            "blobPathBeginsWith": "/datacontainer/raw/",
            "blobPathEndsWith": ".csv",
            "ignoreEmptyBlobs": true,
            "scope": "/subscriptions/{subscription-id}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/{storage-account}",
            "events": [
                "Microsoft.Storage.BlobCreated"
            ]
        },
        "pipelines": [
            {
                "pipelineReference": {
                    "referenceName": "ProcessNewFilePipeline",
                    "type": "PipelineReference"
                },
                "parameters": {
                    "fileName": "@triggerBody().fileName",
                    "folderPath": "@triggerBody().folderPath"
                }
            }
        ]
    }
}
```

---

### 5.4 Custom Event Trigger

**Trigger from Event Grid:**

```json
{
    "name": "CustomEventTrigger",
    "properties": {
        "type": "CustomEventsTrigger",
        "typeProperties": {
            "subjectBeginsWith": "orders/",
            "subjectEndsWith": "/completed",
            "events": [
                "OrderCompleted"
            ],
            "scope": "/subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.EventGrid/topics/{topic-name}"
        },
        "pipelines": [
            {
                "pipelineReference": {
                    "referenceName": "ProcessOrderPipeline",
                    "type": "PipelineReference"
                },
                "parameters": {
                    "orderId": "@triggerBody().data.orderId"
                }
            }
        ]
    }
}
```

---

## 🎛️ LEVEL 6: PARAMETERS & VARIABLES

### 6.1 Pipeline Parameters

**Define Parameters:**

```json
{
    "name": "ParameterizedPipeline",
    "properties": {
        "parameters": {
            "sourceFolder": {
                "type": "string",
                "defaultValue": "raw"
            },
            "targetTable": {
                "type": "string"
            },
            "processingDate": {
                "type": "string",
                "defaultValue": "@formatDateTime(utcnow(), 'yyyy-MM-dd')"
            },
            "batchSize": {
                "type": "int",
                "defaultValue": 1000
            }
        },
        "activities": [...]
    }
}
```

**Use Parameters in Activities:**

```json
{
    "name": "CopyActivity",
    "type": "Copy",
    "inputs": [
        {
            "referenceName": "SourceDataset",
            "type": "DatasetReference",
            "parameters": {
                "folderPath": "@pipeline().parameters.sourceFolder",
                "fileName": "@concat('data_', pipeline().parameters.processingDate, '.csv')"
            }
        }
    ]
}
```

---

### 6.2 Pipeline Variables

**Set and Use Variables:**

```json
{
    "name": "VariablePipeline",
    "properties": {
        "variables": {
            "fileCount": {
                "type": "Integer",
                "defaultValue": 0
            },
            "processedFiles": {
                "type": "Array"
            },
            "errorMessage": {
                "type": "String"
            }
        },
        "activities": [
            {
                "name": "SetFileCount",
                "type": "SetVariable",
                "typeProperties": {
                    "variableName": "fileCount",
                    "value": {
                        "value": "@activity('GetMetadata').output.childItems.length",
                        "type": "Expression"
                    }
                }
            },
            {
                "name": "AppendFileName",
                "type": "AppendVariable",
                "typeProperties": {
                    "variableName": "processedFiles",
                    "value": "@item().name"
                }
            }
        ]
    }
}
```

---

### 6.3 System Variables & Functions

**Common System Variables:**

```javascript
// Pipeline info
@pipeline().Pipeline                    // Pipeline name
@pipeline().RunId                       // Unique run ID
@pipeline().TriggerType                 // Manual, Schedule, etc.
@pipeline().TriggerName                 // Trigger name
@pipeline().TriggerTime                 // When triggered
@pipeline().parameters.paramName        // Pipeline parameter

// Trigger info
@trigger().scheduledTime                // Scheduled time
@trigger().startTime                    // Actual start time
@trigger().outputs.windowStartTime      // Tumbling window start
@trigger().outputs.windowEndTime        // Tumbling window end

// Activity info
@activity('ActivityName').output        // Activity output
@item()                                 // Current item in ForEach
@variables('variableName')              // Variable value

// Dataset info
@dataset().parameterName                // Dataset parameter
```

**Common Functions:**

```javascript
// String functions
@concat('Hello', ' ', 'World')          // HelloWorld
@substring('Hello', 0, 3)               // Hel
@replace('Hello', 'l', 'L')             // HeLLo
@toLower('HELLO')                       // hello
@toUpper('hello')                       // HELLO
@trim(' hello ')                        // hello

// Date/time functions
@utcnow()                               // Current UTC time
@formatDateTime(utcnow(), 'yyyy-MM-dd') // 2024-02-15
@addDays(utcnow(), -1)                  // Yesterday
@addHours(utcnow(), -5)                 // 5 hours ago
@startOfDay(utcnow())                   // Midnight today
@dayOfWeek(utcnow())                    // 0-6 (Sunday=0)

// Logical functions
@if(equals(1, 1), 'true', 'false')      // Conditional
@and(true, false)                       // AND operation
@or(true, false)                        // OR operation
@not(false)                             // NOT operation
@equals('a', 'b')                       // Equality check
@greater(10, 5)                         // Greater than
@less(5, 10)                            // Less than

// Conversion functions
@int('123')                             // Convert to integer
@string(123)                            // Convert to string
@bool('true')                           // Convert to boolean
@json('{"key":"value"}')                // Parse JSON

// Array functions
@length(array)                          // Array length
@first(array)                           // First element
@last(array)                            // Last element
@contains(array, 'value')               // Check if contains
```

---

## 🔀 LEVEL 7: CONTROL FLOW ACTIVITIES

### 7.1 ForEach Activity

**Process multiple items:**

```json
{
    "name": "ForEachFile",
    "type": "ForEach",
    "typeProperties": {
        "items": {
            "value": "@activity('GetFileList').output.childItems",
            "type": "Expression"
        },
        "isSequential": false,
        "batchCount": 10,
        "activities": [
            {
                "name": "ProcessFile",
                "type": "Copy",
                "inputs": [
                    {
                        "referenceName": "SourceDataset",
                        "type": "DatasetReference",
                        "parameters": {
                            "fileName": "@item().name"
                        }
                    }
                ],
                "outputs": [...]
            }
        ]
    }
}
```

**Parameters:**
- `isSequential: false` - Process in parallel (default: true)
- `batchCount: 10` - Max 10 parallel executions

---

### 7.2 If Condition Activity

**Conditional logic:**

```json
{
    "name": "IfFileExists",
    "type": "IfCondition",
    "typeProperties": {
        "expression": {
            "value": "@activity('CheckFile').output.exists",
            "type": "Expression"
        },
        "ifTrueActivities": [
            {
                "name": "ProcessFile",
                "type": "Copy",
                // ... copy configuration
            }
        ],
        "ifFalseActivities": [
            {
                "name": "SendAlert",
                "type": "WebActivity",
                // ... alert configuration
            }
        ]
    }
}
```

---

### 7.3 Switch Activity

**Multiple branches:**

```json
{
    "name": "SwitchByFileType",
    "type": "Switch",
    "typeProperties": {
        "on": {
            "value": "@pipeline().parameters.fileType",
            "type": "Expression"
        },
        "cases": [
            {
                "value": "csv",
                "activities": [
                    {
                        "name": "ProcessCSV",
                        "type": "Copy"
                    }
                ]
            },
            {
                "value": "json",
                "activities": [
                    {
                        "name": "ProcessJSON",
                        "type": "Copy"
                    }
                ]
            },
            {
                "value": "parquet",
                "activities": [
                    {
                        "name": "ProcessParquet",
                        "type": "Copy"
                    }
                ]
            }
        ],
        "defaultActivities": [
            {
                "name": "UnsupportedFormat",
                "type": "Fail",
                "typeProperties": {
                    "message": "Unsupported file format",
                    "errorCode": "1001"
                }
            }
        ]
    }
}
```

---

### 7.4 Until Activity

**Loop until condition is true:**

```json
{
    "name": "UntilComplete",
    "type": "Until",
    "typeProperties": {
        "expression": {
            "value": "@equals(variables('status'), 'completed')",
            "type": "Expression"
        },
        "timeout": "01:00:00",
        "activities": [
            {
                "name": "CheckJobStatus",
                "type": "WebActivity",
                "typeProperties": {
                    "url": "https://api.example.com/job/@{pipeline().parameters.jobId}/status",
                    "method": "GET"
                }
            },
            {
                "name": "UpdateStatus",
                "type": "SetVariable",
                "typeProperties": {
                    "variableName": "status",
                    "value": "@activity('CheckJobStatus').output.status"
                }
            },
            {
                "name": "Wait",
                "type": "Wait",
                "typeProperties": {
                    "waitTimeInSeconds": 30
                }
            }
        ]
    }
}
```

---

### 7.5 Wait Activity

**Pause execution:**

```json
{
    "name": "WaitForProcessing",
    "type": "Wait",
    "typeProperties": {
        "waitTimeInSeconds": 60
    }
}
```

---

### 7.6 Fail Activity

**Explicitly fail pipeline:**

```json
{
    "name": "FailPipeline",
    "type": "Fail",
    "typeProperties": {
        "message": "Data quality check failed: too many null values",
        "errorCode": "DQ001"
    }
}
```

---

## 📊 LEVEL 8: MONITORING & DEBUGGING

### 8.1 ADF Monitoring UI

**Key Monitoring Views:**

1. **Pipeline Runs** - All pipeline executions
2. **Activity Runs** - Individual activity status
3. **Trigger Runs** - Trigger execution history
4. **Integration Runtime** - IR health and metrics
5. **Alerts** - Configure alerts on failures

**Accessing Logs:**

```
1. ADF UI → Monitor tab
2. Filter by:
   - Status (Succeeded, Failed, In Progress)
   - Pipeline name
   - Run ID
   - Time range
3. Click pipeline run → View activity runs
4. Click activity → View details, input, output
5. View error messages, execution time
```

---

### 8.2 Debug Mode

**Test pipelines before deployment:**

```
1. ADF UI → Author tab
2. Open pipeline
3. Click "Debug" button
4. Provide parameter values
5. Watch execution in real-time
6. View activity outputs without saving
```

**Debug Features:**
- ✓ Test without affecting production
- ✓ View intermediate results
- ✓ Step-through debugging
- ✓ View data preview
- ✓ No cost for debug runs (up to 1000/month)

---

### 8.3 Azure Monitor Integration

**Set up Alerts:**

```json
{
    "name": "Pipeline Failure Alert",
    "type": "Metric",
    "criteria": {
        "metricName": "PipelineFailedRuns",
        "operator": "GreaterThan",
        "threshold": 0,
        "timeAggregation": "Total"
    },
    "actions": [
        {
            "actionGroupId": "/subscriptions/.../actionGroups/email-alerts"
        }
    ]
}
```

**Common Metrics:**
- PipelineSucceededRuns
- PipelineFailedRuns
- ActivitySucceededRuns
- ActivityFailedRuns
- TriggerSucceededRuns
- IntegrationRuntimeCpuPercentage
- IntegrationRuntimeAvailableMemory

---

### 8.4 Diagnostic Logs

**Send logs to Log Analytics:**

```
1. ADF → Diagnostic settings
2. Add diagnostic setting
3. Select:
   - ActivityRuns
   - PipelineRuns
   - TriggerRuns
   - SSISPackageEventMessages
4. Send to:
   - Log Analytics workspace
   - Storage Account
   - Event Hub
```

**Query logs with KQL:**

```kql
// Failed pipeline runs in last 24 hours
ADFPipelineRun
| where TimeGenerated > ago(24h)
| where Status == "Failed"
| project TimeGenerated, PipelineName, Status, ErrorMessage
| order by TimeGenerated desc

// Activities with longest duration
ADFActivityRun
| where TimeGenerated > ago(7d)
| summarize AvgDuration = avg(DurationInMs) by ActivityName, PipelineName
| order by AvgDuration desc
| take 10

// Error patterns
ADFActivityRun
| where Status == "Failed"
| summarize Count = count() by ErrorCode, ErrorMessage
| order by Count desc
```

---

## 🖥️ LEVEL 9: INTEGRATION RUNTIME

### 9.1 IR Types

**1. Azure Integration Runtime (Default):**
- Fully managed, serverless
- Cloud-to-cloud data movement
- Data flow execution
- Public endpoints only

**2. Self-Hosted Integration Runtime:**
- On-premises data sources
- Private network access
- Must install on Windows machine
- Supports cloud-to-on-prem scenarios

**3. Azure-SSIS Integration Runtime:**
- Run SSIS packages in cloud
- Lift-and-shift existing SSIS workloads

---

### 9.2 Creating Self-Hosted IR

**Step 1: Create IR in ADF:**

```
1. ADF → Manage → Integration runtimes
2. Click "New"
3. Select "Self-Hosted"
4. Name: "OnPremiseIR"
5. Click "Create"
6. Copy authentication key
```

**Step 2: Install on Windows Server:**

```powershell
# Download installer
Invoke-WebRequest -Uri "https://download.microsoft.com/download/..." -OutFile "IntegrationRuntime.msi"

# Install
Start-Process msiexec.exe -Wait -ArgumentList '/i IntegrationRuntime.msi /quiet'

# Register with key
cd "C:\Program Files\Microsoft Integration Runtime\5.0\Shared"
.\dmgcmd.exe -RegisterNewNode "<AUTH_KEY_FROM_ADF>"
```

**Use Self-Hosted IR:**

```json
{
    "name": "OnPremSQL_LS",
    "properties": {
        "type": "SqlServer",
        "typeProperties": {
            "connectionString": "Server=onprem-server;Database=mydb;..."
        },
        "connectVia": {
            "referenceName": "OnPremiseIR",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

---

### 9.3 IR Sizing & Performance

**Azure IR - Data Integration Units (DIU):**

```json
{
    "typeProperties": {
        "source": {...},
        "sink": {...},
        "dataIntegrationUnits": 32,  // 2-256
        "parallelCopies": 16
    }
}
```

**DIU Recommendations:**
- Small files (<100MB): 2-4 DIU
- Medium (100MB-1GB): 4-8 DIU
- Large (1GB-10GB): 8-16 DIU
- Very large (>10GB): 16-32+ DIU

**Self-Hosted IR - Node Scaling:**

```
1. Add more nodes for high availability
2. Each node can handle ~500 concurrent copy activities
3. Load balanced automatically
4. Monitor CPU/Memory on nodes
```

---

## 🚀 LEVEL 10: CI/CD & DEVOPS

### 10.1 Git Integration

**Connect ADF to Git:**

```
1. ADF UI → Manage → Git configuration
2. Configure repository:
   - Repository type: Azure DevOps / GitHub
   - Organization: your-org
   - Project: data-platform
   - Repository: adf-pipelines
   - Collaboration branch: main
   - Publish branch: adf_publish
   - Root folder: /
3. Import existing resources or start fresh
```

**Branch Strategy:**

```
main (collaboration branch)
  ├── feature/new-pipeline-1
  ├── feature/new-pipeline-2
  └── hotfix/fix-copy-activity

adf_publish (auto-generated ARM templates)
```

---

### 10.2 ARM Template Deployment

**Deploy ADF via ARM:**

```bash
# Get published ARM template from adf_publish branch
git checkout adf_publish

# Deploy to target environment
az deployment group create \
  --resource-group rg-adf-prod \
  --template-file ARMTemplateForFactory.json \
  --parameters ARMTemplateParametersForFactory.json \
  --parameters factoryName=adf-demo-prod \
               AzureSqlDatabase_LS_connectionString="Server=prod-server..."
```

**Parameters File (ARMTemplateParametersForFactory.json):**

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "factoryName": {
            "value": "adf-demo-prod"
        },
        "AzureSqlDatabase_LS_connectionString": {
            "value": "Server=tcp:prod-server.database.windows.net,1433;..."
        },
        "AzureBlobStorage_LS_connectionString": {
            "value": "DefaultEndpointsProtocol=https;AccountName=prodstorageacct;..."
        }
    }
}
```

---

### 10.3 Azure DevOps Pipeline

**Complete CI/CD Pipeline:**

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main

pool:
  vmImage: 'windows-latest'

variables:
  azureSubscription: 'Azure-Subscription-Connection'
  resourceGroupName: 'rg-adf-prod'
  dataFactoryName: 'adf-demo-prod'

stages:
- stage: Build
  displayName: 'Build ADF artifacts'
  jobs:
  - job: Build
    steps:
    - task: NodeTool@0
      inputs:
        versionSpec: '14.x'
      displayName: 'Install Node.js'

    - task: Npm@1
      inputs:
        command: 'install'
        workingDir: '$(Build.Repository.LocalPath)/build'
      displayName: 'Install npm packages'

    - task: Npm@1
      inputs:
        command: 'custom'
        workingDir: '$(Build.Repository.LocalPath)/build'
        customCommand: 'run build export $(Build.Repository.LocalPath) /subscriptions/$(subscriptionId)/resourceGroups/$(resourceGroupName)/providers/Microsoft.DataFactory/factories/$(dataFactoryName) "ArmTemplate"'
      displayName: 'Validate and Generate ARM template'

    - task: PublishPipelineArtifact@1
      inputs:
        targetPath: '$(Build.Repository.LocalPath)/build/ArmTemplate'
        artifact: 'ArmTemplates'
      displayName: 'Publish ARM templates'

- stage: Deploy_Dev
  displayName: 'Deploy to Dev'
  dependsOn: Build
  condition: succeeded()
  jobs:
  - deployment: Deploy
    environment: 'ADF-Dev'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: DownloadPipelineArtifact@2
            inputs:
              artifactName: 'ArmTemplates'
              targetPath: '$(Pipeline.Workspace)/ArmTemplates'

          - task: AzurePowerShell@5
            inputs:
              azureSubscription: '$(azureSubscription)'
              ScriptType: 'FilePath'
              ScriptPath: '$(Pipeline.Workspace)/PrePostDeploymentScript.ps1'
              ScriptArguments: '-armTemplate "$(Pipeline.Workspace)/ArmTemplates/ARMTemplateForFactory.json" -ResourceGroupName $(resourceGroupName) -DataFactoryName $(dataFactoryName) -predeployment $true'
              azurePowerShellVersion: 'LatestVersion'
            displayName: 'Pre-deployment: Stop triggers'

          - task: AzureResourceManagerTemplateDeployment@3
            inputs:
              deploymentScope: 'Resource Group'
              azureResourceManagerConnection: '$(azureSubscription)'
              subscriptionId: '$(subscriptionId)'
              action: 'Create Or Update Resource Group'
              resourceGroupName: '$(resourceGroupName)'
              location: 'East US'
              templateLocation: 'Linked artifact'
              csmFile: '$(Pipeline.Workspace)/ArmTemplates/ARMTemplateForFactory.json'
              csmParametersFile: '$(Pipeline.Workspace)/ArmTemplates/ARMTemplateParametersForFactory.json'
              overrideParameters: '-factoryName "$(dataFactoryName)" -AzureSqlDatabase_LS_connectionString "$(SqlConnectionString)"'
              deploymentMode: 'Incremental'
            displayName: 'Deploy ARM template'

          - task: AzurePowerShell@5
            inputs:
              azureSubscription: '$(azureSubscription)'
              ScriptType: 'FilePath'
              ScriptPath: '$(Pipeline.Workspace)/PrePostDeploymentScript.ps1'
              ScriptArguments: '-armTemplate "$(Pipeline.Workspace)/ArmTemplates/ARMTemplateForFactory.json" -ResourceGroupName $(resourceGroupName) -DataFactoryName $(dataFactoryName) -predeployment $false'
              azurePowerShellVersion: 'LatestVersion'
            displayName: 'Post-deployment: Start triggers'

- stage: Deploy_Prod
  displayName: 'Deploy to Production'
  dependsOn: Deploy_Dev
  condition: succeeded()
  jobs:
  - deployment: Deploy
    environment: 'ADF-Prod'
    strategy:
      runOnce:
        deploy:
          steps:
          # Same deployment steps as Dev
          # But with production parameters
```

---

### 10.4 Environment Management

**Best Practices:**

```
Environments:
├── Dev (adf-demo-dev)
│   ├── Connection: dev-sql-server
│   ├── Storage: devstorage
│   └── Triggers: Disabled by default
│
├── UAT (adf-demo-uat)
│   ├── Connection: uat-sql-server
│   ├── Storage: uatstorage
│   └── Triggers: On-demand only
│
└── Prod (adf-demo-prod)
    ├── Connection: prod-sql-server
    ├── Storage: prodstorage
    └── Triggers: Fully automated
```

**Parameter Files per Environment:**

```json
// parameters.dev.json
{
    "factoryName": "adf-demo-dev",
    "environment": "dev",
    "sqlServer": "dev-sql-server.database.windows.net",
    "storageAccount": "devstorage"
}

// parameters.prod.json
{
    "factoryName": "adf-demo-prod",
    "environment": "prod",
    "sqlServer": "prod-sql-server.database.windows.net",
    "storageAccount": "prodstorage"
}
```

---

## 🏗️ REAL-WORLD PROJECTS

### Project 1: Incremental Data Load

**Scenario: Load only new/changed records from SQL to Data Lake**

```json
{
    "name": "IncrementalLoadPipeline",
    "properties": {
        "parameters": {
            "tableName": {"type": "string"}
        },
        "activities": [
            {
                "name": "GetLastWatermark",
                "type": "Lookup",
                "typeProperties": {
                    "source": {
                        "type": "AzureSqlSource",
                        "sqlReaderQuery": "SELECT MAX(LastModifiedDate) as Watermark FROM WatermarkTable WHERE TableName = '@{pipeline().parameters.tableName}'"
                    },
                    "dataset": {"referenceName": "ControlTableDataset"}
                }
            },
            {
                "name": "GetNewWatermark",
                "type": "Lookup",
                "typeProperties": {
                    "source": {
                        "type": "AzureSqlSource",
                        "sqlReaderQuery": "SELECT MAX(LastModifiedDate) as NewWatermark FROM @{pipeline().parameters.tableName}"
                    },
                    "dataset": {"referenceName": "SourceTableDataset"}
                }
            },
            {
                "name": "CopyIncrementalData",
                "type": "Copy",
                "dependsOn": [
                    {"activity": "GetLastWatermark"},
                    {"activity": "GetNewWatermark"}
                ],
                "typeProperties": {
                    "source": {
                        "type": "AzureSqlSource",
                        "sqlReaderQuery": "SELECT * FROM @{pipeline().parameters.tableName} WHERE LastModifiedDate > '@{activity('GetLastWatermark').output.firstRow.Watermark}' AND LastModifiedDate <= '@{activity('GetNewWatermark').output.firstRow.NewWatermark}'"
                    },
                    "sink": {
                        "type": "ParquetSink",
                        "storeSettings": {
                            "type": "AzureBlobFSWriteSettings",
                            "copyBehavior": "PreserveHierarchy"
                        }
                    }
                },
                "inputs": [{"referenceName": "SourceTableDataset"}],
                "outputs": [{"referenceName": "DataLakeDataset"}]
            },
            {
                "name": "UpdateWatermark",
                "type": "SqlServerStoredProcedure",
                "dependsOn": [{"activity": "CopyIncrementalData"}],
                "typeProperties": {
                    "storedProcedureName": "usp_UpdateWatermark",
                    "storedProcedureParameters": {
                        "TableName": {"value": "@{pipeline().parameters.tableName}"},
                        "Watermark": {"value": "@{activity('GetNewWatermark').output.firstRow.NewWatermark}"}
                    }
                }
            }
        ]
    }
}
```

---

### Project 2: File Ingestion Framework

**Scenario: Automatically process files dropped to blob storage**

```json
{
    "name": "FileIngestionFramework",
    "properties": {
        "activities": [
            {
                "name": "GetFileList",
                "type": "GetMetadata",
                "typeProperties": {
                    "dataset": {"referenceName": "IncomingFolderDataset"},
                    "fieldList": ["childItems"]
                }
            },
            {
                "name": "ForEachFile",
                "type": "ForEach",
                "dependsOn": [{"activity": "GetFileList"}],
                "typeProperties": {
                    "items": "@activity('GetFileList').output.childItems",
                    "isSequential": false,
                    "batchCount": 20,
                    "activities": [
                        {
                            "name": "ProcessFile",
                            "type": "ExecutePipeline",
                            "typeProperties": {
                                "pipeline": {"referenceName": "ProcessSingleFilePipeline"},
                                "waitOnCompletion": true,
                                "parameters": {
                                    "fileName": "@item().name",
                                    "fileType": "@substring(item().name, lastIndexOf(item().name, '.'), sub(length(item().name), lastIndexOf(item().name, '.')))"
                                }
                            }
                        }
                    ]
                }
            }
        ]
    }
}
```

---

### Project 3: Data Quality Check Framework

```json
{
    "name": "DataQualityPipeline",
    "properties": {
        "activities": [
            {
                "name": "CheckRowCount",
                "type": "Lookup",
                "typeProperties": {
                    "source": {
                        "type": "AzureSqlSource",
                        "sqlReaderQuery": "SELECT COUNT(*) as RowCount FROM staging_table"
                    }
                }
            },
            {
                "name": "ValidateRowCount",
                "type": "IfCondition",
                "dependsOn": [{"activity": "CheckRowCount"}],
                "typeProperties": {
                    "expression": "@greater(activity('CheckRowCount').output.firstRow.RowCount, 100)",
                    "ifTrueActivities": [
                        {
                            "name": "CheckNulls",
                            "type": "Lookup",
                            "typeProperties": {
                                "source": {
                                    "sqlReaderQuery": "SELECT COUNT(*) as NullCount FROM staging_table WHERE customer_id IS NULL"
                                }
                            }
                        },
                        {
                            "name": "ValidateNulls",
                            "type": "IfCondition",
                            "dependsOn": [{"activity": "CheckNulls"}],
                            "typeProperties": {
                                "expression": "@equals(activity('CheckNulls').output.firstRow.NullCount, 0)",
                                "ifTrueActivities": [
                                    {"name": "LoadToProduction", "type": "Copy"}
                                ],
                                "ifFalseActivities": [
                                    {
                                        "name": "FailPipeline",
                                        "type": "Fail",
                                        "typeProperties": {
                                            "message": "Data quality check failed: Null values found",
                                            "errorCode": "DQ001"
                                        }
                                    }
                                ]
                            }
                        }
                    ],
                    "ifFalseActivities": [
                        {
                            "name": "FailPipeline",
                            "type": "Fail",
                            "typeProperties": {
                                "message": "Data quality check failed: Insufficient rows",
                                "errorCode": "DQ002"
                            }
                        }
                    ]
                }
            }
        ]
    }
}
```

---

## ✅ ADF MASTERY CHECKLIST

### Beginner ✓
- [ ] Create Data Factory in Azure Portal
- [ ] Create Linked Services (Blob, SQL)
- [ ] Create Datasets
- [ ] Build basic Copy pipeline
- [ ] Run pipeline manually

### Intermediate ✓
- [ ] Use parameters and variables
- [ ] Implement control flow (ForEach, If)
- [ ] Create schedule triggers
- [ ] Debug pipelines
- [ ] Monitor pipeline runs

### Advanced ✓
- [ ] Build mapping data flows
- [ ] Implement incremental loads
- [ ] Use tumbling window triggers
- [ ] Create dynamic pipelines
- [ ] Setup self-hosted IR

### Expert ✓
- [ ] Implement CI/CD with Git
- [ ] Deploy via ARM templates
- [ ] Build reusable frameworks
- [ ] Optimize performance (DIU, parallelism)
- [ ] Implement comprehensive monitoring

---

## 📚 LEARNING RESOURCES

**Official Microsoft:**
- ADF Documentation: https://docs.microsoft.com/azure/data-factory/
- Microsoft Learn: Free ADF courses
- Azure Samples: https://github.com/Azure-Samples/

**Practice:**
- Free Azure account ($200 credit)
- ADF has free tier (1000 activities/month)
- Build real projects

**Certifications:**
- DP-203: Data Engineering on Azure
- DP-900: Azure Data Fundamentals

---

## 🎯 30-DAY LEARNING PLAN

**Week 1: Foundations**
- Days 1-2: Create ADF, linked services, datasets
- Days 3-4: Copy activities, basic pipelines
- Days 5-7: Parameters, triggers, monitoring

**Week 2: Control Flow**
- Days 8-10: ForEach, If Condition, Switch
- Days 11-12: Lookup, Get Metadata
- Days 13-14: Build file ingestion framework

**Week 3: Data Flows & Advanced**
- Days 15-17: Mapping data flows
- Days 18-19: Incremental loads
- Days 20-21: Self-hosted IR

**Week 4: Production**
- Days 22-24: CI/CD setup, Git integration
- Days 25-27: Monitoring, alerts
- Days 28-30: Build complete ETL solution

---

**You've mastered ADF when:**
✓ You can build complex ETL pipelines visually
✓ You understand when to use Copy vs Data Flow
✓ You can implement incremental loads
✓ You know how to optimize performance
✓ You can deploy via CI/CD
✓ You're comfortable debugging production issues

**What's next in your data journey?**
We've covered: SQL, Snowflake, Airflow, ADF

**Continue with:**
1. **Python for Data Engineering** - APIs, pandas, automation
2. **dbt** - SQL-based transformations
3. **Power BI** - Visualization and dashboards
4. **PySpark** - Big data processing
5. **Databricks** - Unified analytics platform

Which one next? 🚀
