# Data Lifecycle in Microsoft Fabric

> **Source:** [End-to-End Tutorials](https://learn.microsoft.com/en-us/fabric/fundamentals/end-to-end-tutorials)
> **Related pages:** [Fundamentals Overview](../fundamentals.md) | [Core Concepts](concepts.md)

---

## Learning Objectives

- Describe the six stages of the data lifecycle in Microsoft Fabric.
- Identify the ingestion methods available and when to use each.
- Understand storage options within OneLake.
- Compare transformation tools (Notebooks, Dataflows Gen2, Pipelines).
- Explain how analytics and visualization connect to stored data.
- Describe external integration patterns for sharing data outside Fabric.

---

## Key Concepts

### The Six-Stage Data Lifecycle

Microsoft Fabric organizes the analytics workflow into six stages, each supported by dedicated tools and services.

```
Get Data → Store Data → Prepare & Transform → Analyze & Train → Track & Visualize → External Integration
```

### Stage 1: Get Data

Ingestion is the process of bringing data into OneLake from internal and external sources.

| Method | Description | Best For |
|--------|-------------|----------|
| **Eventstreams** | Capture, transform, and route real-time events from sources like Azure Event Hubs, Kafka, IoT Hub, custom apps | Streaming / real-time data |
| **Data Pipelines** | Orchestrate data movement with Copy activity, 200+ connectors, scheduling, and monitoring | Batch ingestion from databases, files, APIs |
| **Mirroring** | Near-real-time replication of operational databases (Azure SQL, Cosmos DB, Snowflake, PostgreSQL) into OneLake | Keeping an analytics copy of operational data in sync |
| **Shortcuts** | Mount external storage (ADLS Gen2, S3, GCS, Dataverse) without copying data | Federated access to data that stays in place |
| **Copy Job** | A simplified, scalable data copy experience for moving data into OneLake | Large-scale bulk data movement without complex pipeline design |
| **Dataflows Gen2** | Power Query-based ingestion with built-in transformation | Self-service ingestion by analysts and citizen integrators |
| **Upload** | Drag-and-drop files directly into a lakehouse | Ad-hoc, small-scale data loading |

### Stage 2: Store Data

All data in Fabric is stored in **OneLake** using open formats.

| Storage Item | Format | Use Case |
|-------------|--------|----------|
| **Lakehouse Tables** | Delta Lake (Parquet + transaction log) | Structured, queryable analytics data |
| **Lakehouse Files** | Any format (CSV, JSON, Parquet, images) | Raw, semi-structured, or unstructured data |
| **Warehouse Tables** | Delta Lake | Structured data managed via T-SQL |
| **Eventhouse (KQL Database)** | Columnar store optimized for time-series | Real-time and time-series analytics |
| **SQL Database** | SQL Server engine | Operational / transactional workloads in Fabric |

Key storage principles:

- **One copy of data** — all workloads access the same underlying OneLake storage
- **Open format** — Delta Lake is the standard; data is portable and readable by any compatible engine
- **Shortcuts** — extend OneLake virtually without data duplication

### Stage 3: Prepare & Transform

Raw data typically requires cleaning, enrichment, and structuring before it is useful for analytics.

| Tool | Engine | Best For |
|------|--------|----------|
| **Spark Notebooks** | Apache Spark (PySpark, Scala, SparkSQL, R) | Complex transformations, large-scale processing, ML feature engineering |
| **Dataflows Gen2** | Power Query (M language) | Low-code transformations, familiar to Power BI / Excel users |
| **Data Pipelines** | Orchestration engine | Chaining activities, scheduling, conditional logic, error handling |
| **T-SQL (Warehouse)** | Distributed SQL engine | SQL-native transformations, stored procedures, views |
| **KQL (Eventhouse)** | Kusto engine | Real-time data transformation and enrichment |

Common transformation patterns:

- **Medallion architecture** — Bronze (raw) → Silver (cleaned/conformed) → Gold (business-ready) layers in a lakehouse
- **Star schema** — Fact and dimension tables in a warehouse for BI consumption
- **Streaming enrichment** — Eventstream processors that clean and route data in real time

### Stage 4: Analyze & Train

Once data is prepared, Fabric provides multiple engines for analysis and machine learning.

| Tool | What It Does |
|------|-------------|
| **SQL analytics endpoint** | Read-only T-SQL queries against lakehouse Delta tables |
| **Warehouse T-SQL** | Full read/write T-SQL analytics with cross-database queries |
| **Spark Notebooks** | Exploratory data analysis, statistical analysis, ML model training |
| **ML Experiments (MLflow)** | Track model runs, compare metrics, log parameters and artifacts |
| **Model Registry** | Version, manage, and deploy trained ML models |
| **KQL Queryset** | Ad-hoc real-time analytics using Kusto Query Language |

### Stage 5: Track & Visualize

This stage covers monitoring data quality, presenting insights, and enabling self-service analytics.

| Tool | What It Does |
|------|-------------|
| **Power BI Reports** | Interactive dashboards and visualizations connected to semantic models |
| **Power BI Semantic Models** | Business-logic layer (measures, relationships) on top of data |
| **Direct Lake** | Power BI reads Delta files directly — no import, no DirectQuery overhead |
| **Paginated Reports** | Pixel-perfect, printable reports for operational and regulatory use |
| **Real-Time Dashboards** | Live dashboards connected to Eventhouse KQL databases |
| **Data Activator (Reflex)** | Monitor data conditions and trigger alerts or Power Automate flows |

### Stage 6: External Integration

Fabric data can be shared and consumed by systems outside the platform.

| Method | Description |
|--------|-------------|
| **OneLake APIs** | Programmatic access to OneLake storage via ADLS Gen2-compatible APIs |
| **SQL connection strings** | Connect external tools (SSMS, Azure Data Studio, dbt) to Warehouse or SQL analytics endpoint |
| **Publish to web** | Share Power BI reports publicly (with appropriate governance) |
| **Embed reports** | Embed Power BI visuals in custom applications via Power BI Embedded |
| **XMLA endpoint** | Connect third-party tools to semantic models using the XMLA protocol |
| **Git integration** | Version-control Fabric items via Azure DevOps or GitHub |
| **REST APIs** | Automate Fabric operations programmatically |

---

## Key Terms

| Term | Definition |
|------|-----------|
| **Ingestion** | The process of bringing data into OneLake from external or internal sources |
| **Medallion architecture** | A data organization pattern with Bronze (raw), Silver (cleaned), and Gold (business-ready) layers |
| **Mirroring** | Near-real-time replication of an external database into OneLake as Delta tables |
| **Eventstream** | A Fabric item that captures, transforms, and routes real-time event data |
| **Copy Job** | A simplified data movement experience for bulk loading data into OneLake |
| **Data Activator** | A no-code tool that monitors data conditions and triggers actions (alerts, Power Automate flows) |
| **XMLA endpoint** | An industry-standard protocol for connecting external tools to Power BI semantic models |

---

## Further Reading

- [End-to-End Tutorials in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/fundamentals/end-to-end-tutorials)
- [Data Ingestion Options](https://learn.microsoft.com/en-us/fabric/data-engineering/load-data-lakehouse)
- [Mirroring in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/database/mirrored-database/overview)
- [Eventstreams Overview](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/event-streams/overview)
- [Copy Job Overview](https://learn.microsoft.com/en-us/fabric/data-factory/copy-job-overview)
