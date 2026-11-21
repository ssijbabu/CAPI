# Kubernetes Platform Proposals

Proposals for provisioning Kubernetes clusters and their supporting resources on Microsoft Azure (AKS) and VMware vSphere with Tanzu (VKS).

| Document | Summary |
|---|---|
| [Requirements](Proposal/Requirements.md) | The platform requirements: clusters per team per environment, an isolated `pr`, Git as the only link between environments, SPIFFE identity for all components, no stored secrets or long-lived certificates, OCM's hub-and-managed-cluster design, and a separate control plane namespace per application on shared clusters, which may belong to different teams, and operator approval for every production deployment. |
| [Self-Service Kubernetes Platform](Proposal/Self-Service-Kubernetes-Platform-Proposal.md) | **Recommended direction.** Crossplane as the self-service API, with Cluster API (CAPZ on Azure, the Supervisor on vSphere) as the lifecycle engine, Open Cluster Management (OCM) as the fleet layer for add-ons, in-cluster policy and health, and OpenChoreo as the application platform on top. Each team has its own `dv`, `te` and `pr` clusters; `pr` runs on a fully isolated production stack, and Git is the only link between environments. |
| [Reference Deployment Design](Proposal/Reference-Deployment-Design.md) | Companion to the recommended direction: a 19-cluster reference topology (two isolated stacks, three teams), what runs in each cluster, the Git layout, network flows and sizing. |
| [Multi-Platform Kubernetes with Cluster API](Proposal/Multi-Platform-Kubernetes-with-CAPI-Proposal.md) | Cluster API only, with Argo CD rendering one request format per platform. |
| [Multi-Platform Kubernetes with Crossplane](Proposal/Multi-Platform-Kubernetes-with-Crossplane-Proposal.md) | Crossplane only, with Upjet Azure providers and provider-kubernetes for the Supervisor. |

Diagrams are SVG files in [Proposal/diagrams](Proposal/diagrams/).

## Research

| Document | Summary |
|---|---|
| [OpenChoreo: Capabilities and Use Cases](Research/OpenChoreo-Use-Cases.md) | What OpenChoreo does, 53 use cases by persona (developer, platform engineer, SRE), how each fits this platform, maturity caveats and pilot use cases. |

## Enhancements

Design documents in the style of OCM's Kubernetes Enhancement Proposals (KEPs): a `README.md` and a `metadata.yaml` per enhancement.

| Enhancement | Status | Summary |
|---|---|---|
| [SSP-0001: Self-Service Application Platform on OCM](enhancements/0001-self-service-platform-on-ocm/README.md) | Provisional | OpenChoreo's capabilities rebuilt natively on Open Cluster Management, written in Java (Quarkus, Java Operator SDK), starting with cluster provisioning (Cluster API: local clusters first, then AKS and VKS, imported into OCM automatically): SSP controllers on an OCM hub per stack, delivery through `Placement` and `ManifestWorkReplicaSet`, one hub namespace per cluster and per application, Flux for GitOps, SPIFFE identity, and Operator approval for production. Covers Platform Engineers, Application Teams, SREs and Operators. |

## Plan

| Document | Summary |
|---|---|
| [Implementation Plan](Plan/Implementation-Plan.md) | Overview of the day-by-day plan, with [one folder per week and one file per day](Plan/README.md). It plans the build of SSP-0001 with a team of four in three workstreams: cluster provisioning first (local clusters in weeks 1–3, AKS and VKS in week 13), 80 days to the alpha exit (pilot application in production), 125 days to the beta exit (three teams in production, one shared cluster), then GA by month after six months on beta. Includes milestones, the dates by which each open question must be decided, and schedule risks. |
