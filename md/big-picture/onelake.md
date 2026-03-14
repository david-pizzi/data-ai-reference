# OneLake & Lakehouses — Deep Dive

> **Source:** [MS Learn — Get started with lakehouses in Microsoft Fabric](https://learn.microsoft.com/en-us/training/modules/get-started-lakehouses/)
> **Related pages:** [OneLake & Storage (HTML)](../../big-picture/onelake.html) | [Big Picture Study Guide](../../big-picture.html)

---

## Learning Objectives

- Describe core features and capabilities of lakehouses in Microsoft Fabric.
- Create a lakehouse.
- Ingest and transform data in a lakehouse.
- Query and analyze lakehouse data with SQL and Spark.

---

## Unit 1: Introduction

A **lakehouse** in Microsoft Fabric combines the flexible storage of a data lake with the analytical capabilities of a data warehouse. A lakehouse uses Apache Spark and SQL compute engines to process and analyze data at scale, and is built on the **OneLake** storage layer.

A lakehouse is a unified platform that combines:
- The flexible and scalable storage of a data **lake**
- The ability to query and analyze data of a data ware**house**

**Scenario:** Imagine your organization relies on a traditional data warehouse for business analytics. The warehouse handles structured transactional data well, but struggles with the growing volume of semi-structured and unstructured data from sources like application logs, IoT devices, and external feeds. Storing and processing these diverse data types requires separate systems, creating data silos and complex integration efforts. Your organization needs a unified solution that handles both structured and unstructured data while maintaining strong analytical capabilities.

In this module, you learn how lakehouses address these challenges — how to create a lakehouse, ingest and transform data, and query lakehouse data using SQL and Spark. You also learn how well-structured lakehouse data supports downstream analytics and AI-powered experiences across the Microsoft Fabric platform.

---

## Unit 2: Describe Lakehouse Features and Capabilities

Traditional analytics architectures often force you to choose between two approaches. Data lakes offer flexibility and scalability but lack the structure and performance needed for business analytics. Data warehouses provide strong analytical capabilities but struggle with diverse data formats and can be costly to scale. **Lakehouses** bridge this gap by bringing database-like capabilities directly to your data lake, eliminating the need to maintain separate systems for different workloads.

### Understand Lakehouse Design

A lakehouse organizes data into two main areas: **Tables** and **Files**.

**Tables folder** — contains Delta Lake tables that provide structured, queryable data:
- Support SQL queries through the SQL analytics endpoint
- Enforce schemas and support ACID transactions
- Can be accessed in Power BI for reporting
- Benefit from automatic optimization and maintenance

**Files folder** — stores raw or semi-structured data files in their native format:
- Support any file format (CSV, JSON, Parquet, images, documents)
- Provide flexibility for data exploration and processing
- Can be staged before transformation into tables
- Don't enforce schema or support direct SQL queries

This separation lets you maintain both raw data (for compliance or reprocessing) and structured tables (for analytics) within the same lakehouse. You can process files using Spark notebooks or Dataflows Gen2, then load the results into tables for querying and reporting.

### Understand Delta Lake Tables

At the heart of a lakehouse are **Delta Lake tables**. Delta Lake is an open-source storage layer that brings reliability to data lakes. When you create a table in a lakehouse, the data is stored in Delta format in the underlying OneLake storage.

Delta Lake tables provide several key advantages:

| Feature | Description |
|---|---|
| **ACID transactions** | Ensures data consistency even when multiple users read and write data simultaneously |
| **Schema enforcement** | Validates that the data you write matches the table schema, preventing corrupt data |
| **Time travel** | Maintains a transaction log that lets you query previous versions of your data or roll back changes |
| **Efficient updates and deletes** | Unlike traditional data lake files, Delta tables support efficient update and delete operations |

Each Delta table consists of **Parquet data files** plus a **transaction log** that tracks all changes. This architecture enables both batch and streaming workloads to work reliably with the same data.

### Manage Lakehouse Access

Fabric provides layered access controls to secure lakehouse data at multiple levels:

- **Workspace roles** — for collaborators who need access to all items in the workspace
- **Item-level sharing** — to grant read-only access for specific needs (analytics, Power BI report development)
- **Row-level and column-level security** — via the SQL analytics endpoint, restrict what specific users see
- **Schema-level permissions** — control access by business domain when tables are organized into schemas
- **Data governance** — sensitivity labels, extended by Microsoft Purview integration

> **Further reading:** [Security in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/security/security-overview)

### Build a Foundation for Intelligent Analytics

The data you structure in a lakehouse doesn't just serve traditional reports and dashboards. Well-organized lakehouse data becomes the foundation that intelligent experiences across Microsoft Fabric depend on.

- When you create tables with clear schemas, consistent naming conventions, and descriptive column names, you make data accessible to both human analysts and AI-powered tools
- **Fabric IQ data agents** can query lakehouse tables through the SQL analytics endpoint, translating natural language questions into SQL queries — the quality of answers depends directly on how well you structure and document your data
- **Copilot in Power BI** can generate reports and answer business questions when it can reason over clearly defined tables and relationships
- The same lakehouse data can feed **semantic models** that support natural language exploration across Microsoft 365 experiences

**Key insight:** Good data engineering practices in the lakehouse create a reusable foundation for intelligent experiences across the platform.

---

## Unit 3: Ingest and Transform Data in a Lakehouse

### Create and Explore a Lakehouse

You create lakehouses within a Fabric-enabled workspace. A lakehouse has two main storage areas and a SQL analytics endpoint.

When you create a lakehouse, **schemas are enabled by default**. A schema named **dbo** is created automatically. Schemas let you organize tables into logical groups based on business domains or functions (e.g., `sales`, `marketing`, `hr`).

Schema-enabled lakehouses also support:
- Schema-level permissions
- Cross-workspace queries using the four-part namespace: `workspace.lakehouse.schema.table`

> **Tip:** Clear schema organization improves discoverability for everyone working with the lakehouse, including Fabric IQ data agents that translate natural language questions into SQL queries.

You can work with your lakehouse in **two modes**:

| Mode | Description |
|---|---|
| **Lakehouse explorer** | Add and interact with tables, files, and folders. Manage data, upload files, create tables. Can add reference lakehouses to browse/manage tables across multiple lakehouses side by side. |
| **SQL analytics endpoint** | Query Delta tables using T-SQL in *read-only* mode. Create views, functions, and apply SQL security, but can't modify underlying data. |

### Ingest Data into a Lakehouse

Ingesting data is the first step in your ETL (extract, transform, load) process:

| Method | Description |
|---|---|
| **Upload** | Upload local files or folders directly through the lakehouse explorer |
| **Load to Table** | Select a file/folder → "Load to Table" to create a Delta table without code. Supports Parquet and CSV, append or overwrite. |
| **Dataflows Gen2** | Import and transform data using Power Query |
| **Notebooks** | Use Apache Spark to ingest, transform, and load data programmatically |
| **Data Factory pipelines** | Use the Copy data activity to move data from external sources |

> **Further reading:** [Options to get data into the Fabric Lakehouse](https://learn.microsoft.com/en-us/fabric/data-engineering/load-data-lakehouse)

### Access Data Using Shortcuts

**Shortcuts** let you integrate data into your lakehouse without copying it. A shortcut references data in external storage and makes it appear as a folder in your lakehouse.

Key points:
- Reduce data duplication — create shortcuts to different storage accounts, other cloud providers, or other Fabric items
- OneLake manages source data permissions and credentials
- When accessing a shortcut to another OneLake location, OneLake uses your identity to authorize access
- **Schema shortcuts** map an entire schema to a folder of Delta tables in another lakehouse or in ADLS Gen2 — all referenced tables appear as local tables

> **Further reading:** [OneLake shortcuts documentation](https://learn.microsoft.com/en-us/fabric/onelake/onelake-shortcuts)

### Transform Data in a Lakehouse

Most data requires transformation before loading into tables. You might ingest raw data into the Files area, then transform and load it into tables.

| Tool | Best For |
|---|---|
| **Notebooks** | Data engineers familiar with PySpark, SQL, Scala. Copilot can generate transformation code from natural language and explain existing Spark code. |
| **Dataflows Gen2** | Users familiar with Power BI or Excel — uses the Power Query interface |
| **Pipelines** | Visual interface to orchestrate ETL processes, with activities running in sequence or parallel |

---

## Unit 4: Query and Analyze Lakehouse Data

### Query Data Using the SQL Analytics Endpoint

The SQL analytics endpoint provides **read-only** access to lakehouse tables using T-SQL queries. Automatically created with every lakehouse.

Common use cases:
- **Ad-hoc queries** — quickly investigate data to answer business questions
- **BI connections** — connect Power BI, Excel, or Azure Data Studio to retrieve data for reports
- **Data validation** — verify transformation results after loading or processing

You can create **SQL views** to store reusable query logic — useful for business rules, simplifying complex joins, or providing curated data for downstream consumers.

The endpoint supports **row-level security** and **column-level security**.

> **Tip:** Copilot for SQL queries can help you write T-SQL queries from natural language descriptions. Describe what you want to analyze, and Copilot suggests query code.

### Query Data Using Spark Notebooks

Notebooks provide a flexible, code-based environment for querying and analyzing lakehouse data.

**Spark SQL vs PySpark:**

| Approach | How It Works | Best For |
|---|---|---|
| **Spark SQL** | SQL syntax within a notebook cell, e.g. `SELECT * FROM schema.table` | Familiar SQL patterns |
| **PySpark** | Python code using `spark.sql()` or DataFrame API (`df.select()`, `df.filter()`) | Complex transformations, Python libraries |

Common use cases:
- **Exploratory data analysis** — investigate patterns, outliers, and relationships
- **Complex transformations** — business logic easier to express in code than SQL
- **Cross-workspace queries** — use four-part namespace (`workspace.lakehouse.schema.table`) to join data across multiple lakehouses

> **Tip:** Copilot for notebooks can generate PySpark or Spark SQL code from natural language prompts and explain existing code.

### Analyze and Visualize with Power BI

Power BI is the BI and reporting layer in Fabric — the consumption layer where business users access data.

Two ways to connect to lakehouse data:

| Connection | Description |
|---|---|
| **SQL analytics endpoint** | Analysts connect directly using Power BI or Excel to run ad-hoc queries |
| **Semantic model** | References specific lakehouse tables with defined relationships, measures, and business logic |

When building reports on a lakehouse semantic model, Power BI uses **Direct Lake** mode by default:
- Reads data directly from Delta Lake Parquet files
- No importing or copying data
- Fast query performance
- Reports always reflect current lakehouse data

Semantic models also support downstream intelligent experiences — when you define clear relationships and business measures, Copilot in Power BI can generate visualizations and answer business questions by reasoning over your lakehouse data.

---

## Knowledge Check

1. **What is a Microsoft Fabric lakehouse?**
   - ~~A relational database based on the Microsoft SQL Server database engine~~
   - ~~A hierarchy of folders and files in Azure Data Lake Store Gen2~~
   - **An analytical store that combines the file storage flexibility of a data lake with the SQL-based query capabilities of a data warehouse** ✓

2. **What is the main difference between the lakehouse explorer and SQL analytics endpoint?**
   - ~~The lakehouse explorer provides read-only access, while the SQL analytics endpoint allows data modifications~~
   - **Lakehouse explorer enables interaction with tables, files, and folders, while SQL analytics endpoint provides read-only T-SQL querying of Delta tables** ✓
   - ~~Both provide identical functionality with different user interfaces~~

3. **You want to include data in an external ADLS Gen2 location in your lakehouse, without copying. What should you do?**
   - ~~Create a Data pipeline that uses a Copy Data activity to load the external data into a file~~
   - **Create a shortcut** ✓
   - ~~Create a Dataflow Gen2 that extracts the data and loads it into a table~~

4. **You have CSV files in your lakehouse Files area and want to create Delta tables without writing code. What should you use?**
   - ~~A notebook with PySpark code~~
   - **Load to Table** ✓
   - ~~The SQL analytics endpoint~~

5. **You want to use Apache Spark to interactively explore data in a file in the lakehouse. What should you do?**
   - **Create a notebook** ✓
   - ~~Switch to the SQL analytics endpoint mode~~
   - ~~Create a Dataflow Gen2~~

6. **What connection mode does Power BI use by default when connecting to a lakehouse semantic model?**
   - ~~Import mode, which copies data into Power BI~~
   - ~~DirectQuery mode, which queries the source in real-time~~
   - **Direct Lake mode, which reads directly from Delta Lake files without copying data** ✓

---

## Summary

A lakehouse in Microsoft Fabric eliminates the divide between flexible file storage and structured analytics by combining both capabilities in a single platform, built on OneLake.

Key takeaways:
- **Tables + Files** — two storage areas within one lakehouse; tables for structured Delta Lake data, files for raw/semi-structured data
- **Delta Lake** — open-source storage layer providing ACID transactions, schema enforcement, time travel, and efficient updates
- **SQL analytics endpoint** — auto-generated read-only T-SQL interface with row/column-level security
- **Ingestion methods** — Upload, Load to Table, Dataflows Gen2, Notebooks, Data Factory pipelines
- **Shortcuts** — reference external data without copying; schema shortcuts map entire schemas
- **Two query engines** — SQL analytics endpoint for T-SQL, Spark notebooks for PySpark/SparkSQL
- **Power BI Direct Lake** — reads Delta files directly, no import/copy, always current
- **AI foundation** — well-structured lakehouse data supports Copilot, Fabric IQ data agents, and semantic models

---

## Links

- [Data Engineering in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/data-engineering/)
- [Options to get data into the Fabric Lakehouse](https://learn.microsoft.com/en-us/fabric/data-engineering/load-data-lakehouse)
- [OneLake shortcuts documentation](https://learn.microsoft.com/en-us/fabric/onelake/onelake-shortcuts)
- [Security in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/security/security-overview)
