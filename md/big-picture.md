# Phase 1: The Big Picture — Deep Dive

> **Related pages:** [Phase 1 Study Guide (HTML)](../html/big-picture.html) | [Fabric Overview](../html/fabric-overview.html)
> **Official docs:** [Microsoft Fabric Documentation](https://learn.microsoft.com/en-us/fabric/get-started/microsoft-fabric-overview)

---

## What Is Microsoft Fabric?

Microsoft Fabric is a **unified SaaS analytics platform** that brings together data ingestion, transformation, storage, real-time processing, data science, and business intelligence under a single experience.

### The Problem It Solves

Before Fabric, Microsoft's analytics stack was fragmented across multiple services, each with its own portal, billing model, security boundary, and learning curve:

| Legacy Service | Role | Pain Point |
|---|---|---|
| **Azure Data Factory** | Data ingestion & orchestration | Separate portal, separate billing, separate RBAC |
| **Azure Synapse Analytics** | Data warehousing & Spark processing | Complex provisioning, multiple pool types, steep learning curve |
| **Azure Data Lake Storage (Gen2)** | Raw data storage | Just storage — no compute, no governance built in |
| **Power BI** | Reporting & visualisation | Disconnected from engineering workflows, separate capacity model |
| **Azure Data Explorer** | Real-time analytics (KQL) | Niche service, not integrated with warehouse or lake |
| **Azure Analysis Services** | Semantic models / tabular models | In maintenance mode, replaced by Power BI semantic models |

Customers had to stitch these together manually — building ETL pipelines between them, managing multiple security models, and dealing with data duplication across services.

**Fabric's answer:** unify everything into a single SaaS experience built on **OneLake** — one storage layer, one security model, one billing model, one governance framework.

---

## Platform Architecture

Fabric is built on a **three-layer model**:

### Layer 1: OneLake (Foundation)

The unified storage layer — a single logical data lake per tenant.

- Built on ADLS Gen2 under the hood, but abstracted away as SaaS
- Every table is stored in **Delta Lake format** (Parquet + transaction log)
- All workloads read/write the same data — no ETL between Fabric services
- Think of it as "OneDrive for analytics"

### Layer 2: Platform Services (Middle)

Shared capabilities used by all workloads:

- **Copilot** — AI assistance for query authoring, pipeline building, code generation
- **Governance** — Microsoft Purview integration for sensitivity labels, access control, auditing
- **OneLake Catalog** — discover, explore, and govern all Fabric items across workspaces
- **Security** — workspace roles, item permissions, row-level security
- **Admin** — tenant settings, capacity management, usage monitoring

### Layer 3: Workloads (Top)

Nine role-specific analytics experiences, each optimised for a persona:

1. **Power BI** — interactive dashboards, reports, semantic models
2. **Data Factory** — data integration, 200+ connectors, Dataflows Gen2, orchestration pipelines
3. **Data Engineering** — Apache Spark notebooks & jobs (PySpark, Scala, SparkSQL)
4. **Data Warehouse** — full T-SQL transactional warehouse, compute/storage separation
5. **Data Science** — ML experiments, MLflow, model registry
6. **Real-Time Intelligence** — streaming ingestion, Eventhouse (KQL), real-time dashboards
7. **Databases** — operational SQL databases, mirroring from Azure SQL/Cosmos DB/Snowflake/Databricks
8. **Industry Solutions** — pre-built data models for specific industries
9. **IQ (Preview)** — ontology layer, labelled property graph, AI-ready semantic model

---

## OneLake Concepts

### Hierarchy

```
Tenant
  └── Workspace (= folder in OneLake)
        ├── Lakehouse
        │     ├── Tables/ (Delta format, queryable via SQL endpoint)
        │     └── Files/ (unstructured data, raw files)
        ├── Warehouse
        ├── KQL Database
        ├── Notebook
        ├── Pipeline
        └── Report
```

### Delta Lake Format

Every table in OneLake is stored as **Delta Lake**:

- **Parquet files** for columnar storage (efficient reads, compression)
- **Transaction log** (_delta_log/) for ACID transactions, time travel, schema evolution
- **Open format** — readable by Spark, T-SQL, Python, Power BI, or any engine that supports Delta
- No vendor lock-in on the data layer

### Shortcuts

Shortcuts allow you to **mount external storage** without copying data:

- Supported sources: ADLS Gen2, Amazon S3, Google Cloud Storage, Dataverse
- Data stays in place — Fabric reads it on demand
- Cached for performance on subsequent reads
- Enables a "virtual data lake" that spans cloud providers

### Zero-Copy Access

All Fabric workloads access the same data in OneLake directly:

- A Spark notebook writes a Delta table → Power BI reads it immediately
- A T-SQL warehouse query reads the same table → no ETL needed
- Data Science models read from the same lakehouse → predictions written back
- This eliminates the "data copy" problem that plagued the legacy stack

---

## Lakehouse vs Warehouse

| Dimension | Lakehouse | Warehouse |
|---|---|---|
| **Query language** | Spark (PySpark/Scala/SparkSQL) + SQL endpoint (read-only T-SQL) | Full T-SQL (read/write) |
| **Best for** | Data engineering, data science, large-scale processing | Traditional BI, reporting, analysts who know SQL |
| **Schema** | Schema-on-read (flexible), Delta tables + raw files | Schema-on-write (enforced), relational tables only |
| **Storage** | Tables/ folder (Delta) + Files/ folder (any format) | Delta tables only |
| **Compute** | Apache Spark clusters (managed) | Distributed SQL engine |
| **Insert/Update/Delete** | Via Spark or through SQL endpoint (limited) | Full DML support via T-SQL |
| **When to choose** | You need Spark, have mixed file formats, or do ML | You want pure SQL, need stored procedures, or serve BI workloads |
| **Common pattern** | Medallion architecture (bronze/silver/gold) | Gold layer serving, or standalone for structured data |

**Key insight:** Both store data as Delta Lake in OneLake. The difference is the compute engine and developer experience, not the storage format. Many customers use both — Lakehouse for engineering, Warehouse for serving.

---

## MS Learn Modules — Phase 1

### Module 1: Introduction to End-to-End Analytics Using Microsoft Fabric (~35 min)

**What you'll learn:**
- What Fabric is and how it fits into the Microsoft analytics ecosystem
- The 9 workloads and how they connect
- OneLake as the unified storage layer
- Copilot and IQ capabilities
- How Fabric replaces legacy services

**Link:** [Start module](https://learn.microsoft.com/en-us/training/modules/introduction-end-analytics-use-microsoft-fabric/)

### Module 2: Get Started with Lakehouses in Microsoft Fabric (~45 min)

**What you'll learn:**
- Create a lakehouse and understand its structure (Tables/ and Files/)
- Ingest data using upload, Spark, or pipelines
- Query data via the SQL endpoint (auto-generated read-only T-SQL)
- Understand how Delta tables are created and managed
- Connect Power BI to the lakehouse's default semantic model

**Link:** [Start module](https://learn.microsoft.com/en-us/training/modules/get-started-lakehouses/)

### Module 3: Get Started with Data Warehouses in Microsoft Fabric (~30 min)

**What you'll learn:**
- Create a Fabric warehouse and understand compute/storage separation
- Write T-SQL to create tables, insert data, and query
- Understand how the warehouse stores data as Delta Lake natively
- Compare the warehouse experience to the lakehouse SQL endpoint
- Cross-database queries between warehouse and lakehouse

**Link:** [Start module](https://learn.microsoft.com/en-us/training/modules/get-started-data-warehouse/)

### Module 4: Administer a Microsoft Fabric Environment (~30 min)

**What you'll learn:**
- Tenant-level admin settings and how they affect workspaces
- Capacity management (CUs, SKUs, scaling, pausing)
- Workspace roles and permissions model
- Governance with Microsoft Purview (sensitivity labels, auditing)
- Monitoring usage and performance

**Link:** [Start module](https://learn.microsoft.com/en-us/training/modules/administer-fabric/)

---

## Exploration Checklist

Use the [Fabric trial](https://app.fabric.microsoft.com) to work through these hands-on tasks:

- [ ] **Activate trial** — go to app.fabric.microsoft.com, start a 60-day free trial
- [ ] **Create a workspace** — name it "Phase1-Sandbox", assign trial capacity
- [ ] **Lakehouse** — create one, upload a CSV, see it appear as a Delta table, query via SQL endpoint
- [ ] **Warehouse** — create one, run a T-SQL `CREATE TABLE`, compare the experience to Lakehouse
- [ ] **Notebook** — open a Spark notebook, connect to your lakehouse, run a simple PySpark query
- [ ] **Pipeline** — create a Data Factory pipeline, add a Copy activity, see the orchestration UI
- [ ] **Report** — create a Power BI report from the lakehouse SQL endpoint's default semantic model
- [ ] **Eventhouse** — create one, see the KQL database experience (Real-Time Intelligence)
- [ ] **Ontology** — if available in preview, explore the IQ ontology item type

---

## The 4 Principles — Expanded

### Principle 1: OneLake = One Copy of Data

This is the single biggest architectural shift from the legacy stack. In the old world, you'd copy data from ADLS into Synapse, then copy transformed data into Power BI's import model, then copy subsets into ADX for real-time queries. Each copy introduced latency, inconsistency, and cost.

In Fabric, all workloads read and write from the same OneLake storage. A Spark notebook processes data and writes a Delta table. Power BI reads that same table directly. The warehouse queries it via T-SQL. No copies, no ETL between services.

Shortcuts extend this principle to external data — mount ADLS, S3, or GCS without copying, and Fabric treats it as if it's in OneLake.

### Principle 2: SaaS, Not PaaS

The legacy Azure analytics services were PaaS — you provisioned Synapse workspaces, managed Spark pool sizes, configured ADLS storage accounts, set up networking, and dealt with Azure RBAC. Each service required infrastructure expertise.

Fabric is SaaS. There's nothing to provision. You get a tenant, create workspaces, and start working. Compute scales automatically based on your capacity SKU. Microsoft manages the infrastructure, patching, and availability. You don't even need an Azure subscription to use Fabric (though you can pay via Azure).

This matters for customers because it dramatically lowers the barrier to entry and reduces operational overhead.

### Principle 3: Delta Lake Is the Universal Format

Every table in OneLake is stored as Delta Lake — an open-source format built on Apache Parquet with a transaction log. This means:

- **ACID transactions** — concurrent reads and writes are safe
- **Time travel** — query data as it existed at any point in time
- **Schema evolution** — add columns without rewriting data
- **Open format** — any tool that reads Parquet can read Delta (Spark, DuckDB, Polars, etc.)
- **No vendor lock-in** — your data is portable

This is a deliberate strategic choice by Microsoft. By using an open format, they compete on the value of the platform (workloads, governance, Copilot), not on data lock-in. It's also why Fabric can interoperate with Databricks (which co-created Delta Lake).

### Principle 4: Fabric IQ Is the Next Frontier

IQ is where Fabric is heading — from a "data platform" to an "intelligence platform." The key innovation is the **ontology**: a labelled property graph that models business concepts, rules, and relationships on top of OneLake data.

Why this matters:
- Current analytics tools understand data (tables, columns, relationships)
- IQ understands the business (customers, products, processes, rules)
- AI agents built on IQ can reason about business concepts, not just query tables
- This enables "data agents" (answer questions) and "operations agents" (take actions)

This is the strategic narrative: Fabric isn't just about unifying data services — it's about creating a platform where AI can understand and act on business knowledge.

---

## Links to Official Docs

- [Microsoft Fabric Overview](https://learn.microsoft.com/en-us/fabric/get-started/microsoft-fabric-overview)
- [OneLake Overview](https://learn.microsoft.com/en-us/fabric/onelake/onelake-overview)
- [Fabric Architecture](https://learn.microsoft.com/en-us/fabric/fundamentals/microsoft-fabric-overview#microsoft-fabric-architecture)
- [What's New in Fabric](https://learn.microsoft.com/en-us/fabric/get-started/whats-new)
- [Fabric IQ Overview](https://learn.microsoft.com/en-us/fabric/iq/overview)
- [Fabric Trial Setup](https://learn.microsoft.com/en-us/fabric/get-started/fabric-trial)
- [Get Started with Fabric Learning Path](https://learn.microsoft.com/en-us/training/paths/get-started-fabric/)
- [Fabric Blog: From Data Platform to Intelligence Platform](https://blog.fabric.microsoft.com/en-us/blog/from-data-platform-to-intelligence-platform/)
