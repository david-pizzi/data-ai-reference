# Workspaces in Microsoft Fabric

> **Source:** [Workspaces in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/fundamentals/workspaces)
> **Related pages:** [Fundamentals Overview](../fundamentals.md) | [Core Concepts](concepts.md)

---

## Learning Objectives

- Explain what workspaces are and how they organize Fabric items.
- Describe the four workspace roles and their permissions.
- Understand workspace layout, settings, and configuration options.
- Describe monitoring, administration, and auditing capabilities.
- Explain how the recycle bin protects against accidental deletion.

---

## Key Concepts

### What Are Workspaces?

A **workspace** is the fundamental organizational container in Microsoft Fabric. It is where teams collaborate on Fabric items — lakehouses, warehouses, notebooks, pipelines, reports, semantic models, and more.

Key characteristics:

- Each workspace maps to a **folder in OneLake** — all items in the workspace share the same underlying storage namespace
- Workspaces are assigned to a **capacity** (F-SKU or P-SKU), which determines the compute resources available
- Access is controlled through **workspace roles** — each role grants a specific set of permissions
- Workspaces can be organized by team, project, environment (dev/test/prod), or domain
- Every Fabric tenant has a default "My Workspace" for personal exploration (not shareable)

### Workspace Roles

Fabric uses four roles to control access within a workspace. Roles are **cumulative** — each higher role includes all permissions of the roles below it.

| Permission | Admin | Member | Contributor | Viewer |
|-----------|:-----:|:------:|:-----------:|:------:|
| View and read items | Yes | Yes | Yes | Yes |
| Connect to SQL analytics endpoint / Warehouse | Yes | Yes | Yes | Yes |
| View Power BI reports and dashboards | Yes | Yes | Yes | Yes |
| Create, edit, and delete items | Yes | Yes | Yes | No |
| Publish and update apps | Yes | Yes | No | No |
| Share items and grant access | Yes | Yes | No | No |
| Manage workspace settings | Yes | Yes | No | No |
| Add/remove Members, Contributors, Viewers | Yes | Yes | No | No |
| Add/remove Admins | Yes | No | No | No |
| Manage capacity assignment | Yes | No | No | No |
| Manage Git integration | Yes | No | No | No |
| Delete the workspace | Yes | No | No | No |

**Role assignment guidance:**

- **Admin** — workspace owners, platform administrators
- **Member** — team leads, senior engineers who need to manage access
- **Contributor** — data engineers, analysts, developers who build and modify items
- **Viewer** — business users, stakeholders who consume reports and query data

### Workspace Layout

When you open a workspace, you see a list of all items organized by type. The workspace experience includes:

- **Item list** — all Fabric items (lakehouses, warehouses, notebooks, pipelines, reports, etc.) with metadata such as owner, last modified date, and endorsement status
- **Filters and search** — filter by item type, keyword search across item names and descriptions
- **Lineage view** — visualize data flow and dependencies between items in the workspace
- **Task flows** — visual canvas showing how items connect in an end-to-end workflow
- **Monitoring hub** — view running and recent activities (Spark jobs, pipeline runs, refresh operations)

### Workspace Settings

Workspace settings control behavior, governance, and integration for the workspace.

| Setting Area | What It Controls |
|-------------|-----------------|
| **General** | Workspace name, description, contact list, default storage format |
| **Capacity** | Which Fabric capacity the workspace is assigned to (determines available compute) |
| **License** | License mode — Pro, Premium Per User (PPU), or Fabric capacity |
| **Git integration** | Connect to Azure DevOps or GitHub for version control of workspace items |
| **Spark settings** | Default Spark pool configuration, environment, and library management |
| **OneLake** | OneLake data access settings, ADLS Gen2 API access |
| **Domain** | Assign the workspace to an organizational domain for governance grouping |

### Monitoring

The **Monitoring Hub** provides visibility into all background activities running in the workspace:

- **Spark jobs** — notebook runs, Spark job definitions, lakehouse operations
- **Pipeline runs** — Data Factory pipeline execution status, duration, errors
- **Dataflow refreshes** — Dataflow Gen2 refresh status and history
- **SQL queries** — long-running warehouse queries
- **Copy Jobs** — bulk data movement status

Each activity shows start time, duration, status (running / succeeded / failed), and the user who triggered it.

### Administration

Fabric administrators manage workspaces at the tenant level through the **Admin Portal**:

- **Tenant settings** — enable/disable features across the organization (Copilot, Git integration, external sharing, etc.)
- **Capacity management** — assign workspaces to capacities, monitor utilization, configure autoscale
- **Usage metrics** — track workspace activity, item usage, and user engagement
- **Audit logs** — detailed event logs for compliance, forwarded to Microsoft Purview or Log Analytics

### Auditing

All workspace activity generates **audit events** captured in the Microsoft 365 unified audit log:

- Item creation, modification, deletion
- Role assignments and access changes
- Data access and query execution
- Pipeline and Spark job execution
- Sharing and permission changes

Audit logs can be:

- Viewed in the Microsoft Purview compliance portal
- Exported to Azure Log Analytics for advanced analysis
- Used with Microsoft Sentinel for security monitoring

### Recycle Bin

When items are deleted from a workspace, they are moved to the **recycle bin** rather than permanently destroyed.

- Deleted items are retained for **7 days** before permanent deletion
- Workspace Admins and Members can restore items from the recycle bin
- Restoring an item returns it to the workspace with its original configuration
- After 7 days, items are permanently deleted and cannot be recovered
- The recycle bin is accessible from the workspace settings menu

---

## Key Terms

| Term | Definition |
|------|-----------|
| **Workspace** | A logical container for Fabric items, mapped to a folder in OneLake, with role-based access control |
| **Capacity** | A pool of compute resources (CUs) assigned to one or more workspaces |
| **Workspace role** | One of four access levels (Admin, Member, Contributor, Viewer) controlling permissions within a workspace |
| **Lineage view** | A visual representation of data flow and dependencies between items in a workspace |
| **Monitoring Hub** | A workspace-level dashboard showing running and recent activities |
| **Audit log** | A detailed event log capturing all workspace activity for compliance and security |
| **Recycle bin** | A 7-day retention area for deleted workspace items, allowing recovery before permanent deletion |
| **Domain** | An organizational grouping applied to workspaces for governance and discoverability |

---

## Further Reading

- [Workspaces in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/fundamentals/workspaces)
- [Workspace Roles](https://learn.microsoft.com/en-us/fabric/fundamentals/roles-workspaces)
- [Fabric Admin Portal](https://learn.microsoft.com/en-us/fabric/admin/admin-center)
- [Monitoring Hub](https://learn.microsoft.com/en-us/fabric/admin/monitoring-hub)
- [Audit Logs in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/admin/operation-list)
