# Copilot in Microsoft Fabric

> **Source:** [Copilot in Fabric Overview](https://learn.microsoft.com/en-us/fabric/fundamentals/copilot-fabric-overview)
> **Related pages:** [Fundamentals Overview](../fundamentals.md) | [Core Concepts](concepts.md)

---

## Learning Objectives

- Describe what Copilot in Fabric is and how it integrates across workloads.
- Identify Copilot capabilities specific to each Fabric workload.
- Understand how to enable Copilot at the tenant and workspace level.
- Explain privacy, security, and data handling for Copilot interactions.
- Describe regional availability requirements.

---

## Key Concepts

### Overview

**Copilot in Microsoft Fabric** is an AI assistant powered by Azure OpenAI large language models, integrated directly into Fabric workloads. It helps users write code, build queries, generate reports, create data pipelines, and understand data — all through natural language.

Key principles:

- Copilot is **context-aware** — it understands the schemas, tables, relationships, and metadata in your current workspace
- It operates **within Fabric's security boundary** — Copilot only accesses data the user has permission to see
- Responses are generated in real time and are **not stored or used to train models**
- Copilot is available across most Fabric workloads, with capabilities tailored to each

### Copilot by Workload

#### Data Engineering (Spark Notebooks)

- **Generate code** — describe a transformation in natural language, Copilot writes PySpark, Scala, or SparkSQL code
- **Explain code** — select existing code and ask Copilot to explain what it does
- **Fix errors** — paste an error message and get suggested fixes
- **Chat panel** — conversational interface within the notebook for iterative code development
- **Code completion** — inline suggestions as you type (similar to GitHub Copilot)

#### Data Science

- **Generate ML code** — create model training, evaluation, and prediction code from descriptions
- **Explore data** — ask Copilot to generate exploratory data analysis code (visualizations, statistics)
- **MLflow integration** — generate experiment tracking code and model logging boilerplate
- **Explain results** — ask Copilot to interpret model metrics and outputs

#### Data Factory

- **Build Dataflows Gen2** — describe the data transformation you need, Copilot generates Power Query steps
- **Pipeline assistance** — get help configuring pipeline activities, expressions, and error handling
- **Suggest transformations** — Copilot analyzes your data preview and suggests relevant cleaning or reshaping steps

#### Data Warehouse

- **Write T-SQL** — describe what you want to query, Copilot generates SELECT statements, joins, aggregations
- **Explain queries** — paste a complex T-SQL query and ask Copilot to break it down
- **Generate DDL** — describe a table structure and Copilot generates CREATE TABLE statements
- **Quick actions** — fix, explain, or optimize selected SQL code with one click

#### SQL Database

- **Natural language to SQL** — write queries against operational SQL databases using plain language
- **Schema exploration** — ask Copilot about table structures, relationships, and column meanings
- **Query optimization** — get suggestions for improving query performance

#### Power BI

- **Create reports** — describe the report you want, Copilot generates pages with appropriate visuals
- **Generate DAX measures** — describe a business calculation, Copilot writes the DAX formula
- **Summarize data** — generate narrative summaries of report pages or visual data
- **Create semantic models** — get help defining relationships, hierarchies, and measures
- **Q&A enhancement** — Copilot improves natural language question answering over semantic models

#### Real-Time Intelligence

- **Write KQL queries** — describe what you want to analyze, Copilot generates Kusto Query Language
- **Explain KQL** — select a KQL query and ask for a plain-language explanation
- **Eventstream assistance** — get help configuring streaming transformations and routing

### Enabling Copilot

Copilot availability is controlled at multiple levels:

| Level | Setting | Who Controls It |
|-------|---------|----------------|
| **Tenant** | "Copilot and Azure OpenAI Service" toggle in Admin Portal → Tenant Settings | Fabric Admin |
| **Capacity** | Can be enabled/disabled per capacity | Capacity Admin |
| **Workspace** | Inherits from tenant/capacity settings; cannot override if disabled above | Workspace Admin |

Prerequisites:

- Fabric capacity must be **F64 or higher** (or P1+ for Power BI Premium) for Copilot features
- Smaller SKUs (F2-F32) have limited or no Copilot access
- The tenant must be in a **supported region** for Azure OpenAI processing

### Privacy and Security

Copilot in Fabric follows strict data handling principles:

- **No data used for model training** — your prompts, data, and Copilot responses are not used to train or improve the underlying Azure OpenAI models
- **Data stays in your boundary** — Copilot processes data within your organization's compliance boundary
- **Respects existing permissions** — Copilot can only access data the current user has permission to see; it does not bypass workspace roles, RLS, or item-level security
- **No cross-tenant data sharing** — Copilot interactions are isolated to your Fabric tenant
- **Audit logging** — Copilot usage events are captured in the tenant audit log
- **Responsible AI** — responses include content filtering, and Copilot may decline to answer inappropriate or harmful requests

### Regional Availability

Copilot in Fabric requires Azure OpenAI Service, which is available in specific Azure regions.

- If your Fabric capacity is in a supported region, Copilot processes data in that region
- If your capacity is in an **unsupported region**, you can opt in to allow data to be processed in a different region where Azure OpenAI is available (cross-geo processing)
- Cross-geo processing must be explicitly enabled by the Fabric Admin in tenant settings
- Check the official documentation for the current list of supported regions, as Microsoft regularly expands availability

---

## Key Terms

| Term | Definition |
|------|-----------|
| **Copilot** | An AI assistant integrated into Fabric workloads, powered by Azure OpenAI large language models |
| **Azure OpenAI Service** | The Microsoft-managed service providing GPT models that power Copilot |
| **Cross-geo processing** | Allowing Copilot requests to be processed in a different Azure region when the capacity's region does not support Azure OpenAI |
| **Content filtering** | Automated safeguards that prevent Copilot from generating harmful, biased, or inappropriate content |
| **Tenant setting** | A configuration in the Fabric Admin Portal that controls feature availability across the entire organization |

---

## Further Reading

- [Copilot in Fabric Overview](https://learn.microsoft.com/en-us/fabric/fundamentals/copilot-fabric-overview)
- [Copilot in Fabric FAQ](https://learn.microsoft.com/en-us/fabric/fundamentals/copilot-faq-fabric)
- [Privacy, Security, and Responsible AI for Copilot](https://learn.microsoft.com/en-us/fabric/fundamentals/copilot-privacy-security)
- [Enable Copilot in Fabric](https://learn.microsoft.com/en-us/fabric/admin/service-admin-portal-copilot)
- [Copilot for Power BI](https://learn.microsoft.com/en-us/power-bi/create-reports/copilot-introduction)
