# Governance in Microsoft Fabric

> **Source:** [Governance in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/governance/governance-compliance-overview)
> **Related pages:** [Fundamentals Overview](../fundamentals.md) | [Workspaces](workspaces.md)

---

## Learning Objectives

- Describe the endorsement framework (Promotion, Certification, Master Data) and when to use each level.
- Explain how sensitivity labels from Microsoft Purview protect Fabric items.
- Understand item sharing and permission models.
- Configure job scheduling for automated data refresh and processing.
- Describe Delta Lake interoperability and its governance implications.
- Identify table maintenance operations and why they matter.

---

## Key Concepts

### Endorsement

Endorsement is Fabric's **trust signaling system** — it helps users identify which items are authoritative, vetted, and safe to use. Without endorsement, users in large organizations cannot distinguish a sandbox experiment from a production-ready dataset.

Three endorsement levels:

| Level | Who Can Apply | Meaning | Use Case |
|-------|--------------|---------|----------|
| **Promoted** | Item owner or workspace Member/Admin | "This item is ready for broader use" | Items that have been tested and are considered reliable by their creator |
| **Certified** | Designated certifiers (configured by Fabric Admin) | "This item meets organizational quality standards" | Items that have been reviewed and approved by a governance team or data steward |
| **Master Data** | Designated certifiers | "This is the authoritative, single source of truth for this data domain" | Gold-standard items — the canonical customer list, official revenue dataset, etc. |

Endorsement behavior:

- Endorsed items display a badge (Promoted, Certified, or Master Data) in the workspace and OneLake catalog
- Users can filter and search by endorsement level, making it easier to find trusted data
- Endorsement does **not** change permissions — it is a trust signal, not an access control
- Fabric Admins configure who can certify items and which domains support Master Data endorsement

### Sensitivity Labels

**Sensitivity labels** from Microsoft Purview Information Protection can be applied to Fabric items to classify and protect data based on its sensitivity.

How sensitivity labels work in Fabric:

| Feature | Description |
|---------|-------------|
| **Classification** | Labels like "Public," "General," "Confidential," "Highly Confidential" indicate the sensitivity of the data |
| **Inheritance** | Labels propagate downstream — if a lakehouse is labeled "Confidential," reports built from it inherit the label |
| **Protection** | Labels can enforce encryption and access restrictions when data is exported (e.g., to Excel or PDF) |
| **Visibility** | Labels are visible in the workspace item list, OneLake catalog, and lineage view |
| **Mandatory labeling** | Fabric Admins can require users to apply a sensitivity label before publishing items |
| **Default labels** | Admins can set a default label that applies automatically to new items |

Integration with Microsoft Purview:

- Labels are defined and managed centrally in Microsoft Purview compliance portal
- The same labels used for Microsoft 365 (emails, documents) extend to Fabric items
- Audit logs track label application, changes, and removal

### Item Sharing

Fabric provides multiple ways to share items with users who do not have workspace roles:

| Sharing Method | What It Grants | Use Case |
|---------------|---------------|----------|
| **Workspace role** | Access to all items in the workspace (based on role level) | Team members who need broad access |
| **Item-level sharing** | Read access to a specific item (e.g., a single report or semantic model) | Sharing a report with a stakeholder without giving workspace access |
| **Link sharing** | Generate a shareable link to an item | Quick sharing with specific people or groups |
| **App publishing** | Curated collection of reports/dashboards distributed to a broader audience | Organization-wide BI consumption |
| **SQL connection string** | Direct SQL access to a warehouse or SQL analytics endpoint | External tools (SSMS, Azure Data Studio, dbt) |

Permission layers (from broadest to most granular):

1. **Workspace roles** — Admin, Member, Contributor, Viewer
2. **Item permissions** — Read, Write, Reshare, Build (for semantic models)
3. **Row-level security (RLS)** — restrict which rows a user can see in a table
4. **Column-level security (CLS)** — restrict which columns a user can see
5. **Object-level security (OLS)** — hide entire tables or columns from specific users in semantic models

### Job Scheduling

Fabric supports scheduling for automated execution of data processing tasks.

