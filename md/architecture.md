# Phase 5: Architecture Patterns & Strategic Context — Deep Dive

> **Related pages:** [Phase 5 Study Guide (HTML)](../html/architecture.html) | [Fabric Overview](../html/fabric-overview.html)
> **Official docs:** [Microsoft Fabric Documentation](https://learn.microsoft.com/en-us/fabric/)

---

## Data Mesh on Fabric

Data mesh is an organisational and architectural paradigm that treats data as a product, owned by the domain teams that produce it. Microsoft Fabric's architecture maps naturally to data mesh principles.

### Domain Ownership

Each business domain (e.g., Finance, Supply Chain, Customer Analytics) gets its own Fabric workspace. The workspace acts as the boundary of ownership — the domain team controls what data is produced, how it is transformed, and who can access it. This mirrors the data mesh principle of domain-oriented decentralised ownership.

### OneLake Shortcuts for Cross-Domain Access

OneLake shortcuts are the mechanism that makes data mesh work without data duplication. A consumer domain can create a shortcut to a producer domain's lakehouse table. The data stays in the producer's workspace, governed by their access policies, but is queryable from the consumer's workspace as if it were local. This eliminates the need for centralised ETL pipelines to move data between domains.

### Workspace Boundaries

Workspaces provide natural governance boundaries:

- **Access control:** Each workspace has its own RBAC — admin, member, contributor, viewer roles
- **Capacity assignment:** Workspaces can be assigned to different Fabric capacities, enabling cost allocation per domain
- **Lineage and cataloguing:** OneLake Catalog tracks items across workspaces, providing tenant-wide visibility without centralised control

### Federated Governance

The data mesh principle of federated computational governance maps to Fabric's layered governance model:

- **Tenant-level policies:** Admin portal settings, sensitivity labels via Purview, audit logging
- **Domain-level governance:** Domain admins manage their workspace policies, data quality, and access
- **Platform-level standards:** Naming conventions, medallion architecture patterns, and data product contracts enforced by convention and automation

### Data Products

A data product in Fabric is a well-defined, discoverable, and trustworthy dataset published for consumption. In practice, this is a lakehouse table or warehouse view that is:

- Documented with descriptions in OneLake Catalog
- Labelled with sensitivity classifications via Purview
- Certified or endorsed by the domain team
- Accessible via shortcut to other domains

---

## Lakehouse vs Warehouse Decision Framework

Fabric offers both a Lakehouse and a Warehouse experience. They share OneLake storage (Delta format) but differ in compute engine, query language, and intended audience. Many architectures use both.

### Decision Tree

Ask these questions to guide the recommendation:

1. **What are the team's primary skills?**
   - SQL/T-SQL first → Warehouse
   - Python/Spark first → Lakehouse
   - Mixed → Both (lakehouse for engineering, warehouse for analytics)

2. **What are the dominant query patterns?**
   - Complex joins, stored procedures, views → Warehouse
   - Large-scale transformations, ML feature engineering → Lakehouse
   - Ad-hoc exploration with notebooks → Lakehouse

3. **What data types are involved?**
   - Structured relational data only → Warehouse
   - Semi-structured (JSON, Parquet) or unstructured (images, logs) → Lakehouse
   - Mix of both → Lakehouse as landing zone, Warehouse for curated layer

4. **What is the data volume?**
   - Moderate volumes with complex business logic → Warehouse
   - Very large volumes requiring distributed processing → Lakehouse (Spark)

### Use Case Examples

| Scenario | Recommendation | Rationale |
|----------|---------------|-----------|
| Traditional BI reporting with complex business rules | Warehouse | T-SQL stored procedures, views, complex joins |
| IoT data processing and ML feature engineering | Lakehouse | Semi-structured data, Spark-based transformations |
| Departmental analytics for SQL-skilled team | Warehouse | Familiar tooling, lower learning curve |
| Data science team building ML pipelines | Lakehouse | Notebook-first workflow, PySpark, MLflow integration |
| Enterprise data platform serving multiple teams | Both | Lakehouse for engineering, Warehouse for curated analytics |

### Key Insight

Both Lakehouse and Warehouse store data in Delta format in OneLake. A table created in the Lakehouse is readable from the Warehouse SQL endpoint, and vice versa. The choice is primarily about compute engine and user experience, not about data storage.

---

## Migration Patterns

### From Azure Synapse Analytics

Synapse is the direct predecessor to Fabric. The migration is the most straightforward:

1. **Dedicated SQL pools** → Fabric Warehouse. T-SQL compatibility is high. Export table definitions and data, recreate in Fabric Warehouse. Some syntax differences exist (e.g., distribution hints are not needed — Fabric handles this automatically).
2. **Spark pools** → Fabric Data Engineering (Lakehouse notebooks). PySpark code is largely portable. Key change: Fabric uses a managed Spark runtime with auto-scaling, no cluster configuration.
3. **Synapse pipelines** → Fabric Data Factory. Pipeline JSON is compatible. Activities map directly. Copy-and-reconfigure approach works.
4. **Serverless SQL pools** → Fabric Lakehouse SQL endpoint. The pattern of querying files with T-SQL maps to the Lakehouse SQL analytics endpoint.
5. **Synapse Link** → Fabric Mirroring. The concept of real-time replication from operational databases continues with Mirroring in Fabric.

**Phased approach:** Start with pipelines (Data Factory), then migrate storage to OneLake, then switch compute from Synapse pools to Fabric workloads.

### From Standalone Databricks

Organisations running Databricks independently (not as part of Fabric) can migrate incrementally:

1. **Delta tables in ADLS** → Create OneLake shortcuts to existing ADLS storage. No data movement required. Fabric can query the same Delta tables Databricks writes to.
2. **Notebooks** → Port PySpark notebooks to Fabric Data Engineering. The Fabric Spark runtime is based on Databricks Runtime, so compatibility is high.
3. **Unity Catalog** → Fabric OneLake Catalog + Purview. Governance capabilities overlap but are not identical. Plan for metadata migration.
4. **MLflow experiments** → Fabric Data Science includes MLflow. Model registry and experiment tracking carry over with minimal changes.
5. **Jobs and workflows** → Fabric pipelines for orchestration. Databricks Workflows map to Fabric pipeline activities.

**Key advantage:** OneLake shortcuts mean you can run Fabric workloads alongside existing Databricks workloads during transition, without duplicating data.

### From On-Premises Data Warehouses

Moving from SQL Server, Oracle, or Teradata on-premises:

1. **Assessment:** Inventory tables, views, stored procedures, ETL jobs. Identify dependencies and complexity.
2. **Landing zone:** Set up a Fabric Lakehouse as the landing zone. Use Data Factory pipelines with on-premises data gateway to ingest data.
3. **Medallion pattern:** Implement bronze (raw), silver (cleansed), gold (curated) layers in the Lakehouse. This replaces the traditional staging-ODS-DW pattern.
4. **T-SQL migration:** For stored procedure-heavy workloads, use Fabric Warehouse as the target. T-SQL compatibility is high for SQL Server migrations; Oracle and Teradata require more translation.
5. **BI migration:** Rebuild reports in Power BI using DirectLake mode for optimal performance against Fabric data.
6. **Cutover:** Run parallel environments during validation, then switch over.

**Timeline expectation:** On-premises migrations are typically 3-12 months depending on complexity. The medallion pattern often simplifies the architecture compared to the original.

---

## AI-Infused Analytics

### How the Pieces Connect

Microsoft's AI-infused analytics story in Fabric involves three layers working together:

```
Azure AI Foundry (custom agents, RAG, fine-tuned models)
         ↕
Fabric IQ (ontology, business semantics, data agents)
         ↕
Fabric Platform (OneLake data, semantic models, Copilot)
```

### Fabric IQ and Ontologies

IQ introduces an ontology layer — a formal model of business concepts, their relationships, and rules. An ontology for a retail business might define concepts like Customer, Order, Product, and Store, with relationships between them and business rules (e.g., "a VIP customer has lifetime spend > £10,000").

This ontology sits on top of OneLake data and semantic models. When an AI agent or Copilot query asks "show me VIP customer trends," the ontology provides the business context that makes the answer accurate and consistent.

### Copilot Integration

Copilot is embedded across Fabric workloads:

- **Power BI Copilot:** Natural-language questions about reports and dashboards, grounded in the semantic model
- **Data Engineering Copilot:** Code generation in Spark notebooks
- **Data Factory Copilot:** Pipeline and dataflow authoring assistance
- **SQL Copilot:** T-SQL query generation in the Warehouse

When IQ ontologies are available, Copilot uses them to understand business terminology, improving accuracy and consistency of responses.

### Azure AI Foundry Agents

For scenarios beyond what Copilot offers, Azure AI Foundry enables custom AI agents that can:

- Query Fabric data via OneLake APIs
- Use IQ ontologies for business context
- Combine with external knowledge sources (RAG pattern)
- Execute multi-step reasoning workflows
- Integrate with enterprise applications

The pattern is: Fabric provides the data and business semantics, AI Foundry provides the custom intelligence layer.

---

## Competitive Positioning

### vs Snowflake

**Snowflake's strengths:**
- True cross-cloud deployment (AWS, Azure, GCP) with data sharing across clouds
- Snowflake Marketplace for third-party data sets
- Clean separation of storage and compute with per-second billing
- Strong SQL-based analytics experience
- Snowpark for Python/Java/Scala workloads

**Where Fabric wins:**
- **End-to-end platform:** Snowflake is primarily a data warehouse/lakehouse. Fabric includes BI (Power BI), data integration (Data Factory), real-time analytics, data science, and semantic AI — all in one platform.
- **No data movement:** In a Snowflake architecture, data must move between Snowflake and separate BI, ETL, and ML tools. In Fabric, all workloads share OneLake.
- **Enterprise BI:** Power BI is the market leader in BI (Gartner Magic Quadrant). Snowflake has no native BI — customers use Tableau, Looker, or Power BI alongside it.
- **IQ and Copilot:** Fabric's semantic AI layer (IQ ontologies, embedded Copilot) has no direct Snowflake equivalent. Snowflake Cortex is the response but is less mature.
- **M365 integration:** Fabric connects natively to Teams, Excel, SharePoint — where enterprise users already work.

**Honest acknowledgement:** Snowflake's cross-cloud story is stronger for multi-cloud organisations. If a customer is committed to multi-cloud data sharing, acknowledge this strength.

### vs Databricks (Standalone)

**Databricks' strengths:**
- Pioneer of Delta Lake and the lakehouse architecture
- Unity Catalog is a mature governance layer
- MLflow is the de facto open-source ML lifecycle tool
- Strong data engineering and ML community
- Cloud-agnostic (AWS, Azure, GCP)
- Deep Spark expertise and optimisation

**Where Fabric wins:**
- **Fabric includes Databricks Runtime:** The Spark engine in Fabric is powered by Databricks Runtime. Customers get Databricks-quality Spark execution plus everything else Fabric offers.
- **Built-in BI:** Databricks requires a separate BI tool (typically Power BI or Tableau). Fabric includes Power BI natively with DirectLake for optimal performance.
- **Unified governance:** Fabric uses Purview for governance across all workloads. Databricks Unity Catalog governs Databricks assets but not the BI layer.
- **Simpler billing:** One Fabric capacity covers all workloads. Databricks has separate billing for compute, storage, and each workspace.
- **IQ semantic layer:** The ontology and business semantics layer in Fabric has no direct Databricks equivalent.

**Honest acknowledgement:** Databricks has deeper ML/AI capabilities for advanced data science teams. If a customer's primary workload is ML engineering, Databricks standalone may offer more flexibility.

### vs AWS Stack (Redshift + Glue + Athena + SageMaker)

**AWS strengths:**
- Broadest cloud service catalogue
- Mature individual services (Redshift is battle-tested at scale)
- SageMaker for end-to-end ML workflows
- Strong ecosystem and partner network
- Deep integration with AWS infrastructure services

**Where Fabric wins:**
- **Single platform:** An equivalent AWS architecture requires Redshift (warehouse) + Glue (ETL) + Athena (serverless queries) + S3 (storage) + Lake Formation (governance) + QuickSight (BI) + SageMaker (ML). That is 7+ services to configure, secure, and bill separately. Fabric is one platform.
- **One security model:** Each AWS service has its own IAM policies, encryption settings, and audit configuration. Fabric has one security model across all workloads.
- **One billing model:** AWS billing for analytics is notoriously complex (per-node, per-query, per-GB scanned, per-training-hour). Fabric bills by capacity units.
- **Enterprise BI:** AWS QuickSight has limited market share compared to Power BI. Most AWS customers use Tableau or Power BI anyway, adding another integration point.

**Honest acknowledgement:** AWS's breadth is unmatched. For organisations deeply invested in AWS infrastructure, the cost of moving to Fabric may outweigh the simplification benefit.

### vs GCP Stack (BigQuery + Vertex AI + Dataflow)

**GCP strengths:**
- BigQuery's serverless architecture scales effortlessly
- Competitive pricing, especially for analytical queries
- Vertex AI is strong for ML model training and deployment
- BigQuery ML enables ML directly in SQL
- Dataflow (Apache Beam) for streaming and batch processing

**Where Fabric wins:**
- **Enterprise BI maturity:** Power BI is the enterprise BI standard. Google's Looker is growing but has lower enterprise penetration. Fabric includes Power BI natively.
- **M365 integration:** Fabric connects to the Microsoft 365 ecosystem — Teams, Excel, SharePoint, Outlook. For organisations running on M365 (which is most enterprises), this is a natural extension.
- **Copilot everywhere:** Fabric's embedded Copilot across all workloads, grounded in IQ ontologies, provides AI assistance that GCP's analytics tools do not yet match.
- **Unified platform:** Like AWS, GCP requires multiple services (BigQuery + Dataflow + Vertex AI + Looker + Cloud Storage) to match Fabric's scope. Fabric is one platform with one governance model.

**Honest acknowledgement:** BigQuery's serverless model and pricing are compelling for pure analytical workloads. For GCP-native organisations with limited BI requirements, BigQuery + Vertex AI is a strong combination.

---

## Recommended Resources

| Resource | Description | URL |
|----------|-------------|-----|
| **Fabric Documentation** | Official Microsoft documentation covering all workloads, architecture, administration, and best practices | [learn.microsoft.com/en-us/fabric/](https://learn.microsoft.com/en-us/fabric/) |
| **Fabric Blog** | Official blog with product announcements, feature releases, roadmap updates, and deep technical posts | [blog.fabric.microsoft.com](https://blog.fabric.microsoft.com) |
| **Data Mozart** | Independent Fabric analysis by Nikola Ilic — practical tips, performance insights, and honest evaluations from a practitioner perspective | [data-mozart.com](https://data-mozart.com) |
| **Fabric Community** | Official community forums for questions, discussions, best practices, and peer support | [community.fabric.microsoft.com](https://community.fabric.microsoft.com) |

---

## Full Plan Summary

| Week | Phase | Focus | Hands-On |
|------|-------|-------|----------|
| 1 | The Big Picture | Platform architecture, OneLake, workloads overview | Explore Fabric trial |
| 2-3 | Fabric IQ | Ontology, labelled property graph, AI agents | Build an ontology |
| 3-5 | Data Platform | Lakehouse, Warehouse, Spark, Real-Time Intelligence | End-to-end data flow |
| 5-6 | Semantic & BI | Semantic models, Power BI, DirectLake | Model to Ontology |
| **7+** | **Architecture** | **Design patterns, competitive positioning, IP creation** | **Reading & strategy** |

---

## Key Links

| Resource | URL |
|----------|-----|
| Fabric Trial | [learn.microsoft.com/en-us/fabric/get-started/fabric-trial](https://learn.microsoft.com/en-us/fabric/get-started/fabric-trial) |
| Training Hub | [learn.microsoft.com/en-us/training/fabric/](https://learn.microsoft.com/en-us/training/fabric/) |
| Get Started Path | [learn.microsoft.com/en-us/training/paths/get-started-fabric/](https://learn.microsoft.com/en-us/training/paths/get-started-fabric/) |
| IQ Documentation | [learn.microsoft.com/en-us/fabric/iq/overview](https://learn.microsoft.com/en-us/fabric/iq/overview) |
| Ontology Tutorial | [learn.microsoft.com/en-us/fabric/iq/ontology-tutorial](https://learn.microsoft.com/en-us/fabric/iq/ontology-tutorial) |
| IQ Fundamentals Module | [learn.microsoft.com/en-us/training/modules/explore-fabric-iq/](https://learn.microsoft.com/en-us/training/modules/explore-fabric-iq/) |
| Fabric Blog | [blog.fabric.microsoft.com](https://blog.fabric.microsoft.com) |
| Power BI Desktop | [powerbi.microsoft.com/desktop/](https://powerbi.microsoft.com/desktop/) |
