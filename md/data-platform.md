# Phase 3: The Data Platform Layer — Deep Dive

> **Related pages:** [Phase 3 Study Guide (HTML)](../html/data-platform.html) | [Fabric Overview](../html/fabric-overview.html)
> **Official docs:** [Microsoft Fabric Documentation](https://learn.microsoft.com/en-us/fabric/get-started/microsoft-fabric-overview)

---

## Data Ingestion Patterns

Fabric offers three primary ways to get data into OneLake. Choosing the right one depends on the persona, complexity, and whether the task is orchestration or transformation.

### Pipelines (Data Factory)

- **What:** Visual orchestration tool — sequences activities (copy, notebook, dataflow, stored proc) into workflows
- **When to use:** Scheduled batch ingestion, multi-step ETL workflows, copying data from external sources into OneLake
- **Key features:** 200+ connectors, parameterisation, error handling, retry logic, triggers (schedule, event, tumbling window)
- **Think of it as:** The conductor — it does not transform data itself, it tells other tools when and how to run
- **Familiar to:** Azure Data Factory users — the pipeline experience is nearly identical

### Dataflows Gen2 (Power Query Online)

- **What:** Low-code visual transformation using Power Query (M language under the hood)
- **When to use:** Business analyst-driven ETL, simple transformations, data cleansing without code
- **Key features:** Visual UI, auto-generate M code, direct output to Lakehouse/Warehouse, reusable dataflows
- **Think of it as:** Excel Power Query but running in the cloud at scale
- **Familiar to:** Power BI Desktop users who already use Power Query

### Notebooks (Spark)

- **What:** Code-first transformation using PySpark, SparkSQL, Scala, or R
- **When to use:** Complex transformations, data engineering at scale, ML feature engineering, anything needing programmatic control
- **Key features:** Full Spark engine, Delta Lake native, parameterised runs, integration with pipelines
- **Think of it as:** Databricks notebooks but fully integrated into Fabric
- **Familiar to:** Data engineers and data scientists

### Decision Guide: When to Use Each

| Scenario | Tool | Why |
|----------|------|-----|
| Copy data from SQL Server to Lakehouse on schedule | **Pipeline** | Orchestration + copy activity |
| Clean and reshape a CSV before loading | **Dataflow Gen2** | Low-code, visual transformation |
| Process 10 GB of log files with custom logic | **Notebook** | Code-first, Spark scale |
| Run a notebook, then a dataflow, then refresh a dataset | **Pipeline** | Orchestration of multiple steps |
| Business analyst needs to merge two Excel files | **Dataflow Gen2** | Familiar Power Query experience |
| Apply medallion architecture transforms | **Notebook** | Full control over Delta operations |

---

## Apache Spark in Fabric

### What Fabric Provides

- Managed Spark clusters — no infrastructure to provision or manage
- Starter pools for instant-on (no cold start for small workloads)
- Notebooks with multi-language support: PySpark, SparkSQL, Scala, R
- Native Delta Lake integration — all tables are Delta by default
- V-Order optimisation applied automatically for Power BI performance

### Key Concepts

**Spark Pools and Compute:**
- Fabric manages Spark compute per workspace
- Starter pools provide instant access (small cluster, shared)
- Custom environments allow installing additional libraries (pip, conda)
- Autoscale adjusts nodes based on workload
- Session timeout reclaims idle resources

**Notebooks:**
- Multi-language cells in a single notebook (%%pyspark, %%sql, %%scala)
- Built-in visualisation for dataframes
- Parameterised notebooks for pipeline integration
- Git integration for version control
- Collaboration with co-authoring

**PySpark Basics:**

```python
# Read from Lakehouse Files
df = spark.read.format("csv").option("header", "true").load("Files/raw/sales.csv")

# Transform
from pyspark.sql.functions import col, year, sum
df_clean = (df
    .filter(col("amount") > 0)
    .withColumn("year", year(col("date")))
    .groupBy("year", "region")
    .agg(sum("amount").alias("total_sales"))
)

# Write as Delta table to Lakehouse Tables
df_clean.write.format("delta").mode("overwrite").save("Tables/gold_sales")
```

**SparkSQL:**

```sql
-- Query Delta tables directly
SELECT year, region, SUM(amount) as total_sales
FROM lakehouse.raw_sales
WHERE amount > 0
GROUP BY year, region
```

---

## Delta Lake Deep Dive

### What is Delta Lake?

Delta Lake is an open-source storage layer that brings ACID transactions to data lakes. In Fabric, Delta is the default and only table format — every table in a Lakehouse or Warehouse is stored as Delta (Parquet files + a JSON transaction log).

### Core Features

**ACID Transactions:**
- Every write (insert, update, delete, merge) is atomic
- Readers never see partial writes
- Concurrent reads and writes are safe
- Transaction log (`_delta_log/`) records every change as a JSON commit

**Time Travel:**
- Query data as it existed at any previous version or timestamp
- Useful for auditing, debugging, and reproducing results

```sql
-- Query a specific version
SELECT * FROM my_table VERSION AS OF 5

-- Query a specific timestamp
SELECT * FROM my_table TIMESTAMP AS OF '2026-03-01'
```

**Schema Evolution:**
- Add new columns without rewriting existing data
- `mergeSchema` option allows automatic schema updates during writes
- Schema enforcement rejects writes that do not match the table schema (default behaviour)

```python
df.write.format("delta").option("mergeSchema", "true").mode("append").save("Tables/my_table")
```

**Optimisation Commands:**
- `OPTIMIZE` — compacts small files into larger ones for better read performance
- `VACUUM` — removes old files no longer referenced by the transaction log (default 7-day retention)
- `V-Order` — Fabric-specific optimisation that reorders data within Parquet files for fastest Power BI reads

```sql
-- Compact small files
OPTIMIZE my_table

-- Z-Order by commonly filtered column
OPTIMIZE my_table ZORDER BY (region)

-- Clean up old files (default: older than 7 days)
VACUUM my_table
```

**Table Maintenance Best Practice:**
1. Run `OPTIMIZE` after large batch writes
2. Use Z-Order on columns frequently used in filters or joins
3. Schedule `VACUUM` to reclaim storage (be aware of time travel retention)
4. V-Order is applied automatically in Fabric — no manual action needed

---

## Medallion Architecture Expanded

The medallion architecture (also called multi-hop) organises data in a Lakehouse into three layers of increasing quality and structure.

### Bronze Layer (Raw)

- **Purpose:** Land raw data exactly as it arrives from source systems
- **Format:** Append-only, preserves original schema and values
- **Examples:** Raw CSV files, JSON API responses, database CDC streams
- **Transformations:** None — this is the "single source of truth" for raw data
- **Retention:** Keep indefinitely for auditability and reprocessing

### Silver Layer (Cleansed)

- **Purpose:** Apply quality rules, deduplication, and standard schema
- **Format:** Delta tables with enforced schema, partitioned where appropriate
- **Examples:** Deduplicated customer records, validated transactions, joined reference data
- **Transformations:** Type casting, null handling, deduplication, joins, filtering invalid records
- **Pattern:** Read from Bronze, transform, write to Silver

### Gold Layer (Business-Ready)

- **Purpose:** Aggregated, business-oriented tables ready for reporting and analytics
- **Format:** Delta tables optimised for Power BI consumption (V-Order, Z-Order)
- **Examples:** Monthly sales summary, customer lifetime value, KPI tables
- **Transformations:** Aggregation, business logic, dimensional modelling
- **Consumed by:** Power BI (via DirectLake), data science notebooks, APIs

### Flow

```
Sources → Bronze (raw, append-only)
           → Silver (cleansed, deduplicated, schema-enforced)
              → Gold (aggregated, business-ready, optimised)
                 → Power BI / Data Science / APIs
```

---

## Lakehouse vs Warehouse — Detailed Comparison

| Dimension | Lakehouse | Warehouse |
|-----------|-----------|-----------|
| **Storage format** | Delta Lake (Parquet + log) in OneLake | Delta Lake in OneLake |
| **Query engine** | Apache Spark + SQL analytics endpoint | T-SQL engine |
| **Primary language** | PySpark, SparkSQL, Scala, R | T-SQL |
| **Best for** | Data engineering, ML, mixed workloads | SQL analytics, BI, familiar DW patterns |
| **Schema management** | Schema-on-read (flexible) | Schema-on-write (enforced) |
| **Indexing** | Z-Order, V-Order | Automatic indexing, statistics |
| **Stored procedures** | No | Yes (T-SQL) |
| **Cross-database queries** | Via shortcuts | Via cross-warehouse queries |
| **Security** | OneLake RBAC, workspace roles | T-SQL RBAC, row-level security, column-level security, dynamic data masking |
| **Ideal persona** | Data engineer, data scientist | Data analyst, BI developer, DBA |
| **DirectLake support** | Yes | Yes |
| **When to choose** | Unstructured/semi-structured data, Python-heavy, ML pipelines | Structured data, T-SQL expertise, traditional DW migration |

**Key insight:** Both store data in the same format (Delta) in OneLake. The difference is the compute engine and the developer experience. Many organisations use both — Lakehouse for engineering and science, Warehouse for structured analytics.

---

## Pipelines vs Dataflows vs Notebooks — Decision Table

| Dimension | Pipelines | Dataflows Gen2 | Notebooks |
|-----------|-----------|----------------|-----------|
| **Purpose** | Orchestrate workflows | Low-code ETL | Code-first transformation |
| **Language** | Visual designer (JSON under the hood) | M / Power Query | PySpark, SparkSQL, Scala, R |
| **Persona** | Data engineer, architect | Business analyst, citizen developer | Data engineer, data scientist |
| **Transforms data?** | No (orchestrates tools that do) | Yes | Yes |
| **Scale** | N/A (orchestration) | Medium (Power Query engine) | Large (Spark distributed compute) |
| **Scheduling** | Built-in triggers | Via pipelines | Via pipelines |
| **Error handling** | Retry, branching, alerts | Basic | Programmatic (try/except) |
| **Best for** | Multi-step workflows, scheduling | Simple cleansing, merging, reshaping | Complex logic, large-scale processing |
| **Typical combo** | Pipeline calls Notebook + Dataflow | Embedded in a Pipeline | Called by a Pipeline |

**Common pattern:** A Pipeline orchestrates the entire workflow — it triggers a Notebook for heavy Spark processing, then a Dataflow Gen2 for final business-friendly cleansing, then refreshes the semantic model.

---

## DirectLake Mode Explained

### What is DirectLake?

DirectLake is a Power BI connectivity mode unique to Fabric. It reads Delta tables directly from OneLake — no data import, no DirectQuery translation. The VertiPaq engine loads column segments on demand from Parquet files.

### How It Works

1. Delta tables are stored in OneLake as Parquet files (with V-Order optimisation)
2. Power BI semantic model points to the Delta table (no import, no scheduled refresh)
3. When a report visual fires a query, VertiPaq loads the required column segments from Parquet into memory
4. Subsequent queries on the same columns hit the in-memory cache
5. When underlying data changes, the next query reads the updated Parquet files

### Comparison

| Mode | Data Location | Freshness | Performance | Capacity Cost |
|------|--------------|-----------|-------------|---------------|
| **Import** | Copied into Power BI dataset | Stale until refresh | Fastest (all in memory) | Storage + refresh compute |
| **DirectQuery** | Stays in source | Real-time | Slowest (query translation) | Query compute |
| **DirectLake** | OneLake (Delta/Parquet) | Near real-time | Fast (on-demand memory load) | Read compute |

### Limitations

- Only works with Delta tables in Fabric Lakehouse or Warehouse
- Falls back to DirectQuery if data exceeds memory limits (per SKU)
- Not all DAX patterns are supported in DirectLake (some force fallback)
- Requires V-Order for optimal performance (applied automatically in Fabric)
- Row-level security requires careful configuration

### When to Use

- **Use DirectLake when:** Your data is in Fabric, you want import-like speed with near-real-time freshness, and your dataset fits within SKU memory limits
- **Fall back to Import when:** You need guaranteed in-memory performance for very large datasets, or data is outside Fabric
- **Fall back to DirectQuery when:** You need true real-time data from an external source, or dataset is too large for memory

---

## Real-Time Intelligence

### Architecture

Real-Time Intelligence (RTI) is the streaming analytics workload in Fabric. It handles event-driven data — IoT telemetry, application logs, clickstreams, financial ticks.

### Key Components

**Eventstream:**
- Ingests real-time data from sources (Event Hubs, Kafka, custom apps, IoT Hub)
- Visual no-code designer for routing and basic transformations
- Supports multiple destinations (Eventhouse, Lakehouse, custom endpoints)

**Eventhouse:**
- Optimised for time-series and log analytics
- Built on Azure Data Explorer (Kusto) engine
- Contains one or more KQL Databases
- Auto-scales ingestion and query compute independently

**KQL (Kusto Query Language):**
- Purpose-built query language for time-series and log data
- Pipe-based syntax: `TableName | where Timestamp > ago(1h) | summarize count() by bin(Timestamp, 5m)`
- Much faster than SQL for log/telemetry patterns
- Supports anomaly detection, forecasting, and pattern matching natively

**Real-Time Dashboards:**
- KQL-powered dashboards that auto-refresh
- Tiles bound to KQL queries
- Parameters for interactive filtering

### RTI to Fabric IQ Pipeline

RTI feeds directly into Fabric IQ for AI-powered operations:

```
Eventstream → Eventhouse → KQL Database → Ontology bindings → Operations Agent
```

1. **Eventstream** ingests streaming data from IoT sensors, application logs, etc.
2. **Eventhouse** stores and indexes the data in KQL Databases
3. **KQL Database** provides fast query access for time-series analysis
4. **Ontology** (Fabric IQ) maps KQL data to business concepts and rules
5. **Operations Agent** monitors conditions and triggers automated actions

This is the path from raw streaming data to AI-driven operational intelligence.

---

## MS Learn Modules

### Data Ingestion & Orchestration

| Module | Duration | What You Learn |
|--------|----------|----------------|
| [Use Apache Spark in Microsoft Fabric](https://learn.microsoft.com/en-us/training/modules/use-apache-spark-work-files-fabric/) | ~45 min | Spark notebooks, compute engine, reading/writing files and Delta tables |
| [Work with Delta Lake tables in Microsoft Fabric](https://learn.microsoft.com/en-us/training/modules/work-delta-lake-tables-fabric/) | ~45 min | Delta format, ACID transactions, time travel, table maintenance |
| [Orchestrate processes with Microsoft Fabric](https://learn.microsoft.com/en-us/training/modules/use-data-factory-pipelines-fabric/) | ~30 min | Data Factory pipelines, activities, scheduling, error handling |
| [Ingest data with Dataflows Gen2 in Microsoft Fabric](https://learn.microsoft.com/en-us/training/modules/use-dataflow-gen-2-fabric/) | ~30 min | Power Query Online, visual ETL, outputting to Lakehouse |

### Real-Time Intelligence

| Module | Duration | What You Learn |
|--------|----------|----------------|
| [Get started with Real-Time Intelligence in Microsoft Fabric](https://learn.microsoft.com/en-us/training/modules/get-started-kusto-fabric/) | ~30 min | Eventhouses, KQL basics, streaming ingestion |
| [Implement Real-Time Intelligence with Microsoft Fabric](https://learn.microsoft.com/en-us/training/paths/explore-real-time-analytics-microsoft-fabric/) | ~2 hrs | Applied Skill — end-to-end RTI scenario |

### Full Learning Path

[Get started with Microsoft Fabric](https://learn.microsoft.com/en-us/training/paths/get-started-fabric/) — 10-module path covering all Fabric workloads.

---

## Hands-On Exercise: End-to-End Lakehouse Flow

### Prerequisites

- Fabric trial capacity (free 60-day trial)
- A workspace assigned to the trial capacity

### Step-by-Step

**Step 1 — Create a Lakehouse**
1. Navigate to your Fabric workspace
2. Select **New item** > **Lakehouse**
3. Name it `phase3_exercise`
4. Note the two sections: **Files** (raw storage) and **Tables** (Delta tables)

**Step 2 — Ingest via Pipeline**
1. Create a new **Data pipeline** in the same workspace
2. Add a **Copy data** activity
3. Source: use a built-in sample dataset (e.g., NYC Taxi from the sample data gallery) or upload a CSV to Lakehouse Files
4. Destination: your Lakehouse Files folder (e.g., `Files/raw/`)
5. Run the pipeline — verify files appear in the Lakehouse

**Step 3 — Transform with Notebook**
1. Open the Lakehouse and select **Open notebook** > **New notebook**
2. Or use a built-in sample notebook if available
3. Write PySpark to read the raw files:
   ```python
   df = spark.read.format("csv").option("header", "true").option("inferSchema", "true").load("Files/raw/")
   display(df.limit(10))
   ```
4. Apply basic transforms:
   ```python
   from pyspark.sql.functions import col, year, month
   df_clean = (df
       .filter(col("fare_amount") > 0)
       .withColumn("trip_year", year(col("pickup_datetime")))
       .withColumn("trip_month", month(col("pickup_datetime")))
   )
   ```
5. Write as Delta table:
   ```python
   df_clean.write.format("delta").mode("overwrite").saveAsTable("phase3_exercise.silver_trips")
   ```
6. Verify the table appears in the Lakehouse Tables section

**Step 4 — Create Gold Aggregation**
```python
df_gold = (spark.table("phase3_exercise.silver_trips")
    .groupBy("trip_year", "trip_month")
    .agg(
        {"fare_amount": "sum", "trip_distance": "avg", "*": "count"}
    )
    .withColumnRenamed("sum(fare_amount)", "total_fares")
    .withColumnRenamed("avg(trip_distance)", "avg_distance")
    .withColumnRenamed("count(1)", "trip_count")
)
df_gold.write.format("delta").mode("overwrite").saveAsTable("phase3_exercise.gold_monthly_summary")
```

**Step 5 — Connect Power BI with DirectLake**
1. In the Lakehouse, click **New semantic model**
2. Select the `gold_monthly_summary` table
3. Open the semantic model and select **New report**
4. Build a simple bar chart: trip_month on axis, total_fares as value
5. Notice: no import, no scheduled refresh — DirectLake reads directly from Delta

**Step 6 — Verify the Full Flow**
- Pipeline ingested raw data into Lakehouse Files
- Notebook transformed and wrote Delta tables (Silver and Gold)
- Power BI reports via DirectLake from Gold tables
- This is the core Fabric data platform pattern: **Ingest → Transform → Store → Report**

---

## Key Terms Glossary

| Term | Definition |
|------|-----------|
| **ACID** | Atomicity, Consistency, Isolation, Durability — transaction guarantees provided by Delta Lake |
| **Bronze / Silver / Gold** | Medallion architecture layers: raw, cleansed, business-ready |
| **CU (Capacity Unit)** | Fabric compute billing unit — all workloads consume CUs from a shared capacity pool |
| **Dataflows Gen2** | Low-code Power Query-based ETL tool in Fabric Data Factory |
| **Delta Lake** | Open-source storage layer providing ACID transactions on Parquet files |
| **DirectLake** | Power BI connectivity mode that reads Delta tables from OneLake without import or DirectQuery |
| **Eventhouse** | RTI component for storing and querying time-series/log data using KQL |
| **Eventstream** | RTI component for ingesting real-time streaming data |
| **KQL** | Kusto Query Language — pipe-based query language optimised for log and time-series analytics |
| **Lakehouse** | Fabric item combining file storage (OneLake) with Spark and SQL query engines |
| **Medallion Architecture** | Data organisation pattern with Bronze, Silver, and Gold layers |
| **OneLake** | Fabric's unified data lake — single logical namespace per tenant, built on ADLS Gen2 |
| **Pipeline** | Data Factory orchestration workflow — sequences activities like copy, notebook, dataflow |
| **Shortcuts** | OneLake feature to reference external storage (ADLS, S3, GCS) without copying data |
| **Spark Pool** | Managed Apache Spark compute in Fabric — auto-provisioned, autoscaled |
| **SQL Analytics Endpoint** | Read-only T-SQL interface automatically generated for every Lakehouse |
| **V-Order** | Fabric-specific Parquet optimisation for fastest Power BI reads — applied automatically |
| **Warehouse** | Fabric item providing a full T-SQL data warehouse experience on Delta tables |
| **Z-Order** | Delta Lake file optimisation that co-locates related data for faster filtered queries |
