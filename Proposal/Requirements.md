# Requirements: Self-Service Kubernetes Platform

| ID | Requirement |
|---|---|
| **SR-1** | Each team must have its own clusters for each environment: `dv` (development), `te` (test) and `pr` (production). |
| **SR-2** | `pr` must be completely isolated at every level, and must use its own control plane. |
| **SR-3** | Git must be the only link between environments. Promotion to `pr` must go through Git. |
| **SR-4** | All components must use SPIFFE for workload identity. |
| **SR-5** | There must be no stored secrets and no long-lived certificates. |
| **SR-6** | The platform must use OCM's architecture and internal design: a hub with managed clusters, where each managed cluster has its own namespace on the hub. |
| **SR-7** | Some clusters will host several applications. Each application must still have its own namespace in the control plane, even when it shares a cluster with others. |
| **SR-8** | Applications on a shared cluster can belong to different teams. |
| **SR-9** | Every production deployment must be approved by the operator: a new role, held by the team that manages the production environment. |
