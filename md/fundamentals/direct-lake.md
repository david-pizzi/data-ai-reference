# Direct Lake Mode — Deep Dive

> **Source:** [Direct Lake Overview](https://learn.microsoft.com/en-us/fabric/fundamentals/direct-lake-overview)
> **Related pages:** [Fundamentals Overview](../fundamentals.md) | [Semantic Models & BI](../semantic-bi.md)

---

## Learning Objectives

- Explain what Direct Lake mode is and how it differs from Import and DirectQuery.
- Describe transcoding and how columns are loaded into memory.
- Understand framing and automatic updates.
- Identify when DirectQuery fallback occurs and how to manage it.
- Apply capacity guardrails to plan Direct Lake deployments by SKU.
- Determine when to use Direct Lake vs Import vs DirectQuery.

---

## Key Concepts

### What Is Direct Lake?

**Direct Lake** is a connectivity mode for Power BI semantic models that reads data **directly from Delta Lake Parquet files** in OneLake. It combines the performance benefits of Import mode with the freshness of DirectQuery — without copying data into the semantic model or querying the source in real time.

```
Import:       Data copied into Power BI's in-memory engine (VertiPaq)
DirectQuery:  Queries sent to the source database on every interaction
Direct Lake:  Data read directly from Parquet files in OneLake into memory on demand
```

### Import vs DirectQuery vs Direct Lake

| Dimension | Import | DirectQuery | Direct Lake |
|-----------|--------|-------------|-------------|
| **Data freshness** | Stale until refresh | Always current | Current as of last Delta commit (frame) |
| **Data location** | Copied into VertiPaq | Stays in source | Stays in OneLake (Parquet files) |
| **Query performance** | Fastest (in-memory) | Depends on source | Near-Import speed |
| **Data size limit** | Model size limit (capacity-dependent) | No limit (queries fan out) | Guardrails per SKU (rows, size) |
| **Refresh needed** | Yes (scheduled or on-demand) | No | No (automatic framing) |
| **Data duplication** | Yes | No | No |
| **Supported sources** | 200+ connectors | 200+ connectors | OneLake Delta tables only |

### Transcoding (Column Loading)

When a DAX query references a column, Direct Lake **transcodes** the Parquet data into the VertiPaq columnar format in memory. This happens on demand:

- Only referenced columns are loaded — not the entire table
- Transcoded columns are cached in memory for subsequent queries
- If memory pressure occurs, least-recently-used columns are evicted
- Transcoding is significantly faster than a full Import refresh because it reads compressed Parquet directly

**Key insight:** Direct Lake does not "import" data in the traditional sense. It reads Parquet files on demand and transcodes them into the in-memory format column by column, only when needed.

### Framing

A **frame** is a snapshot of the Delta table's state that the semantic model uses to determine which Parquet files to read.

- When a semantic model is first opened, it reads the Delta transaction log to identify the current set of Parquet files — this is the "frame"
- The frame is **automatically updated** when the underlying Delta table changes (new data is written, compaction occurs, etc.)
- Automatic framing happens transparently — no manual refresh is needed
- You can also trigger framing manually via the XMLA endpoint or the Fabric portal
- Framing is lightweight — it only reads the Delta log, not the data itself

### Automatic Updates

Direct Lake semantic models stay current without scheduled refreshes:

1. Data is written to a Delta table (via Spark, Pipeline, Dataflow, T-SQL, etc.)
2. A new Delta commit is created (new Parquet files, updated transaction log)
3. The semantic model detects the change and updates its frame
4. The next DAX query reads the new Parquet files

This eliminates the refresh cycle that Import mode requires, reducing latency between data landing and data being available in reports.

### DirectQuery Fallback

In certain situations, Direct Lake **falls back to DirectQuery** mode, sending queries to the SQL analytics endpoint instead of reading Parquet files directly.

Common fallback triggers:

| Trigger | Description |
|---------|-------------|
| **Guardrail exceeded** | The table exceeds the row count or memory size limit for the SKU |
| **Unsupported DAX** | Certain DAX patterns that cannot be served from Parquet files |
| **Column not in Parquet** | Calculated columns or columns added via views (not in the physical Delta table) |
| **RLS/OLS with certain patterns** | Some row-level or object-level security configurations |

When fallback occurs:

- Query performance degrades to DirectQuery speed (depends on the SQL analytics endpoint)
- The semantic model logs a fallback event
- You can **disable fallback** to force errors instead of slow queries — useful for performance-critical reports

To disable fallback, set the `DirectLakeBehavior` property to `DirectLakeOnly` on the semantic model.

### Capacity Guardrails by SKU

Each Fabric capacity SKU has limits on what Direct Lake can handle before triggering fallback.

| SKU | Max rows per table | Max model size on disk | Max columns per table |
|-----|-------------------|----------------------|---------------------|
| **F2** | 300 million | 10 GB | 1,000 |
| **F4** | 300 million | 10 GB | 1,000 |
| **F8** | 300 million | 10 GB | 1,000 |
| **F16** | 300 million | 20 GB | 1,000 |
| **F32** | 600 million | 40 GB | 1,000 |
| **F64** | 1.5 billion | 80 GB | 1,500 |
| **F128** | 3 billion | 160 GB | 1,500 |
| **F256** | 6 billion | 320 GB | 1,500 |
| **F512** | 12 billion | 640 GB | 1,500 |
| **F1024** | 24 billion | 1,280 GB | 1,500 |
| **F2048** | 24 billion | 2,560 GB | 1,500 |

> **Note:** These guardrails are per-table, not per-model. Check the official documentation for the latest values as Microsoft updates these over time.

### When to Use Each Mode

| Scenario | Recommended Mode |
|----------|-----------------|
| Data is in OneLake (lakehouse or warehouse), freshness matters, and data fits within guardrails | **Direct Lake** |
| Data is in OneLake but exceeds guardrails, and you cannot reduce table sizes | **DirectQuery** (to SQL analytics endpoint) |
| Data is not in OneLake (external SQL Server, Oracle, etc.) | **Import** or **DirectQuery** |
| Maximum query performance is critical and data can be refreshed on a schedule | **Import** |
| Real-time data from an operational system, sub-second freshness required | **DirectQuery** |
| Composite model — some tables from OneLake, some from external sources | **Direct Lake + DirectQuery** (composite) |

---

## Key Terms

| Term | Definition |
|------|-----------|
| **Direct Lake** | A Power BI semantic model connectivity mode that reads Delta Lake Parquet files directly from OneLake |
| **Transcoding** | The process of reading Parquet data and converting it to VertiPaq columnar format in memory, on demand |
| **Framing** | The process of reading the Delta transaction log to identify the current set of Parquet files for a table |
| **DirectQuery fallback** | When Direct Lake cannot serve a query from Parquet files and falls back to querying the SQL analytics endpoint |
| **Guardrail** | A capacity-level limit (row count, model size) that determines when Direct Lake falls back to DirectQuery |
| **VertiPaq** | The in-memory columnar storage engine used by Power BI for Import and Direct Lake modes |
| **Delta transaction log** | The `_delta_log/` folder in a Delta table that records all changes (inserts, updates, deletes, compaction) |

---

## Further Reading

- [Direct Lake Overview](https://learn.microsoft.com/en-us/fabric/fundamentals/direct-lake-overview)
- [Direct Lake Guardrails](https://learn.microsoft.com/en-us/fabric/fundamentals/direct-lake-overview#guardrails)
- [Direct Lake vs Import vs DirectQuery](https://learn.microsoft.com/en-us/power-bi/connect-data/service-dataset-modes-understand)
- [Semantic Models in Fabric](https://learn.microsoft.com/en-us/power-bi/connect-data/service-datasets-understand)
