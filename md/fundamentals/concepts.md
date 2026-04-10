# Core Concepts — Microsoft Fabric Fundamentals

> **Source:** [Microsoft Fabric Overview](https://learn.microsoft.com/en-us/fabric/fundamentals/microsoft-fabric-overview)
> **Related pages:** [Fundamentals Overview](../fundamentals.md) | [Big Picture](../big-picture.md)

---

## Learning Objectives

- Describe what Microsoft Fabric is and how it differs from legacy Azure analytics services.
- Explain the SaaS architecture and its benefits over PaaS.
- Identify all Fabric workloads and the personas they serve.
- Understand OneLake as the unified storage foundation.
- Describe the Real-Time Hub and its role in streaming analytics.
- Define key Fabric terminology used across the platform.

---

## Key Concepts

### What Is Microsoft Fabric?

Microsoft Fabric is a **unified SaaS analytics platform** that brings together data ingestion, engineering, warehousing, data science, real-time analytics, business intelligence, and AI under a single experience. It replaces the fragmented Azure analytics stack (Azure Data Factory, Azure Synapse, Azure Data Lake Storage, Power BI Service, Azure Data Explorer) with a single integrated platform.

Key characteristics:

- **Single tenant** — one Fabric tenant per Microsoft Entra ID (Azure AD) tenant
- **Single billing model** — capacity-based (Capacity Units / CUs), shared across all workloads
- **Single security model** — workspace roles, item permissions, sensitivity labels
- **Single storage layer** — OneLake, built on ADLS Gen2, with all data stored in Delta Lake format
- **No infrastructure management** — Microsoft manages compute, networking, patching, and availability

### SaaS Architecture

Fabric is **SaaS, not PaaS**. This distinction matters:

| Aspect | PaaS (Legacy Azure) | SaaS (Fabric) |
|--------|---------------------|----------------|
| **Provisioning** | You create and configure each service | You get a tenant and start working |
| **Compute management** | You size and manage Spark pools, SQL pools | Compute scales automatically within your capacity SKU |
| **Networking** | You configure VNets, private endpoints | Microsoft manages networking |
| **Billing** | Per-service, per-resource | Single capacity-based model across all workloads |
| **Upgrades** | You manage versions and patching | Automatic, managed by Microsoft |
| **Azure subscription** | Required | Not required (can pay via Azure or directly) |

### Workloads

Fabric provides multiple workloads, each optimized for a specific persona and use case:

| Workload | Persona | What It Does |
|----------|---------|-------------|
| **Power BI** | Business analysts, report authors | Interactive dashboards, reports, semantic models, paginated reports |
| **Data Factory** | Data integration engineers | 200+ connectors, Dataflows Gen2 (Power Query), orchestration pipelines, Copy Jobs |
| **Data Engineering** | Data engineers | Apache Spark notebooks and jobs (PySpark, Scala, SparkSQL), lakehouse management |
| **Data Science** | Data scientists, ML engineers | ML experiments, MLflow tracking, model registry, prediction scoring |
| **Data Warehouse** | SQL analysts, BI developers | Full T-SQL read/write warehouse, stored procedures, cross-database queries |
| **Databases** | Application developers, DBAs | Operational SQL databases in Fabric, mirroring from Azure SQL/Cosmos DB/Snowflake |
| **Real-Time Intelligence** | Streaming analysts, IoT engineers | Eventstreams, Eventhouse (KQL), real-time dashboards, Real-Time Hub |
| **Fabric IQ** | Knowledge workers, AI builders | Ontology layer, labelled property graph, data agents, operations agents |
| **Industry Solutions** | Domain specialists | Pre-built data models and solutions for specific industries |

### OneLake

OneLake is Fabric's **unified storage layer** — a single logical data lake per tenant.

- Built on Azure Data Lake Storage Gen2, abstracted as SaaS
- Every table stored in **Delta Lake format** (Parquet files + transaction log)
- All workloads read and write the same data — no ETL between Fabric services
- Organized hierarchically: Tenant > Workspace > Item (Lakehouse, Warehouse, etc.)
- Supports **shortcuts** to mount external storage (ADLS Gen2, Amazon S3, Google Cloud Storage, Dataverse) without copying data
- Think of it as "OneDrive for analytics data"

### Real-Time Hub

The Real-Time Hub is a **centralized catalog for streaming data sources** within Fabric.

- Discover and manage all streaming data sources across the tenant
- Browse data streams from Azure Event Hubs, Azure IoT Hub, Kafka, custom apps, and more
- Connect streams to Eventstreams for processing, Eventhouse for analytics, or alerts for notifications
- Provides a single pane of glass for all real-time data flowing into the organization
- Enables self-service access to streaming data — consumers can find and subscribe to streams without involving the data engineering team

---

## Key Terms

| Term | Definition |
|------|-----------|
| **Capacity** | A pool of compute resources (measured in Capacity Units / CUs) shared across all Fabric workloads in assigned workspaces |
| **Workspace** | A logical container for Fabric items (lakehouses, warehouses, reports, notebooks, etc.); maps to a folder in OneLake |
| **Item** | Any object within a workspace — lakehouse, warehouse, notebook, pipeline, report, semantic model, etc. |
| **OneLake** | The unified storage layer for all Fabric data; one logical data lake per tenant |
| **Delta Lake** | Open-source storage format (Parquet + transaction log) used for all tables in OneLake |
| **Shortcut** | A pointer to external data (ADLS, S3, GCS) that appears as a local folder in a lakehouse without copying data |
| **Lakehouse** | A Fabric item combining flexible file storage with SQL-queryable Delta tables |
| **Warehouse** | A Fabric item providing a full T-SQL read/write data warehouse experience |
| **Eventhouse** | A Fabric item for real-time analytics using KQL (Kusto Query Language) |
| **Semantic model** | A business-logic layer (measures, relationships, hierarchies) on top of data, used by Power BI for reporting |
| **SQL analytics endpoint** | An auto-generated read-only T-SQL interface to query Delta tables in a lakehouse |
| **Capacity Unit (CU)** | The unit of compute measurement in Fabric; different SKUs provide different CU allocations |
| **Tenant** | The top-level organizational boundary in Fabric, tied to a Microsoft Entra ID tenant |
| **Real-Time Hub** | A centralized catalog for discovering and managing streaming data sources |
| **Dataflow Gen2** | A Power Query-based data transformation tool in Fabric (replaces Dataflows Gen1) |

---

## Further Reading

- [Microsoft Fabric Overview](https://learn.microsoft.com/en-us/fabric/fundamentals/microsoft-fabric-overview)
- [Microsoft Fabric Terminology](https://learn.microsoft.com/en-us/fabric/get-started/fabric-terminology)
- [OneLake Overview](https://learn.microsoft.com/en-us/fabric/onelake/onelake-overview)
- [Real-Time Hub Overview](https://learn.microsoft.com/en-us/fabric/real-time-hub/real-time-hub-overview)
- [Microsoft Fabric Workloads](https://learn.microsoft.com/en-us/fabric/get-started/microsoft-fabric-overview#components-of-microsoft-fabric)
