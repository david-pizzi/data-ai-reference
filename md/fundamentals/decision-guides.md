# Decision Guides — Choosing the Right Fabric Components

> **Source:** [Microsoft Fabric Decision Guide](https://learn.microsoft.com/en-us/fabric/fundamentals/decision-guide-data-store)
> **Related pages:** [Fundamentals Overview](../fundamentals.md) | [Core Concepts](concepts.md)

---

## Learning Objectives

- Choose the right data store for a given scenario (Eventhouse, Cosmos DB, SQL Database, Warehouse, Lakehouse).
- Map personas and skillsets to appropriate Fabric workloads.
- Apply decision criteria when choosing between Lakehouse and Warehouse.
- Compare Copy activity, Dataflow Gen2, Eventstream, and Spark for data movement and transformation.

---

## Key Concepts

### Choosing a Data Store

Fabric offers multiple data stores, each optimized for different workloads. Choosing the wrong one leads to poor performance, unnecessary complexity, or data duplication.

| Data Store | Engine | Best For | Query Language |
|-----------|--------|----------|---------------|
| **Lakehouse** | Apache Spark + SQL analytics endpoint | Data engineering, data science, large-scale processing, mixed file formats, medallion architecture | PySpark, SparkSQL, T-SQL (read-only via SQL endpoint) |
| **Warehouse** | Distributed SQL engine | Traditional BI, reporting, SQL-native teams, full DML operations | T-SQL (read/write) |
| **Eventhouse** | Kusto engine | Real-time analytics, time-series data, log analytics, IoT telemetry | KQL (Kusto Query Language) |
| **SQL Database** | SQL Server engine | Operational/transactional workloads, application backends, OLTP | T-SQL |
| **Cosmos DB (Mirrored)** | Cosmos DB (mirrored into OneLake) | Globally distributed operational data that needs analytics in Fabric | SQL API (source), T-SQL/Spark (mirrored copy) |

#### Decision Flow

1. **Is the data streaming or time-series?** → **Eventhouse**
2. **Is it an operational/transactional workload?** → **SQL Database** (or mirror from Cosmos DB)
3. **Do you need full T-SQL DML (INSERT/UPDATE/DELETE)?** → **Warehouse**
4. **Do you need Spark, mixed file formats, or ML?** → **Lakehouse**
5. **Do you need a SQL-only experience with simpler governance?** → **Warehouse**

### Personas and Skillsets

Different Fabric workloads target different personas. Understanding this mapping helps organizations assign the right tools to the right teams.

| Persona | Primary Skillset | Recommended Workloads |
|---------|-----------------|----------------------|
| **Data Engineer** | Python, Spark, SQL, ETL design | Data Engineering (notebooks, Spark jobs), Data Factory (pipelines) |
| **Data Analyst** | SQL, DAX, data visualization | Data Warehouse, Power BI |
| **Data Scientist** | Python, ML frameworks, statistics | Data Science (experiments, models), Data Engineering (notebooks) |
| **BI Developer** | DAX, Power Query, data modeling | Power BI, Data Warehouse |
| **SQL Developer / DBA** | T-SQL, database administration | Data Warehouse, SQL Database, Databases (mirroring) |
| **Citizen Integrator** | Low-code, Power Query | Data Factory (Dataflows Gen2, pipelines) |
| **Streaming Analyst** | KQL, event processing | Real-Time Intelligence (Eventstreams, Eventhouse) |
| **Business User** | Report consumption, natural language | Power BI (reports, apps), Copilot, Fabric IQ |

### Lakehouse vs Warehouse

This is the most common decision point for teams building analytics in Fabric.

| Criterion | Lakehouse | Warehouse |
|-----------|-----------|-----------|
| **Primary compute** | Apache Spark (managed clusters) | Distributed SQL engine |
| **Query language** | PySpark, Scala, SparkSQL, R + T-SQL (read-only endpoint) | Full T-SQL (read/write) |
| **Schema approach** | Schema-on-read (flexible) | Schema-on-write (enforced) |
| **Storage structure** | Tables/ (Delta) + Files/ (any format) | Delta tables only |
| **DML support** | Via Spark; SQL endpoint is read-only | Full INSERT, UPDATE, DELETE, MERGE via T-SQL |
| **Stored procedures** | Not supported | Supported |
| **File format flexibility** | Supports CSV, JSON, Parquet, images, etc. in Files/ | Delta tables only |
| **Best for** | Data engineering, data science, large-scale processing, ML | Traditional BI, reporting, SQL-native teams |
| **Common pattern** | Medallion architecture (bronze/silver/gold) | Gold layer serving, standalone structured analytics |
| **Copilot support** | Code generation in notebooks | T-SQL generation in query editor |

**Key insight:** Both store data as Delta Lake in OneLake. The difference is the **compute engine** and **developer experience**, not the storage format. Many organizations use both — Lakehouse for engineering/science, Warehouse for serving BI.

### Data Movement: Copy Activity vs Dataflow Gen2 vs Eventstream vs Spark

| Capability | Copy Activity | Dataflow Gen2 | Eventstream | Spark (Notebook/Job) |
|-----------|--------------|---------------|-------------|---------------------|
| **Interface** | Pipeline visual designer | Power Query (low-code) | Streaming canvas | Code (PySpark, Scala, SQL) |
| **Execution model** | Batch | Batch | Real-time streaming | Batch or streaming |
| **Transformation** | Minimal (column mapping, type conversion) | Rich (Power Query M transformations) | In-stream processors | Full programmatic control |
| **Connectors** | 200+ source/sink connectors | 200+ connectors via Power Query | Event Hubs, Kafka, IoT Hub, custom apps | Any source accessible via Spark |
| **Best for** | Simple data movement at scale | Self-service ETL by analysts | Continuous streaming ingestion | Complex transformations, large-scale processing |
| **Scheduling** | Pipeline triggers and schedules | Refresh schedules | Always-on (continuous) | Pipeline triggers, schedules, or manual |
| **Skillset** | Low-code (pipeline designer) | Low-code (Power Query) | Low-code (streaming designer) | Code (Python, Scala, SQL) |
| **Typical user** | Data integration engineer | Citizen integrator, analyst | Streaming engineer | Data engineer, data scientist |

#### Decision Flow for Data Movement

1. **Is the data streaming/real-time?** → **Eventstream**
2. **Do you need complex transformations and full programmatic control?** → **Spark**
3. **Do you need self-service, low-code transformations?** → **Dataflow Gen2**
4. **Do you need simple, high-volume data copy with minimal transformation?** → **Copy Activity**

### Common Scenarios

| Scenario | Recommended Approach |
|----------|---------------------|
| Ingest daily sales data from Azure SQL into a lakehouse | **Copy Activity** in a pipeline with a daily schedule |
| Clean and reshape Excel/CSV files uploaded by business users | **Dataflow Gen2** with Power Query transformations |
| Process IoT sensor data in real time for dashboards | **Eventstream** → Eventhouse → Real-Time Dashboard |
| Build a medallion architecture with complex business logic | **Spark notebooks** orchestrated by a pipeline |
| Create a BI-ready star schema for Power BI reports | **Warehouse** with T-SQL stored procedures |
| Train ML models on historical data | **Lakehouse** + Data Science notebooks |
| Replicate an Azure SQL database for analytics | **Mirroring** (Databases workload) |
| Enable natural language querying over business data | **Lakehouse/Warehouse** + semantic model + Copilot/Fabric IQ |

---

## Key Terms

| Term | Definition |
|------|-----------|
| **Copy Activity** | A pipeline activity that moves data from a source to a destination with minimal transformation |
| **Dataflow Gen2** | A low-code, Power Query-based tool for ingesting and transforming data |
| **Eventstream** | A streaming ingestion and processing item for real-time data |
| **Medallion architecture** | A data organization pattern (Bronze → Silver → Gold) commonly used in lakehouses |
| **Star schema** | A data modeling pattern with fact tables and dimension tables, optimized for BI queries |
| **Mirroring** | Near-real-time replication of operational databases into OneLake |

---

## Further Reading

- [Choose a Data Store in Fabric](https://learn.microsoft.com/en-us/fabric/fundamentals/decision-guide-data-store)
- [Lakehouse vs Warehouse](https://learn.microsoft.com/en-us/fabric/data-warehouse/data-warehousing#compare-fabric-data-stores)
- [Data Movement Options](https://learn.microsoft.com/en-us/fabric/fundamentals/decision-guide-pipeline-dataflow-spark)
- [Personas and Workloads](https://learn.microsoft.com/en-us/fabric/fundamentals/decision-guide-data-store#personas)
