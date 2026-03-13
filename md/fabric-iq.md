# Phase 2: Fabric IQ & the Intelligence Layer — Deep Dive

> **Related pages:** [Phase 2 Study Guide (HTML)](../html/fabric-iq.html) | [Fabric Overview](../html/fabric-overview.html)
> **Official docs:** [Fabric IQ Documentation](https://learn.microsoft.com/en-us/fabric/iq/overview)

---

## The Shift: Data Platform to Intelligence Platform

Fabric started as a unified **data platform** — replacing Azure Synapse, Data Factory, ADLS, and Power BI Premium under one SaaS roof. That alone was a major simplification for customers. But with IQ, Fabric takes a fundamentally different step: it moves from being a platform that *stores and queries data* to one that *understands business meaning and acts on it*.

The progression looks like this:

```
Data Lake → Lakehouse → Semantic Models → Ontology → Graph → Agents
   (raw)     (structured)    (measures)     (meaning)  (relationships)  (action)
```

Each layer adds more **semantic richness**:

- **Data Lake / Lakehouse:** Raw and curated data in Delta format. Fabric's storage foundation.
- **Semantic Models (Power BI):** DAX measures, relationships, row-level security. Defines *how to calculate*.
- **Ontology (IQ):** Entity types, properties, business rules, and relationships. Defines *what things are and how they connect*.
- **Graph (IQ):** Live, queryable knowledge graph auto-generated from the ontology. Enables traversals and pattern matching.
- **Agents (IQ):** AI-powered actors that use the ontology as grounding to answer questions (data agents) or take action (operations agents).

IQ does not replace the existing workloads — it builds on top of them. OneLake stores the data, Spark and SQL transform it, semantic models define measures, and IQ adds the business context layer that makes AI agents possible.

---

## Core Concepts

### 1. Ontology

An ontology is a **machine-readable model of business vocabulary**. It defines:

- **Entity types:** Customer, Product, Order, Supplier, Store Location
- **Properties:** Customer.Name, Product.SKU, Order.TotalAmount
- **Relationships:** Customer *places* Order, Order *contains* Product, Product *supplied by* Supplier
- **Rules and constraints:** "A VIP customer has lifetime spend > £50,000", "High-risk supplier has compliance score < 70"

The ontology is **bound to real data in OneLake** — entity types map to tables, properties map to columns, relationships map to foreign keys or explicit edge definitions.

**How it differs from a semantic model:**
- A Power BI semantic model defines *how to calculate* (measures, aggregations, time intelligence).
- An ontology defines *what things are* and *how they relate* (business vocabulary, rules, constraints).
- They are complementary. An ontology can be created *from* a semantic model, inheriting its structure and enriching it with business meaning.

### 2. Graph

IQ includes **native graph storage and compute** — a labelled property graph where:

- **Nodes** = entity instances (a specific customer, a specific order)
- **Edges** = relationships (this customer placed this order)
- **Properties** on both nodes and edges

The ontology **auto-generates** a queryable graph. Once you define entity types and relationships in the ontology, IQ materialises a graph that you can query using **GQL (Graph Query Language)**.

This enables questions that are difficult or impractical with relational SQL:

- "Which suppliers are connected to our highest-risk products through fewer than 3 hops?"
- "What is the shortest path between this customer and this return incident?"
- "Show me all entities within 2 relationships of this flagged transaction."

### 3. Data Agents

Data agents are **virtual analysts** that use the ontology as their grounding context. They:

- Accept **natural language questions** from users
- Convert those questions into **structured queries** against the graph and underlying data
- Return **auditable answers** — you can see the query that was generated, the data sources that were accessed, and the reasoning chain

Because data agents are grounded in the ontology (not just raw table schemas), their answers are **consistent and reliable**. The ontology provides the business vocabulary that prevents ambiguity — when a user asks about "active customers", the agent knows exactly what that means because the ontology defines it.

### 4. Operations Agents

Operations agents are **autonomous, always-on agents** that:

- **Monitor real-time streams** (from Eventhouse / Real-Time Intelligence)
- **Detect constraint violations** defined in the ontology (e.g., "supplier compliance score dropped below threshold")
- **Trigger workflows** — send alerts, create tickets, invoke Power Automate flows, call Azure Functions

Operations agents use the ontology rules as an **operational playbook**. They connect to **Azure AI Foundry** for orchestration and can be composed with other agents in multi-agent architectures.

The key difference from data agents: data agents *answer questions*, operations agents *take action*.

---

## How Ontology Connects to Semantic Models

The relationship between Power BI semantic models and IQ ontologies is a natural evolution:

```
Power BI Semantic Model          Ontology (IQ)
─────────────────────           ──────────────
Tables & columns          →     Entity types & properties
Relationships (FK)        →     Edges in the graph
DAX measures              →     Computed properties / rules
Row-level security        →     Access policies on entities
```

You can **create an ontology directly from a semantic model** — IQ will import the structure, relationships, and metadata. You then enrich it with:

- Additional entity types not in the semantic model
- Business rules and constraints
- Streaming data sources (from Eventhouse)
- Cross-domain relationships

This makes the adoption path straightforward for customers who already have mature Power BI deployments: start with what you have, then layer on ontology and agents.

---

## The Full IQ Pipeline

```
Data (OneLake)
    ↓
Semantic Model (Power BI)
    ↓
Ontology (IQ)          ← define entity types, properties, rules, relationships
    ↓
Graph (IQ)             ← auto-generated from ontology, queryable via GQL
    ↓
┌─────────────┐  ┌──────────────────┐
│ Data Agents │  │ Operations Agents │
│ (answer)    │  │ (act)             │
└─────────────┘  └──────────────────┘
```

1. **Data** lives in OneLake — lakehouses, warehouses, Eventhouse
2. **Semantic models** define measures and calculations for BI
3. **Ontology** adds business vocabulary, rules, and relationships on top
4. **Graph** is auto-materialised — a live, queryable knowledge graph
5. **Data agents** answer natural language questions grounded in the ontology
6. **Operations agents** monitor streams, detect violations, trigger actions

---

## MS Learn Modules

| # | Module | Duration | Description |
|---|--------|----------|-------------|
| 1 | **Understand Microsoft Fabric IQ fundamentals** | ~30 min | Structured learning module covering IQ concepts, architecture, and use cases |
| 2 | **[Fabric IQ Overview](https://learn.microsoft.com/en-us/fabric/iq/overview)** (documentation) | ~20 min | Official documentation covering the IQ workload, its components, and how it fits into Fabric |
| 3 | **[What is Ontology?](https://learn.microsoft.com/en-us/fabric/iq/ontology/overview)** (documentation) | ~20 min | Deep dive into the ontology concept — entity types, properties, relationships, and how ontologies connect to data |

**Recommended order:** Start with module 2 (IQ Overview) for the big picture, then module 3 (Ontology) for the foundational concept, then module 1 for the structured learning path.

---

## Ontology Tutorial Walkthrough (5 Parts)

The official tutorial uses the **Lakeshore Retail** sample dataset. It walks through creating an ontology, enriching it, exploring the graph, and building a data agent.

**Tutorial URL:** [learn.microsoft.com/en-us/fabric/iq/ontology/tutorial-0-introduction](https://learn.microsoft.com/en-us/fabric/iq/ontology/tutorial-0-introduction)

### Part 0: Introduction and Environment Setup

- Load the Lakeshore Retail sample data into a Fabric workspace
- Set up the lakehouse with tables for customers, orders, products, and suppliers
- Create the semantic model that the ontology will be built from
- **What you learn:** How IQ fits into an existing Fabric workspace and the prerequisites for creating an ontology

### Part 1: Create an Ontology

- Create a new ontology item in the workspace
- Import structure from the semantic model (entity types, properties, relationships)
- Alternatively, create entity types directly from OneLake tables
- **What you learn:** The two paths to ontology creation — from semantic model (recommended for existing BI deployments) or from raw data

### Part 2: Enrich the Ontology

- Add streaming data from Eventhouse as an additional data source
- Define new relationships not present in the original semantic model
- Add business rules and constraints (e.g., "VIP customer" definition)
- **What you learn:** Ontologies are living artefacts that evolve beyond the initial import. They can span batch and streaming data sources.

### Part 3: Preview the Ontology

- Explore the instance graph visually — see nodes and edges
- Run queries using business terms instead of SQL
- Drill into entity instances and follow relationships
- **What you learn:** How the ontology materialises into a queryable graph and how business users can explore data without writing code

### Part 4: Create a Data Agent

- Create a data agent grounded in the ontology
- Ask natural language questions: "Who are our top customers by order value?" / "Which products have the most returns?"
- Inspect the generated queries and data sources
- **What you learn:** How agents use the ontology as context, why grounding matters for accuracy, and what the audit trail looks like

---

## Why This Matters

### Early-Mover Advantage

Fabric IQ is in preview. Most practitioners are still focused on the traditional Fabric story — data engineering, lakehouse architecture, Power BI migration. Understanding IQ early positions you ahead of the curve.

### IP Creation Opportunities

1. **Ontology design patterns** — Reusable templates for common industry ontologies (retail, healthcare, financial services, public sector). Customers will need guidance on how to model their business domains.

2. **Agent integration architectures** — Reference architectures showing how data agents and operations agents connect to Azure AI Foundry, Copilot Studio, Power Automate, and existing enterprise workflows.

3. **Migration guidance** — How to evolve from traditional BI semantic models to ontology-backed intelligence. This is a natural upsell path for existing Fabric customers.

4. **Governance frameworks** — How ontology definitions and agent permissions fit into existing data governance (Purview, sensitivity labels, access control).

### The Strategic Angle

IQ is where Microsoft is investing for the next wave of Fabric. Being someone who can articulate the **ontology-to-agent story** — from raw data through semantic models to ontologies to graph to agents — is a differentiated position in the field. This is not just a feature walkthrough; it is the narrative for why Fabric is more than a data platform.

---

## Key Links and Resources

| Resource | URL |
|----------|-----|
| Fabric IQ Overview | [learn.microsoft.com/en-us/fabric/iq/overview](https://learn.microsoft.com/en-us/fabric/iq/overview) |
| Ontology Overview | [learn.microsoft.com/en-us/fabric/iq/ontology/overview](https://learn.microsoft.com/en-us/fabric/iq/ontology/overview) |
| Ontology Tutorial (Part 0) | [learn.microsoft.com/en-us/fabric/iq/ontology/tutorial-0-introduction](https://learn.microsoft.com/en-us/fabric/iq/ontology/tutorial-0-introduction) |
| Azure AI Foundry | [learn.microsoft.com/en-us/azure/ai-studio/](https://learn.microsoft.com/en-us/azure/ai-studio/) |
| Fabric Overview (HTML) | [../html/fabric-overview.html](../html/fabric-overview.html) |
| Phase 2 Study Guide (HTML) | [../html/fabric-iq.html](../html/fabric-iq.html) |

---

## Glossary of IQ-Specific Terms

| Term | Definition |
|------|-----------|
| **Ontology** | A machine-readable model of business vocabulary — entity types, properties, relationships, and rules bound to real data in OneLake |
| **Entity type** | A business concept defined in the ontology (e.g., Customer, Product, Order). Maps to a table or view in OneLake. |
| **Property** | An attribute of an entity type (e.g., Customer.Name, Order.TotalAmount). Maps to a column. |
| **Relationship** | A named connection between entity types (e.g., Customer *places* Order). Becomes an edge in the graph. |
| **Constraint / Rule** | A business rule defined in the ontology (e.g., "VIP customer has lifetime spend > £50,000"). Used by agents for reasoning. |
| **Graph** | The labelled property graph auto-generated from the ontology. Nodes = entity instances, edges = relationships. |
| **GQL** | Graph Query Language — the query language for traversing and querying the IQ graph |
| **Data agent** | An AI agent that answers natural language questions by querying the ontology-backed graph. Answers are grounded and auditable. |
| **Operations agent** | An autonomous AI agent that monitors real-time streams, detects constraint violations, and triggers workflows. |
| **Grounding** | The process of anchoring an AI agent's reasoning in defined business concepts (ontology) and real data, rather than relying on general knowledge. |
| **Azure AI Foundry** | Microsoft's platform for building, deploying, and managing AI agents and models. Operations agents connect to Foundry for orchestration. |
| **Instance graph** | The materialised graph containing actual data instances (specific customers, specific orders) as opposed to the schema-level ontology definition. |
| **Semantic model** | A Power BI concept — defines measures, relationships, and calculations. Can serve as the starting point for creating an ontology. |
