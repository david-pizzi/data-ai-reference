# Microsoft Fabric Overview — Deep Dive

> **Related pages:** [Fabric Overview (HTML)](../html/fabric-overview.html)
> **Official docs:** [Microsoft Fabric Documentation](https://learn.microsoft.com/en-us/fabric/get-started/microsoft-fabric-overview)

---

## What is Microsoft Fabric?

Microsoft Fabric is an **all-in-one SaaS analytics platform** that unifies data ingestion, transformation, storage, real-time processing, data science, and business intelligence into a single experience. It replaces the need to stitch together multiple Azure services (Synapse, ADF, ADLS, Power BI Premium, ADX) by providing all capabilities under one roof.

### Key Characteristics

| Characteristic | Detail |
|---|---|
| **Delivery model** | SaaS — no infrastructure to manage, no Azure subscription required |
| **Storage** | OneLake — one logical data lake per tenant, built on ADLS Gen2 |
| **Format** | Delta Lake (Parquet + transaction log) — open, readable by any engine |
| **Compute** | Capacity-based (CU) — shared pool across all workloads, scales up/down, can pause |
| **Governance** | Microsoft Purview built in — sensitivity labels, access control, auditing |
| **AI** | Copilot embedded across workloads for query authoring, code gen, data exploration |
| **Architecture** | Supports data mesh — each domain manages its own workspace and data products |

### The Pitch

> "Fabric is the analytics platform your customers don't have to build. Instead of wiring together 6+ Azure services, they get one SaaS experience with shared storage, shared governance, and shared compute — and it works with the data they already have via Shortcuts."

---

## Platform Architecture

Fabric follows a three-layer model:

### Layer 1: Workloads (Top)

Nine role-specific experiences, each with dedicated tools, APIs, and UX:

| # | Workload | Persona | What It Does |
|---|---|---|---|
| 1 | **Power BI** | Business Analyst | Interactive reports, dashboards, semantic models, DAX |
| 2 | **Data Factory** | Data Engineer | Data integration — 200+ connectors, Dataflows Gen2, pipelines |
| 3 | **Data Engineering** | Data Engineer | Apache Spark notebooks & jobs — PySpark, Scala, SparkSQL |
| 4 | **Data Warehouse** | Data Engineer / DBA | Full T-SQL warehouse with compute/storage separation |
| 5 | **Data Science** | Data Scientist | ML experiments, MLflow, model registry, Azure ML integration |
| 6 | **Real-Time Intelligence** | Data Engineer | Streaming ingestion, Eventhouse (KQL), real-time dashboards |
| 7 | **Databases** | DBA / Developer | Operational SQL DBs in Fabric, mirroring from external sources |
| 8 | **Industry Solutions** | Domain Specialist | Pre-built industry data models (healthcare, retail, etc.) |
| 9 | **IQ (Preview)** | All | Business ontology, labelled property graph, semantic AI layer |

### Layer 2: Platform Services (Middle)

Shared capabilities used by all workloads:

- **Copilot** — AI assistance for authoring queries, pipelines, code; respects tenant/data boundaries
- **Governance** — Microsoft Purview integration: sensitivity labels, DLP, auditing, compliance
- **OneLake Catalog** — central hub for discovering, exploring, and governing all Fabric items
- **Security** — RBAC, row-level security (RLS), object-level security, managed identities
- **Admin** — tenant settings, capacity management, usage monitoring

### Layer 3: OneLake (Foundation)

See dedicated OneLake section below.

---

## OneLake — The Unified Data Lake

OneLake is the storage foundation for all of Fabric. Think of it as **"OneDrive for analytics"**.

### Key Concepts

| Concept | Detail |
|---|---|
| **One per tenant** | Every Fabric tenant gets exactly one OneLake — automatic, no provisioning |
| **Built on ADLS Gen2** | Azure Data Lake Storage Gen2 underneath, but abstracted as SaaS |
| **Single namespace** | Hierarchical: Tenant → Workspaces → Items (Lakehouses, Warehouses, KQL DBs) |
| **Delta format** | All tables stored as Delta Lake (Parquet + transaction log) — open, versionable |
| **Shortcuts** | Virtual pointers to external storage (ADLS, S3, GCS, Dataverse) — no data copy |
| **Zero-copy access** | All workloads read/write the same physical data — no ETL between Fabric services |
| **No Azure account needed** | Fabric is SaaS; OneLake comes with the tenant |

### OneLake Hierarchy

```
Tenant (root)
├── Workspace A (Sales)
│   ├── Lakehouse: sales-raw
│   ├── Lakehouse: sales-curated
│   ├── Warehouse: sales-dw
│   └── Semantic Model: sales-reports
├── Workspace B (Engineering)
│   ├── Lakehouse: telemetry
│   └── KQL Database: real-time-logs
└── Workspace C (Data Science)
    ├── Lakehouse: ml-features
    └── ML Experiment: churn-prediction
```

### Shortcuts — Access External Data Without Copying

Shortcuts are OneLake's killer feature for hybrid/migration scenarios:

- Mount existing ADLS Gen2, AWS S3, or GCS storage into OneLake as if it were local
- Data stays in place — Fabric reads it directly with intelligent caching
- Enables cross-cloud analytics without egress costs
- Also supports Dataverse, Azure SQL (via mirroring), Cosmos DB, Snowflake, Databricks

---

## Lakehouse vs Warehouse — When to Use Which

| Aspect | Lakehouse | Warehouse |
|---|---|---|
| **Engine** | Apache Spark + SQL analytics endpoint (read-only) | Full T-SQL (read/write) |
| **Best for** | Data engineering, ELT, unstructured + structured data, data science | Traditional BI workloads, complex SQL queries, stored procedures |
| **Format** | Delta Lake in Files + Tables sections | Delta Lake (managed internally) |
| **Language** | PySpark, SparkSQL, Scala | T-SQL |
| **Schema** | Schema-on-read (flexible) | Schema-on-write (enforced) |
| **Transactions** | Delta Lake ACID | Full T-SQL ACID |
| **SQL endpoint** | Auto-generated, read-only | Full read/write |
| **When to choose** | Multi-language workloads, data science, raw data staging | SQL-heavy teams, DBA-managed, complex joins & aggregations |

**Common pattern:** Use Lakehouse for Bronze/Silver (ingestion, transformation) and Warehouse for Gold (curated, business-ready).

---

## End-to-End Data Flow

### Typical Pattern

```
Source Data → Data Factory (ingest) → Lakehouse / Warehouse (store)
           → Spark / T-SQL (transform: Bronze → Silver → Gold)
           → Semantic Model (model: DAX, relationships, RLS)
           → Power BI (report: dashboards, paginated reports)
```

### Parallel Paths

- **Real-Time:** Event sources → Eventstream → Eventhouse (KQL) → Real-time dashboard
- **Data Science:** Lakehouse → Spark notebook → ML experiment → Model registry → Predictions back to lakehouse
- **Mirroring:** Azure SQL / Cosmos DB / Snowflake → continuous replication into OneLake (no pipelines needed)

---

## Licensing & Capacity

### How Fabric Billing Works

| Concept | Detail |
|---|---|
| **Capacity Units (CU)** | Fabric's unit of compute — a shared pool used by all workloads |
| **F-SKUs** | F2, F4, F8, F16, F32, F64, F128, F256, F512, F1024, F2048 |
| **Pay-as-you-go** | Billed per second of CU consumption via Azure subscription |
| **Reservations** | 1-year or 3-year commitment for significant discount |
| **Pause/Resume** | Capacity can be paused when not in use — no charges while paused |
| **Auto-scale** | Can burst beyond base CU for short periods |

### Power BI Licensing in Fabric

| Licence | What It Unlocks |
|---|---|
| **Power BI Free** | View content in workspaces assigned to Fabric capacity |
| **Power BI Pro** | Share content, collaborate, publish to non-capacity workspaces |
| **Power BI PPU** | Premium per-user features (dataflows, paginated reports) without capacity |
| **Fabric Capacity (F-SKU)** | Unlocks all Fabric workloads + Power BI Premium features |

### Key Point: F-SKUs Replace P-SKUs

Fabric F-SKUs are the successor to Power BI Premium P-SKUs. An F64 is roughly equivalent to a P1. The difference: F-SKUs unlock **all** Fabric workloads, not just Power BI Premium features.

### Free Trial

- 60-day Fabric trial per user
- Includes all workloads and a trial capacity
- Great for POCs and customer demos

---

## Fabric IQ (Preview)

IQ is the newest Fabric workload — the **semantic AI layer** that sits on top of everything.

### What It Introduces

| Concept | Detail |
|---|---|
| **Ontology** | New Fabric item type: models business concepts, rules, and relationships |
| **Labelled property graph** | Connects to OneLake data + semantic models → live, structured business model |
| **Data agents** | AI agents that understand the ontology and can query/reason over data |
| **Operations agents** | AI agents that automate business processes using ontology context |
| **Reusable metrics** | Define once in ontology, consume everywhere — consistent across all reports/apps |

### Why It Matters

IQ is where Fabric is heading next. It moves beyond "store and query data" into "understand and reason about the business." This is the forward-looking story:

> "Today we help customers build lakehouses and dashboards. Tomorrow, Fabric IQ lets them build a living model of their business that AI agents can understand and act on."

---

## Fabric vs Legacy Azure Analytics Services

| Legacy Service | Fabric Replacement | Status |
|---|---|---|
| **Azure Synapse Analytics** | Fabric Data Engineering + Warehouse + Data Factory | Synapse is in maintenance; Fabric is the successor |
| **Azure Data Factory** | Fabric Data Factory | ADF v2 continues but no major new investment |
| **Azure Data Lake Storage** | OneLake | ADLS still works; use Shortcuts to bridge |
| **Azure Data Explorer** | Real-Time Intelligence | ADX clusters can link to Fabric |
| **Power BI Premium (P-SKUs)** | Fabric Capacity (F-SKUs) | P-SKUs being phased out in favour of F-SKUs |
| **Azure Analysis Services** | Power BI semantic models | AAS is in maintenance mode |
| **Azure Stream Analytics** | Real-Time Intelligence Eventstreams | ASA still available but Fabric is the modern path |

### Customer Conversation

When a customer is using legacy services:
1. **Don't panic them** — existing services still work and are supported
2. **Show the simplification** — "Instead of managing 6 services, you get one SaaS platform"
3. **Shortcuts are the bridge** — "You don't have to migrate data; OneLake can read your existing ADLS"
4. **Start small** — "Try one workload (e.g. Power BI on Fabric capacity) and expand from there"

---

## Microsoft Learn — Get Started with Fabric

10-module learning path for building foundational Fabric knowledge:

| # | Module | Key Topics |
|---|---|---|
| 1 | Introduction to end-to-end analytics | Fabric overview, workloads, Copilot, IQ |
| 2 | Get started with lakehouses | Create lakehouse, ingest data, SQL + Spark queries |
| 3 | Use Apache Spark | Spark clusters, notebooks, PySpark data processing |
| 4 | Work with Delta Lake tables | Delta format, time travel, optimisation, maintenance |
| 5 | Orchestrate with pipelines | Data Factory pipelines, activities, scheduling |
| 6 | Ingest with Dataflows Gen2 | Power Query Online, visual ETL, 200+ connectors |
| 7 | Get started with data warehouses | T-SQL warehouse, compute separation, Delta native |
| 8 | Real-Time Intelligence | Streaming, KQL, Eventhouse, real-time dashboards |
| 9 | Data Science in Fabric | ML experiments, MLflow, model registry |
| 10 | Administer Fabric | Tenant settings, capacity, governance, Purview |

**Link:** [Get Started with Microsoft Fabric (Learning Path)](https://learn.microsoft.com/en-us/training/paths/get-started-fabric/)

---

## Key Terms Glossary

| Term | Definition |
|---|---|
| **OneLake** | Fabric's unified logical data lake — one per tenant, built on ADLS Gen2 |
| **Lakehouse** | Combines data lake flexibility with warehouse analytics — files + tables + SQL endpoint |
| **Warehouse** | Full T-SQL transactional data warehouse in Fabric |
| **Eventhouse** | KQL-based real-time analytics database in Fabric |
| **Semantic Model** | Power BI's data model layer — DAX measures, relationships, RLS |
| **Delta Lake** | Open-source storage format: Parquet files + JSON transaction log = ACID on a data lake |
| **Shortcut** | OneLake virtual pointer to external storage — no data copy, intelligent caching |
| **Mirroring** | Continuous replication from external databases into OneLake |
| **Capacity Unit (CU)** | Fabric's unit of compute billing — shared across all workloads |
| **F-SKU** | Fabric capacity SKU (F2–F2048) — replaces Power BI Premium P-SKUs |
| **Dataflows Gen2** | Power Query Online for Fabric — visual, no-code data transformation |
| **Ontology (IQ)** | Business concept model — the semantic layer IQ introduces |
| **OneLake Catalog** | Central hub for discovering and governing all Fabric items |

---

*Data & AI Reference — Microsoft Fabric Overview*
