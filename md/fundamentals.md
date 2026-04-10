# Fabric Fundamentals — Deep Dive

> **Related pages:** [Fabric Overview](fabric-overview.md) | [Big Picture](big-picture.md)
> **Official docs:** [Microsoft Fabric Fundamentals](https://learn.microsoft.com/en-us/fabric/fundamentals/)

---

## Overview

The Fabric Fundamentals section covers the foundational concepts, tools, and governance features that every Microsoft Fabric practitioner needs to understand. These modules build a solid base before diving into specific workloads like Data Engineering, Data Warehouse, or Power BI.

---

## Modules

### 1. [Core Concepts](fundamentals/concepts.md)

What Microsoft Fabric is, its SaaS architecture, the nine workloads (Power BI, Data Factory, Data Engineering, Data Science, Data Warehouse, Databases, Real-Time Intelligence, Fabric IQ, Industry Solutions), OneLake as the unified storage layer, Real-Time Hub, and essential terminology.

### 2. [Data Lifecycle](fundamentals/data-lifecycle.md)

The six-stage data lifecycle in Fabric — Get Data, Store Data, Prepare & Transform, Analyze & Train, Track & Visualize, and External Integration. Covers ingestion methods (Eventstreams, Pipelines, Mirroring, Shortcuts, Copy Job), storage options, transformation tools, analytics engines, and visualization.

### 3. [Workspaces](fundamentals/workspaces.md)

Workspaces as the foundational organizational unit in Fabric. Covers the four workspace roles (Admin, Member, Contributor, Viewer) with a detailed permissions matrix, workspace layout, settings, monitoring, administration, auditing, and the recycle bin.

### 4. [Direct Lake](fundamentals/direct-lake.md)

The Direct Lake connectivity mode for semantic models — how it differs from Import and DirectQuery, transcoding and column loading, framing, automatic updates, DirectQuery fallback behavior, capacity guardrails by SKU, and guidance on when to use each mode.

### 5. [Copilot in Fabric](fundamentals/copilot.md)

Overview of Copilot capabilities across all Fabric workloads — Data Engineering, Data Science, Data Factory, Data Warehouse, SQL Database, Power BI, and Real-Time Intelligence. Covers how to enable Copilot, privacy and security considerations, and regional availability.

### 6. [Task Flows](fundamentals/task-flows.md)

Task flows as a visual way to organize and connect Fabric items into end-to-end workflows. Covers the canvas, tasks, and connectors, the ten task types, and how to create, manage, and share task flows within workspaces.

### 7. [Decision Guides](fundamentals/decision-guides.md)

Guidance for choosing the right Fabric data store (Eventhouse, Cosmos DB, SQL Database, Warehouse, Lakehouse), mapping personas and skillsets to workloads, Lakehouse vs Warehouse decision criteria, and comparing Copy activity, Dataflow Gen2, Eventstream, and Spark for data movement.

### 8. [Governance](fundamentals/governance.md)

Fabric governance features — endorsement levels (Promotion, Certification, Master Data), sensitivity labels via Microsoft Purview, item sharing and permissions, job scheduling, Delta Lake interoperability, and table maintenance operations.

---

## Further Reading

- [Microsoft Fabric Fundamentals Documentation](https://learn.microsoft.com/en-us/fabric/fundamentals/)
- [Microsoft Fabric Overview](https://learn.microsoft.com/en-us/fabric/get-started/microsoft-fabric-overview)
- [Get Started with Fabric Learning Path](https://learn.microsoft.com/en-us/training/paths/get-started-fabric/)