| Item Type | Scheduling Capability |
|-----------|----------------------|
| **Data Pipelines** | Trigger-based scheduling (time, event, tumbling window), manual runs |
| **Dataflow Gen2** | Refresh schedule (daily, hourly, custom intervals) |
| **Notebook / Spark Job** | Schedule via pipeline, or Spark Job Definition with cron-like triggers |
| **Semantic Model Refresh** | Scheduled refresh (up to 48 times per day on Premium/Fabric capacities) |
| **Copy Job** | Built-in scheduling with configurable frequency |

Scheduling best practices:

- Stagger schedules to avoid capacity spikes — not everything should run at midnight
- Use pipeline dependencies to chain jobs (e.g., refresh lakehouse tables before refreshing the semantic model)
- Monitor scheduled jobs via the Monitoring Hub
- Configure failure alerts to detect broken data pipelines early

### Delta Lake Interoperability

All data in OneLake is stored as **Delta Lake** — an open-source format. This has significant governance implications:

| Aspect | Implication |
|--------|------------|
| **Open format** | Data is not locked into Fabric — it can be read by Spark, Databricks, DuckDB, Polars, or any Delta-compatible tool |
| **ADLS Gen2 APIs** | OneLake exposes data via standard ADLS Gen2 REST APIs, enabling external access |
| **Shortcut interoperability** | Fabric can read Delta tables from external ADLS Gen2, S3, or GCS via shortcuts |
| **Cross-platform governance** | The same Delta tables can be governed by both Fabric (sensitivity labels, endorsement) and external catalogs (Unity Catalog, Hive Metastore) |
| **No vendor lock-in** | Organizations can migrate away from Fabric without converting data — Delta is portable |

### Table Maintenance

Delta Lake tables require periodic maintenance to ensure optimal query performance and storage efficiency.

| Operation | What It Does | Why It Matters |
|-----------|-------------|---------------|
| **OPTIMIZE (Compaction)** | Merges small Parquet files into larger ones | Reduces the number of files, improving read performance and reducing storage overhead |
| **VACUUM** | Removes old Parquet files that are no longer referenced by the Delta log | Reclaims storage space; files older than the retention period (default 7 days) are deleted |
| **Z-ORDER** | Co-locates related data within Parquet files by specified columns | Improves query performance for filters on those columns (data skipping) |
| **V-ORDER** | Fabric-specific optimization that applies special sorting within Parquet files | Improves read performance for Power BI Direct Lake and SQL analytics endpoint |
| **Table maintenance policies** | Automatic, scheduled maintenance configured at the lakehouse level | Ensures tables stay optimized without manual intervention |

Maintenance considerations:

- **OPTIMIZE** should run after large batch writes to consolidate small files (the "small file problem")
- **VACUUM** should run periodically but only after confirming no long-running queries depend on old files
- **V-ORDER** is applied by default in Fabric for new writes — it optimizes Parquet files for the VertiPaq engine
- Lakehouse **table maintenance policies** can automate OPTIMIZE and VACUUM on a schedule, reducing operational burden

---

## Key Terms

| Term | Definition |
|------|-----------|
| **Endorsement** | A trust signal (Promoted, Certified, Master Data) applied to Fabric items to indicate their reliability and authority |
| **Sensitivity label** | A Microsoft Purview classification label (e.g., Confidential) applied to Fabric items to indicate data sensitivity |
| **Row-level security (RLS)** | A security mechanism that restricts which rows of data a user can see based on their identity |
| **Column-level security (CLS)** | A security mechanism that restricts which columns a user can see |
| **Object-level security (OLS)** | A security mechanism that hides entire tables or columns from users in a semantic model |
| **OPTIMIZE** | A Delta Lake maintenance operation that compacts small files into larger ones |
| **VACUUM** | A Delta Lake maintenance operation that removes old, unreferenced files to reclaim storage |
| **V-ORDER** | A Fabric-specific Parquet optimization that sorts data for fast VertiPaq transcoding |
| **Master Data** | The highest endorsement level, indicating the single authoritative source of truth for a data domain |

---

## Further Reading

- [Governance in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/governance/governance-compliance-overview)
- [Endorsement Overview](https://learn.microsoft.com/en-us/fabric/governance/endorsement-overview)
- [Sensitivity Labels in Fabric](https://learn.microsoft.com/en-us/fabric/governance/information-protection)
- [Delta Lake Table Maintenance](https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-table-maintenance)
- [Security in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/security/security-overview)
