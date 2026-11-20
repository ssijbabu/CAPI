# Kubernetes Platform Proposals

Proposals for provisioning Kubernetes clusters and their supporting resources on Microsoft Azure (AKS) and VMware vSphere with Tanzu (VKS).

| Document | Summary |
|---|---|
| [Self-Service Kubernetes Platform](Proposal/Self-Service-Kubernetes-Platform-Proposal.md) | **Recommended direction.** Crossplane as the self-service API, with Cluster API (CAPZ on Azure, the Supervisor on vSphere) as the lifecycle engine, Open Cluster Management (OCM) as the fleet layer for add-ons, in-cluster policy and health, and OpenChoreo as the application platform on top. Each team has its own `dv`, `te` and `pr` clusters; `pr` runs on a fully isolated production stack, and Git is the only link between environments. |
| [Multi-Platform Kubernetes with Cluster API](Proposal/Multi-Platform-Kubernetes-with-CAPI-Proposal.md) | Cluster API only, with Argo CD rendering one request format per platform. |
| [Multi-Platform Kubernetes with Crossplane](Proposal/Multi-Platform-Kubernetes-with-Crossplane-Proposal.md) | Crossplane only, with Upjet Azure providers and provider-kubernetes for the Supervisor. |

Diagrams are SVG files in [Proposal/diagrams](Proposal/diagrams/).
