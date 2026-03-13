# Phase 4: The Semantic & BI Layer — Deep Dive

> **Related pages:** [Phase 4 Study Guide (HTML)](../html/semantic-bi.html) | [Fabric Overview](../html/fabric-overview.html)
> **Official docs:** [Power BI Documentation](https://learn.microsoft.com/en-us/power-bi/)

---

## Why Semantic Models Matter

Semantic models are the **bridge between data engineering and business consumption** — and increasingly, the bridge to Fabric IQ's AI-powered semantic layer.

In traditional analytics, the gap between "data is in the warehouse" and "business users can answer questions" is filled by reports, dashboards, and ad-hoc queries. Semantic models formalise this gap into a **governed, reusable abstraction layer** that defines:

- What business terms mean (Revenue = SUM of line items after discounts, excluding returns)
- How entities relate (Customer → Orders → Products → Categories)
- Who can see what (RLS rules applied once, inherited by every report)
- How time intelligence works (YTD, QoQ, rolling averages)

**The IQ connection:** Fabric IQ can generate ontologies directly from Power BI semantic models. When you define a star schema with measures, relationships, and hierarchies in Power BI, you're simultaneously building the foundation for an IQ ontology. Tables become entity types, relationships become graph edges, measures become properties. Understanding semantic models is therefore **essential** for understanding IQ — they're not separate topics.

---

## Semantic Model Anatomy

A Power BI semantic model (formerly called a "dataset") contains several components:

### Tables
- **Fact tables:** event-level data (sales transactions, web clicks, support tickets). Typically large, narrow, and append-only.
- **Dimension tables:** context for facts (date, customer, product, geography). Typically small, wide, and slowly changing.
- **Bridge tables:** resolve many-to-many relationships (e.g., a customer with multiple accounts).

### Relationships
- **One-to-many (1:*):** the standard pattern. Dimension → Fact. The "one" side is the dimension key, the "many" side is the fact foreign key.
- **Cross-filter direction:** single (default, dimension filters fact) or both (bidirectional — use sparingly, performance implications).
- **Active vs inactive:** only one active relationship between two tables at a time. Inactive relationships can be activated in DAX using `USERELATIONSHIP()`.
- **Cardinality matters:** Power BI uses relationships to generate efficient SQL/DAX queries. Wrong cardinality = wrong numbers.

### Measures
- Defined in DAX, evaluated at query time (not stored in the table).
- Examples: `Total Revenue := SUM(Sales[Amount])`, `YTD Revenue := TOTALYTD([Total Revenue], 'Date'[Date])`
- Measures respect filter context — a measure on a report page filtered to "UK" automatically calculates for UK only.

### Hierarchies
- Logical groupings for drill-down: Year → Quarter → Month → Day, or Country → Region → City.
- Define once in the model, available in every report built on it.

### Row-Level Security (RLS)
- DAX filter expressions applied to tables: `[Region] = USERPRINCIPALNAME()`.
- Users see only the rows their RLS rule permits — applied at the model level, inherited by all reports.

---

## Star Schema Design

The star schema is the **standard pattern** for analytical models. It's called a "star" because fact tables sit at the centre with dimension tables radiating outward.

### Fact Tables
- Contain **measurable events**: sales amount, quantity, duration, count.
- Grain matters — define the grain first (one row per transaction? per day? per customer-product-day?).
- Foreign keys to dimension tables. No business descriptions — those live in dimensions.

### Dimension Tables
- Contain **descriptive attributes**: customer name, product category, date month/quarter/year.
- Denormalised (flat) — don't normalise dimensions into snowflake schemas. Star > snowflake for analytical performance.
- Include a surrogate key (integer) as the primary key.

### Best Practices
- **One fact, many dimensions:** a well-designed model has a clear central fact table (or a few) surrounded by shared dimensions.
- **Conformed dimensions:** Date, Customer, Product dimensions shared across multiple fact tables ensure consistent filtering.
- **Avoid wide fact tables:** keep facts narrow (keys + measures). Move descriptions to dimensions.
- **Role-playing dimensions:** the same Date dimension can play multiple roles (Order Date, Ship Date) via inactive relationships.

### Visual Example

```
         ┌──────────┐
         │   Date   │
         └────┬─────┘
              │ 1:*
┌──────────┐  │  ┌───────────┐
│ Customer ├──┼──┤   Sales   │
└──────────┘  │  │   (Fact)  │
              │  └─────┬─────┘
         ┌────┴─────┐  │
         │ Product  │──┘
         └──────────┘
```

In IQ terms: Customer, Product, and Date become **entity types**. The relationships between them become **graph edges**. Measures like Total Revenue become **properties** on the Sales entity.

---

## DAX Conceptual Overview

DAX (Data Analysis Expressions) is the formula language used in Power BI semantic models. You need to understand **what DAX does** conceptually, not write complex formulas.

### What DAX Does
- **Defines measures:** calculations evaluated at query time (Total Revenue, Profit Margin, YTD Growth).
- **Defines calculated columns:** row-level computations stored in the table (Full Name = First & Last, Age Band).
- **Defines calculated tables:** virtual tables created from expressions (Date table, bridge tables).
- **Applies security:** RLS rules are DAX filter expressions.

### Core Concept: Filter Context
Every DAX measure evaluates within a **filter context** — the combination of slicers, filters, rows, and columns on a report visual. This is the key insight:

- A report visual showing Revenue by Region automatically provides a filter context where each row filters to one Region.
- DAX measures respond to this context automatically — no "WHERE clause" needed.

### Common Measure Patterns
| Pattern | Example | What It Does |
|---------|---------|--------------|
| Simple aggregation | `SUM(Sales[Amount])` | Totals a column |
| Count distinct | `DISTINCTCOUNT(Sales[CustomerID])` | Unique customers |
| Year-to-date | `TOTALYTD([Revenue], 'Date'[Date])` | Cumulative from Jan 1 |
| Prior year | `CALCULATE([Revenue], SAMEPERIODLASTYEAR('Date'[Date]))` | Same metric, previous year |
| Ratio | `DIVIDE([Profit], [Revenue], 0)` | Safe division (handles zero) |

### CALCULATE Explained Simply
`CALCULATE` is the most important DAX function. It does one thing: **evaluates a measure in a modified filter context**.

```
CALCULATE(
    [Total Revenue],           -- the measure to evaluate
    'Date'[Year] = 2024        -- override the filter context to 2024 only
)
```

Think of `CALCULATE` as "give me this number, but change what rows are included." It's how you build comparison measures (this year vs last year), running totals, and conditional aggregations.

---

## DirectLake Mode

DirectLake is Fabric's **breakthrough connectivity mode** for semantic models — unique to Fabric and a key differentiator.

### The Three Modes

| Mode | How It Works | Pros | Cons |
|------|-------------|------|------|
| **Import** | Data copied into Power BI's in-memory engine (VertiPaq) | Fastest queries, full DAX support | Data refresh needed, memory limits, data duplication |
| **DirectQuery** | Queries sent to source at runtime | Always current, no duplication | Slower queries, limited DAX, source must handle load |
| **DirectLake** | Reads Delta/Parquet files directly from OneLake into memory on demand | Near-Import speed, no refresh needed, no duplication | Fabric-only, some DAX limitations, falls back to DirectQuery if data exceeds memory |

### Why DirectLake Matters
- **No scheduled refreshes:** data changes in the lakehouse are immediately available in the semantic model.
- **No data duplication:** unlike Import mode, DirectLake doesn't copy data — it reads directly from OneLake.
- **Performance:** columnar Parquet/Delta format is already optimised for analytical queries. VertiPaq-like speed without the import step.
- **Fallback behaviour:** if a query exceeds memory or hits an unsupported pattern, DirectLake silently falls back to DirectQuery. Monitor for this — frequent fallbacks mean the model needs optimisation.

### When to Use Each Mode
- **DirectLake:** default choice for any semantic model in Fabric pointing to lakehouse/warehouse tables.
- **Import:** when you need absolute maximum query performance and can tolerate scheduled refreshes (e.g., executive dashboards with complex DAX).
- **DirectQuery:** when data freshness is critical and the source can handle query load (e.g., real-time operational dashboards, Azure SQL).

---

## Power BI Security Model

Security in Power BI operates at multiple levels. Understanding this is important because **semantic model security flows through to IQ ontologies**.

### Workspace Roles
| Role | Can Do |
|------|--------|
| **Admin** | Full control — settings, membership, delete workspace |
| **Member** | Publish content, edit items, share reports |
| **Contributor** | Edit items, create reports — cannot share or manage access |
| **Viewer** | View reports and dashboards only |

### Row-Level Security (RLS)
- Defined in the semantic model using DAX filter expressions.
- Applied per user or security group.
- Static RLS: hardcoded filter (e.g., `[Region] = "UK"`).
- Dynamic RLS: filter based on user identity (e.g., `[ManagerEmail] = USERPRINCIPALNAME()`).
- **Testing:** use "View as Role" in Power BI Desktop to verify.

### Object-Level Security (OLS)
- Hides entire tables or columns from specific roles.
- Use case: salary data visible to HR role but hidden from general viewers.
- Defined in Tabular Editor (not natively in Power BI Desktop).

### Sensitivity Labels
- Microsoft Purview sensitivity labels (Confidential, Internal, Public) applied to semantic models, reports, and dashboards.
- Labels travel with the data — if you export to Excel, the label follows.
- Can enforce encryption and restrict sharing.

---

## Data Science in Fabric

Fabric's Data Science workload integrates ML workflows directly into the platform, with outputs that feed into semantic models and Power BI.

### Key Components
- **Notebooks:** Jupyter-style notebooks with PySpark, Python, R, and Scala support. Direct access to OneLake data.
- **Experiments:** MLflow-backed experiment tracking. Log metrics, parameters, and artefacts for each model run.
- **Model Registry:** centralised store for trained models. Version management, staging (Development → Staging → Production).
- **MLflow Integration:** open-source ML lifecycle management. Fabric uses MLflow natively — no separate MLflow server needed.

### How Data Science Connects to BI
1. Train a model in a Fabric notebook (e.g., churn prediction).
2. Register the model in the model registry.
3. Use `PREDICT()` in a Spark notebook or dataflow to score new data — predictions written back to a lakehouse table.
4. The prediction table appears in the semantic model via DirectLake.
5. Power BI report shows churn risk alongside business metrics.

This pipeline — **train → register → predict → visualise** — is a powerful story for customers who want ML integrated into their BI without a separate ML platform.

---

## The Bridge: Semantic Model → Ontology → Graph → Data Agent

This is the **most important conceptual flow** in Phase 4 — it connects everything from Phase 2 (IQ) through Phase 4 (semantic models).

### The Pipeline

```
┌─────────────┐    ┌──────────────┐    ┌─────────────┐    ┌──────────────┐
│  Lakehouse  │───→│   Semantic   │───→│  Ontology   │───→│  Data Agent  │
│  / Warehouse│    │    Model     │    │  (IQ)       │    │  (IQ)        │
└─────────────┘    └──────────────┘    └─────────────┘    └──────────────┘
   Phase 3             Phase 4             Phase 2             Phase 2
   Raw/curated         Business terms      Knowledge graph     Natural language
   data in Delta       measures, KPIs      entity types        queries over
                       relationships       graph edges         the ontology
```

### How the Conversion Works
| Semantic Model Component | Ontology Component | Example |
|--------------------------|-------------------|---------|
| Table | Entity Type | `Sales` table → `Sale` entity type |
| Relationship (1:*) | Graph Edge | Customer → Sales becomes a `purchased` edge |
| Measure | Property | `[Total Revenue]` becomes a numeric property on Sale |
| Hierarchy | Navigation Path | Year → Quarter → Month becomes a temporal navigation path |
| RLS Rule | Access Policy | Region-based RLS carries forward as ontology access control |

### Why This Matters
- Customers don't need to build ontologies from scratch — **they already have semantic models**.
- The pitch: "You've already defined your business logic in Power BI. Fabric IQ turns that into an AI-ready knowledge graph automatically."
- This is the **complete Fabric story**: ingest → store → transform → model → report → reason. No other platform does this end-to-end.

---

## MS Learn Modules — Details

### 1. Get Started with Data Science in Microsoft Fabric (~30 min)
- Overview of the Data Science workload within Fabric.
- Creating and running ML notebooks with PySpark.
- Experiment tracking with MLflow — logging parameters, metrics, and models.
- Model registry basics — versioning and deploying models.
- [Start module →](https://learn.microsoft.com/en-us/training/modules/get-started-data-science-fabric/)

### 2. Design Scalable Semantic Models (~30 min)
- Star schema principles applied to Power BI semantic models.
- Choosing the right grain for fact tables.
- Optimising for query performance — reducing model size, aggregations.
- Composite models and hybrid tables.
- [Start module →](https://learn.microsoft.com/en-us/training/modules/design-scalable-semantic-models/)

### 3. Create Power BI Model Relationships (~45 min)
- Relationship types: one-to-many, many-to-many, one-to-one.
- Cross-filter direction and its impact on DAX calculations.
- Active vs inactive relationships — when and why.
- Role-playing dimensions (multiple relationships between two tables).
- [Start module →](https://learn.microsoft.com/en-us/training/modules/create-power-bi-model-relationships/)

### 4. Understand Power BI Model Security (~30 min)
- Implementing Row-Level Security (RLS) with DAX filters.
- Static vs dynamic RLS patterns.
- Object-Level Security (OLS) for column/table hiding.
- Workspace roles and how they interact with RLS.
- [Start module →](https://learn.microsoft.com/en-us/training/modules/enforce-power-bi-model-security/)

---

## Hands-On Exercise Walkthrough

This exercise is the **single most valuable exercise** in the entire learning plan because it connects Phases 2, 3, and 4 into one end-to-end flow.

### Prerequisites
- Completed Phase 3 (data is in a lakehouse/warehouse with medallion-layer tables).
- Fabric trial or capacity available.
- Power BI Desktop installed (or use the web-based model editor in Fabric).

### Step 1: Create a Semantic Model
1. Navigate to your lakehouse in the Fabric portal.
2. Open the **SQL analytics endpoint** — this exposes lakehouse tables as SQL views.
3. Select the Gold-layer tables (cleaned, aggregated) and choose **New semantic model**.
4. In the model view, verify relationships: dimensions connected to fact tables via 1:* relationships.
5. Create 2-3 basic measures: `Total Revenue := SUM(Sales[Amount])`, `Order Count := COUNTROWS(Sales)`.

### Step 2: Build a Power BI Report
1. From the semantic model, choose **Create report**.
2. Add a bar chart (Revenue by Product Category), a card (Total Revenue), and a table (Top 10 Customers).
3. Verify the report is using **DirectLake mode** — check the connection type in the model settings.
4. Add a date slicer and confirm that filtering works across all visuals.

### Step 3: Generate an Ontology
1. Navigate to your workspace and create a new **Ontology** item (IQ workload).
2. Select "Create from semantic model" and choose the model from Step 1.
3. Review the generated ontology: tables → entity types, relationships → edges, measures → properties.
4. Explore the graph view — see how your star schema maps to a knowledge graph.

### Step 4: Connect a Data Agent
1. From the ontology, create a **Data Agent**.
2. Ask natural language questions: "What was total revenue last quarter?" or "Which customer had the most orders?"
3. Observe how the agent uses the ontology to translate questions into queries.
4. This completes the loop: raw data → semantic model → ontology → natural language access.

### What Success Looks Like
You can describe the full pipeline to a customer: "Your data lands in OneLake, gets transformed through medallion layers, surfaces as a semantic model with business measures, and then Fabric IQ turns that model into a knowledge graph that AI agents can reason over — all within one platform, no stitching required."

---

## Key Terms Glossary

| Term | Definition |
|------|-----------|
| **Semantic model** | A curated analytical model in Power BI containing tables, relationships, measures, and hierarchies. Formerly called a "dataset." |
| **DAX** | Data Analysis Expressions — the formula language for measures and calculations in Power BI semantic models. |
| **Star schema** | Design pattern with central fact tables (events) surrounded by dimension tables (context). Standard for analytical models. |
| **Fact table** | A table containing measurable events — sales, clicks, transactions. Contains foreign keys and numeric measures. |
| **Dimension table** | A table containing descriptive attributes — customer name, product category, dates. Provides context for facts. |
| **DirectLake** | Fabric-native connectivity mode that reads Delta/Parquet files directly from OneLake into memory. No import or scheduled refresh needed. |
| **DirectQuery** | Connectivity mode where queries are sent to the source at runtime. Always current but slower than Import or DirectLake. |
| **Import mode** | Connectivity mode where data is copied into Power BI's in-memory engine (VertiPaq). Fastest queries but requires refresh. |
| **VertiPaq** | Power BI's in-memory columnar storage engine. Compresses data for fast analytical queries. Used by Import and DirectLake modes. |
| **RLS** | Row-Level Security — DAX filter expressions that restrict which rows a user can see in a semantic model. |
| **OLS** | Object-Level Security — hides entire tables or columns from specific roles. More restrictive than RLS. |
| **CALCULATE** | The most important DAX function. Evaluates a measure in a modified filter context. |
| **Filter context** | The combination of slicers, filters, rows, and columns that determine which data a DAX measure operates on. |
| **Ontology** | A Fabric IQ item that models business concepts, rules, and relationships as a labelled property graph. Can be generated from semantic models. |
| **Entity type** | An ontology concept corresponding to a business object (Customer, Product, Sale). Maps from semantic model tables. |
| **Graph edge** | An ontology relationship between entity types. Maps from semantic model relationships. |
| **MLflow** | Open-source ML lifecycle management platform. Fabric uses MLflow natively for experiment tracking and model registry. |
| **Model registry** | Centralised store for trained ML models with versioning and staging (Development → Staging → Production). |
| **Sensitivity label** | Microsoft Purview classification applied to Fabric items (Confidential, Internal, Public). Travels with exported data. |
