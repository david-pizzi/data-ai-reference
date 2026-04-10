# Task Flows in Microsoft Fabric

> **Source:** [Task Flows Overview](https://learn.microsoft.com/en-us/fabric/fundamentals/task-flow-overview)
> **Related pages:** [Fundamentals Overview](../fundamentals.md) | [Workspaces](workspaces.md)

---

## Learning Objectives

- Explain what task flows are and the problem they solve.
- Describe the three key components: canvas, tasks, and connectors.
- Identify the ten task types and when to use each.
- Create and manage task flows within a workspace.

---

## Key Concepts

### What Are Task Flows?

A **task flow** is a visual, workflow-oriented way to organize and connect Fabric items within a workspace. Instead of seeing a flat list of items, task flows let you lay out your data workflow on a canvas — showing how data moves from ingestion through transformation to visualization.

Task flows solve the **discoverability problem**: in a workspace with dozens of items, it can be hard to understand how they relate. A task flow makes the end-to-end workflow visible at a glance.

Key characteristics:

- Task flows are **workspace-level** — they live inside a workspace and reference items in that workspace
- They are **visual** — items are arranged on a canvas with connectors showing data flow
- They are **non-executable** — task flows are documentation/navigation aids, not orchestration tools (use Data Pipelines for orchestration)
- Multiple task flows can exist in a single workspace for different workflows or perspectives

### Key Components

#### Canvas

The **canvas** is the visual workspace where you arrange tasks and connectors. It provides:

- A drag-and-drop surface for placing tasks
- Zoom and pan for navigating large workflows
- Grid snapping for alignment
- A clean, diagram-like view of your data workflow

#### Task

A **task** represents a step in your data workflow. Each task can be:

- **Linked to a Fabric item** — clicking the task navigates directly to the item (e.g., a specific notebook, pipeline, or report)
- **Unlinked** — a placeholder for a step that has not yet been built
- **Typed** — each task has a type that indicates its role in the workflow (see task types below)

Tasks display the item name, type icon, and status, making it easy to see what exists and what still needs to be created.

#### Connector

A **connector** is a visual arrow linking two tasks, showing the flow of data or the sequence of operations. Connectors are purely visual — they do not execute or trigger anything.

### Task Types

Task flows support ten task types, each representing a stage in the data lifecycle:

| Task Type | Purpose | Typical Fabric Items |
|-----------|---------|---------------------|
| **General** | Generic placeholder for any step | Any item |
| **Get data** | Ingest data from external sources | Pipeline, Dataflow Gen2, Copy Job, Eventstream |
| **Mirror data** | Replicate operational databases into OneLake | Mirrored Database |
| **Store data** | Persist data in OneLake | Lakehouse, Warehouse, Eventhouse, SQL Database |
| **Prepare data** | Clean, transform, and enrich data | Notebook, Dataflow Gen2, Pipeline |
| **Analyze** | Run analytical queries and explore data | Notebook, KQL Queryset, SQL Query |
| **Track** | Monitor data quality and pipeline health | Data Activator (Reflex), Monitoring |
| **Visualize** | Create reports and dashboards | Power BI Report, Real-Time Dashboard |
| **Distribute** | Share insights with consumers | Power BI App, Subscriptions |
| **Develop** | Build and test code | Notebook, Spark Job Definition, Environment |

### Creating a Task Flow

To create a task flow:

1. Open a workspace in the Fabric portal
2. Select **New** → **Task flow** (or use the task flow view toggle)
3. The canvas opens with an empty workspace view
4. **Add tasks** — drag from the task type palette onto the canvas
5. **Link items** — associate each task with an existing Fabric item, or leave it as a placeholder
6. **Draw connectors** — connect tasks to show data flow direction
7. **Arrange** — position tasks logically (left-to-right is the common pattern: ingest → transform → analyze → visualize)

### Managing Task Flows

| Action | How |
|--------|-----|
| **Edit** | Open the task flow and modify tasks, connectors, or item links |
| **Create items from tasks** | Click an unlinked task to create the corresponding Fabric item directly |
| **Navigate to items** | Click a linked task to open the associated Fabric item |
| **Delete tasks** | Remove a task from the flow (does not delete the underlying Fabric item) |
| **Share** | Task flows are shared via workspace access — anyone with workspace access can view the task flow |
| **Multiple flows** | Create separate task flows for different workflows (e.g., one for batch ETL, one for real-time streaming) |

### Task Flows vs Pipelines

| Aspect | Task Flow | Data Pipeline |
|--------|-----------|--------------|
| **Purpose** | Visual documentation and navigation | Executable data orchestration |
| **Execution** | Does not run anything | Runs activities in sequence/parallel |
| **Scheduling** | No scheduling | Supports triggers and schedules |
| **Error handling** | N/A | Retry policies, failure paths |
| **Scope** | Workspace-level overview | Specific ETL/ELT workflow |
| **Audience** | Anyone browsing the workspace | Data engineers building pipelines |

**Key insight:** Task flows and pipelines are complementary. Use task flows to document and navigate your end-to-end workflow. Use pipelines to orchestrate the actual data movement and transformation.

---

## Key Terms

| Term | Definition |
|------|-----------|
| **Task flow** | A visual, non-executable diagram organizing Fabric items into a workflow on a canvas |
| **Canvas** | The drag-and-drop surface where tasks and connectors are arranged |
| **Task** | A step in the task flow, optionally linked to a Fabric item |
| **Connector** | A visual arrow showing the flow direction between two tasks |
| **Task type** | A category indicating the role of a task in the data lifecycle (e.g., Get data, Store data, Visualize) |

---

## Further Reading

- [Task Flows Overview](https://learn.microsoft.com/en-us/fabric/fundamentals/task-flow-overview)
- [Create a Task Flow](https://learn.microsoft.com/en-us/fabric/fundamentals/task-flow-create)
- [Task Flow Task Types](https://learn.microsoft.com/en-us/fabric/fundamentals/task-flow-overview#task-types)
