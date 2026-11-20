# Reference Design: Deployment Topology

**Where every component runs, for a platform of 18 clusters**

| | |
|---|---|
| **Status** | Draft |
| **Date** | 2026-11-20 |
| **Companion to** | [Self-Service-Kubernetes-Platform-Proposal.md](Self-Service-Kubernetes-Platform-Proposal.md) (the *proposal*). This document turns its architecture into a concrete deployment: cluster inventory, components per cluster, Git layout, network flows and sizing. |

---

## 1. Summary

This reference design shows the platform at a realistic starting size: **four application teams, three environments each, two isolated stacks, 18 clusters in total.**

- **6 platform clusters**, three per stack, all VKS on premises:
  - `platform`: lifecycle and fleet.
  - `choreo`: OpenChoreo control plane, portal and builds.
  - `observe`: logs, metrics and alerts.
- **12 team clusters**: each team has one cluster per environment (`dv`, `te`, `pr`). They are a mix of VKS and AKS, according to each team's needs.
- **Git** is outside both stacks and is the only link between them.

The numbers scale linearly. Each new team adds three team clusters. The six platform clusters stay the same until the number of teams or the build load needs a split (Section 10).

![Two isolated stacks below a shared Git and Entra ID; the non-production stack has three platform clusters and five VKS team clusters on premises, three AKS team clusters and a registry and key vault in Azure; the production stack has three platform clusters and two VKS team clusters on premises, two AKS team clusters and its own registry and key vault in Azure; there is no network path between the stacks](diagrams/reference-topology.svg)

## 2. Teams and Environments in This Example

| Team | `dv` | `te` | `pr` | Why |
|---|---|---|---|---|
| payments | VKS | AKS | AKS | Customer-facing on Azure; cheap on-premises development |
| orders | VKS | VKS | VKS | Data must stay on premises |
| search | AKS | AKS | AKS | Uses Azure services throughout |
| ledger | VKS | VKS | VKS | Regulated; on premises only |

All twelve team clusters have the default `purpose: openchoreo-data-plane`. A team that chooses `self-managed` keeps the same cluster count. The only difference is that its clusters run a GitOps agent instead of the OpenChoreo data plane components (Section 5.4).

## 3. Cluster Inventory

### 3.1 Non-production stack: 11 clusters

| # | Cluster | Platform | Location | Created by | Role |
|---|---|---|---|---|---|
| 1 | `np-platform` | VKS | vSphere Namespace `np-platform` | Supervisor, directly (bootstrap) | Platform control plane |
| 2 | `np-choreo` | VKS | vSphere Namespace `np-platform` | `np-platform` (`purpose: openchoreo-control-plane`) | OpenChoreo control plane, portal, workflow plane |
| 3 | `np-observe` | VKS | vSphere Namespace `np-platform` | `np-platform` (`purpose: openchoreo-observability-plane`) | OpenChoreo observability plane, platform monitoring |
| 4 | `payments-dv` | VKS | vSphere Namespace `payments-np` | `np-platform` | Team data plane |
| 5 | `payments-te` | AKS | Subscription `sub-k8s-nonprod`, `rg-payments-te` | `np-platform` | Team data plane |
| 6 | `orders-dv` | VKS | vSphere Namespace `orders-np` | `np-platform` | Team data plane |
| 7 | `orders-te` | VKS | vSphere Namespace `orders-np` | `np-platform` | Team data plane |
| 8 | `search-dv` | AKS | `sub-k8s-nonprod`, `rg-search-dv` | `np-platform` | Team data plane |
| 9 | `search-te` | AKS | `sub-k8s-nonprod`, `rg-search-te` | `np-platform` | Team data plane |
| 10 | `ledger-dv` | VKS | vSphere Namespace `ledger-np` | `np-platform` | Team data plane |
| 11 | `ledger-te` | VKS | vSphere Namespace `ledger-np` | `np-platform` | Team data plane |

### 3.2 Production stack: 7 clusters

| # | Cluster | Platform | Location | Created by | Role |
|---|---|---|---|---|---|
| 12 | `pr-platform` | VKS | vSphere Namespace `pr-platform` (production Supervisor) | Supervisor, directly (bootstrap) | Platform control plane |
| 13 | `pr-choreo` | VKS | vSphere Namespace `pr-platform` | `pr-platform` | OpenChoreo control plane, portal, workflow plane |
| 14 | `pr-observe` | VKS | vSphere Namespace `pr-platform` | `pr-platform` | OpenChoreo observability plane, platform monitoring |
| 15 | `payments-pr` | Private AKS | Subscription `sub-k8s-prod`, `rg-payments-pr` | `pr-platform` | Team data plane |
| 16 | `orders-pr` | VKS | vSphere Namespace `orders-pr` | `pr-platform` | Team data plane |
| 17 | `search-pr` | Private AKS | `sub-k8s-prod`, `rg-search-pr` | `pr-platform` | Team data plane |
| 18 | `ledger-pr` | VKS | vSphere Namespace `ledger-pr` | `pr-platform` | Team data plane |

### 3.3 Naming

| Thing | Pattern | Example |
|---|---|---|
| Platform cluster | `<stack>-<role>` (`np` or `pr`) | `np-choreo` |
| Team cluster | `<team>-<env>` | `orders-te` |
| vSphere Namespace, platform | `<stack>-platform` | `pr-platform` |
| vSphere Namespace, team | `<team>-<stack>` (holds the team's VKS clusters for that stack) | `orders-np` |
| Azure subscription | `sub-k8s-<nonprod or prod>` | `sub-k8s-prod` |
| Azure resource group | `rg-<cluster>` (one per AKS cluster, created by ASO) | `rg-search-te` |
| Control plane namespace for a team | `team-<team>` | `team-orders` |
| OpenChoreo namespace for a team | `<team>` | `orders` |

## 4. Shared Services per Stack

These run outside the clusters, with **one copy per stack**.

| Service | Non-production | Production | Used by |
|---|---|---|---|
| vSphere Supervisor | Non-production Supervisor (or shared Supervisor, Open Question 11 of the proposal) | Production Supervisor | Platform control plane (creates VKS clusters) |
| Azure subscription | `sub-k8s-nonprod` | `sub-k8s-prod` | CAPZ and ASO |
| Azure hub network | Non-production hub | Production hub, firewall egress | AKS spokes peer to it |
| Container registry | `acrk8snonprod` (private endpoint) | `acrk8sprod` (private endpoint) | Workflow plane pushes; team clusters pull |
| Secret store | `kv-k8s-nonprod` | `kv-k8s-prod` | External Secrets in every cluster of the stack |
| Workload identity | `id-k8s-nonprod`, trusting only `np-platform`'s issuer | `id-k8s-prod`, trusting only `pr-platform`'s issuer | CAPZ, ASO, External Secrets |
| Certificate authorities | OCM and OpenChoreo CAs held in `kv-k8s-nonprod` | Held in `kv-k8s-prod` | Agent mutual TLS |
| DNS zone | `np.k8s.example.org` | `k8s.example.org` | external-dns, portal and gateway endpoints |

**Shared by both stacks, on purpose.** The only things both stacks depend on are Git (Section 7), the Entra ID tenant, and corporate DNS and network core. In Entra ID, each stack has its own app registrations, groups and workload identities.

## 5. What Runs in Each Cluster

Every cluster carries the **baseline** in 5.4. The platform clusters add their own components.

### 5.1 `np-platform` and `pr-platform`: platform control plane

| Namespace | Component | Purpose |
|---|---|---|
| `capi-system`, `capz-system` | Cluster API core, CAPZ | AKS lifecycle (`aks-standard`, `aks-private` ClusterClasses) |
| `azureserviceoperator-system` | Azure Service Operator v2 | Resource groups, VNets, NSGs, identities, role assignments, Log Analytics |
| `crossplane-system` | Crossplane v2; `function-go-templating`, `function-environment-configs`, `function-auto-ready`; `provider-kubernetes` | `KubernetesCluster` and `Tenant` APIs; VKS clusters on the Supervisor |
| `open-cluster-management`, `open-cluster-management-hub` | OCM cluster manager: registration, work, placement, add-on manager; Policy framework; cluster-proxy; managed-serviceaccount | Fleet layer for every cluster in the stack |
| `kyverno` | Kyverno | Admission guardrails for requests and releases |
| `argocd` | Argo CD | Syncs Git into this cluster, and OpenChoreo configuration and releases into the `choreo` cluster |
| `team-<team>` (×4) | Team `KubernetesCluster` XRs, CAPI `Cluster` objects, ASO objects, kubeconfig secrets (platform-only) | One per team |
| `platform` | Platform tenant: `KubernetesCluster` for `choreo` and `observe` | Platform-owned clusters |

The platform cluster registers itself with its own OCM hub, so it also receives the baseline.

### 5.2 `np-choreo` and `pr-choreo`: OpenChoreo control plane and workflow plane

| Namespace | Component | Purpose |
|---|---|---|
| `openchoreo-control-plane` | `openchoreo-control-plane` chart: controller manager, OpenChoreo API, cluster gateway, event forwarder | OpenChoreo API and controllers; the gateway that all plane agents connect to |
| `openchoreo-control-plane` | Backstage portal (part of the same chart) | Developer portal; also hosts the `KubernetesCluster` template and the OCM plugin (proposal Section 9.4) |
| `openchoreo-workflow-plane`, `argo` | `openchoreo-workflow-plane` chart: Argo Workflows and a cluster agent | Builds. In production, it rebuilds images from the commit in the release (proposal Section 9.3). |
| `kgateway-system` | Gateway API CRDs and kgateway | Ingress for the API, portal and cluster gateway |
| `<team>` (×4) | OpenChoreo namespaces: Projects, Components, Environments, the team's `DataPlane`s, releases and bindings | Delivered by OCM and Argo CD from Git |

OpenChoreo is configured to sign users in through **Entra ID**, instead of the identity provider it bundles. The workflow plane holds the only push credential for the stack's registry.

### 5.3 `np-observe` and `pr-observe`: observability

| Namespace | Component | Purpose |
|---|---|---|
| `openchoreo-observability-plane` | `openchoreo-observability-plane` chart: observer, OpenTelemetry Collector, Prometheus, OpenSearch, and a cluster agent | Application logs, metrics, traces and alerts, shown in OpenChoreo |
| `platform-monitoring` | Prometheus (remote-write receiver), Alertmanager, Grafana | Platform metrics: Crossplane, CAPI, CAPZ, ASO, OCM, Kyverno, OpenChoreo and cluster health; the alerts in proposal Section 12.5 |

OpenSearch and Prometheus use persistent volumes on vSAN. This is the heaviest platform cluster for storage.

### 5.4 Every cluster: the baseline (delivered by OCM)

| Namespace | Component | Purpose |
|---|---|---|
| `open-cluster-management-agent`, `-addon` | OCM klusterlet and add-on agents (policy, cluster-proxy, managed-serviceaccount) | Registration, add-ons, policy, break-glass; outbound to the stack's hub |
| `cert-manager` | cert-manager | Certificates, including the OpenChoreo agent's client certificate |
| `external-secrets` | External Secrets, plus a `ClusterSecretStore` named `default` pointing at the stack's Key Vault | Secrets; OpenChoreo expects the store name `default` |
| `external-dns` | external-dns | DNS records in the stack's zone |
| `platform-monitoring` | OpenTelemetry Collector (metrics and logs) | Ships to the stack's `observe` cluster |
| `kyverno` | Kyverno with fleet policies (Pod Security, required labels) | In-cluster baseline, reported by OCM Policy |

**Additionally, on team clusters with `purpose: openchoreo-data-plane`:**

| Namespace | Component | Purpose |
|---|---|---|
| `kgateway-system` | Gateway API CRDs and kgateway | The data plane's ingress gateway |
| `openchoreo-data-plane` | `openchoreo-data-plane` chart: cluster agent and Gateway | Agent connects out to the stack's `choreo` gateway; applies releases |
| `dp-*` (created by OpenChoreo) | The team's applications | One namespace per project and environment |

**On `self-managed` team clusters instead:** a GitOps agent (Argo CD or Flux) and an ingress controller, and no OpenChoreo components.

**Platform-native extras:**

| AKS | VKS |
|---|---|
| Entra ID with Azure RBAC; Workload Identity; Azure Policy add-on; Defender for Containers; Container Insights | vSphere CSI; Antrea or Calico; VKS-managed packages as published by the Supervisor |

## 6. Tool Placement Matrix

| Tool | `platform` | `choreo` | `observe` | Team clusters | Outside clusters |
|---|:-:|:-:|:-:|:-:|---|
| Cluster API core, CAPZ | ● | | | | |
| Azure Service Operator | ● | | | | |
| Supervisor CAPI and VKS | | | | | vSphere (VMware-owned) |
| Crossplane and functions, provider-kubernetes | ● | | | | |
| OCM hub (cluster manager, Policy, cluster-proxy) | ● | | | | |
| OCM klusterlet and add-on agents | ● | ● | ● | ● | |
| Kyverno, for requests and releases | ● | | | | |
| Kyverno, for the in-cluster baseline | ● | ● | ● | ● | |
| Argo CD | ● | | | | |
| OpenChoreo control plane and portal | | ● | | | |
| OpenChoreo workflow plane (Argo Workflows) | | ● | | | |
| OpenChoreo observability plane | | | ● | | |
| OpenChoreo data plane (agent, kgateway) | | | | ● (default purpose) | |
| Platform monitoring (Prometheus, Alertmanager, Grafana) | | | ● | | |
| cert-manager, External Secrets, external-dns, OTel Collector | ● | ● | ● | ● | |
| Git hosting and pull request checks | | | | | Git (Section 7) |
| Container registry, Key Vault | | | | | Azure, one per stack |
| Entra ID | | | | | Shared tenant |

## 7. Git

Git is the only thing both stacks read, and the only path from non-production to production. It is hosted outside both stacks, in one GitHub organisation (GitHub Enterprise).

### 7.1 Repositories

| Repository | Owner | Contents | Read by | Written by |
|---|---|---|---|---|
| `platform` | Platform team | XRDs, Compositions and render tests, ClusterClasses, Kyverno policies, OCM add-ons and fleet policies, OpenChoreo component types, traits and workflows | Both stacks' Argo CD, **by tag only** | Platform engineers, by pull request; released as `vX.Y.Z` |
| `platform-nonprod` | Platform team | Non-production stack: platform tag in use, bootstrap values, `platform-defaults`, tenants, platform tenant's clusters | `np-platform` Argo CD | Platform engineers |
| `platform-pr` | Platform team (production approvers) | The same, for production | `pr-platform` Argo CD | Platform engineers, with production approval |
| `team-payments`, `team-orders`, `team-search`, `team-ledger` | Each team | `nonprod/clusters/`, `nonprod/releases/te/`, `pr/clusters/`, `pr/releases/` | `np-platform` Argo CD reads `nonprod/`; `pr-platform` Argo CD reads `pr/` | Team members; release bots (7.3) |
| Application source (for example `payments-api`) | Each team | Code and Dockerfile or buildpack settings | Both stacks' workflow planes (clone at a commit) | Team members |

### 7.2 Protection rules

| Repository or path | Rules |
|---|---|
| `platform` | Pull requests only; required checks; two platform reviewers; signed commits; release tags protected |
| `platform-nonprod` | Pull requests only; required checks; one platform reviewer |
| `platform-pr` | Pull requests only; required checks; production approver group (CODEOWNERS); signed commits |
| `team-*/nonprod/` | Pull requests only; required checks; one team reviewer |
| `team-*/pr/` | Pull requests only; required checks, including the promotion check; CODEOWNERS = team leads; signed commits |
| All | No force pushes; no direct pushes to `main`; merge queue; branch deletion protected |

If one repository with a protected `pr/` folder is not enough for audit, split `pr/` into `team-<team>-pr`. The rest of this design does not change (proposal Open Question 13).

### 7.3 Machine identities in Git

Every machine identity is a GitHub App with the smallest permissions that work, one per stack.

| Identity | Stack | Permissions | Used for |
|---|---|---|---|
| `np-argocd` | Non-production | Read: `platform`, `platform-nonprod`, `team-*` | Argo CD sync. AppProjects limit each team to `nonprod/`. |
| `pr-argocd` | Production | Read: `platform`, `platform-pr`, `team-*` | Argo CD sync. AppProjects limit each team to `pr/`. |
| `np-release-bot` | Non-production | Read: application source; write: pull requests on `team-*` | Exports a `ComponentRelease` and opens the `te` and `pr` promotion pull requests |
| `pr-build-bot` | Production | Read: application source; write: pull request branches on `team-*` | Adds the production image digest to an open `pr` promotion pull request |
| `ci-checks` | Neither | Read-only | Pull request checks on hosted runners; holds no stack or cloud credentials |

GitHub App permissions apply to whole repositories, not folders. The folder boundaries are enforced by Argo CD AppProjects, by CODEOWNERS, and by Kyverno in each stack (which accepts only that stack's environments).

### 7.4 Pull request checks (hosted runners, no stack access)

| Check | Repositories | What it does |
|---|---|---|
| Schema | all | `kubeconform` against the XRD and OpenChoreo CRD schemas |
| Policy | all | `kyverno apply` with the policies from the `platform` tag the stack uses |
| Render | `platform`, `platform-*` | `crossplane render` and `crossplane beta validate`; `clusterctl alpha topology plan` |
| Quota | `team-*` | Counts clusters per environment against the tenant's quota |
| Promotion | `team-*/pr/releases/` | The same `ComponentRelease` must be bound to `te` in `nonprod/releases/te/`, for at least the soak time |
| Digest present | `team-*/pr/releases/` | The release must carry a production-registry digest added by `pr-build-bot` |

### 7.5 Triggers into the stacks

- **Argo CD polls** Git, so Git never needs to reach into a stack to keep it in sync.
- **Builds** start from a Git webhook to the stack's OpenChoreo API, through an endpoint that only accepts GitHub's published webhook ranges. If inbound webhooks are not allowed, the workflow plane polls instead.
- **Nothing in one stack ever reads or writes the other stack.** The only way something from non-production can reach production is as a commit in Git.

## 8. Network Flows

All flows stay within one stack. Every flow between the stacks is denied.

| From | To | Port | Direction | Why |
|---|---|---|---|---|
| `platform` | Supervisor API | 443 | On premises | Create and update VKS clusters (`provider-kubernetes`) |
| `platform` | Azure Resource Manager, Entra ID | 443 | Outbound to internet or private link | CAPZ, ASO |
| `platform` | AKS API servers in the stack | 443 | Over hybrid connectivity to private endpoints | CAPI reads node state; one-time OCM import |
| `platform` | VKS API servers in the stack | 6443 | On premises | One-time OCM import |
| `platform` (Argo CD) | `choreo` API server | 6443 | On premises | Apply OpenChoreo configuration and releases from Git |
| `platform`, `choreo` | Git | 443 | Outbound | Argo CD sync; clone source |
| Every cluster | `platform` hub API (OCM) | 443 | Outbound from the cluster; AKS reaches on premises over hybrid connectivity | Klusterlet and add-on agents |
| Team clusters, `observe` | `choreo` cluster gateway | 443 (WSS) | Outbound from the cluster | OpenChoreo cluster agents |
| Every cluster | `observe` | 4317 (OTLP), 443 | Outbound | Logs and metrics |
| Every cluster | Key Vault (private endpoint) | 443 | Outbound | External Secrets |
| `choreo` (workflow plane) | Registry | 443 | Outbound | Push built images |
| Team clusters | Registry (private endpoint) | 443 | Outbound | Pull images |
| GitHub | `choreo` webhook endpoint | 443 | Inbound, GitHub ranges only (optional) | Build triggers |
| People | Portal, AKS API (Entra ID), VKS API (vSphere SSO) | 443, 6443 | Inbound from corporate network | Use and break-glass |
| **Either stack** | **The other stack** | **any** | **Denied** | Isolation |

**Dependency to plan for.** The production stack's AKS clusters must be able to reach `pr-platform` and `pr-choreo` on premises. In the other direction, `pr-platform` must reach the private AKS API servers. Both paths use the production hybrid connection, so it must be resilient (redundant ExpressRoute or VPN). Running workloads are not affected if it fails, but changes and promotions stop.

## 9. Sizing (Starting Point)

VM classes and Azure sizes follow the proposal's size mapping. Revisit them after the pilot.

| Cluster | Control plane | Workers | Storage |
|---|---|---|---|
| `np-platform`, `pr-platform` | 3 × `best-effort-large` | 3 × `best-effort-large` | 50 GiB per node |
| `np-choreo` | 3 × `best-effort-large` | 4 × `best-effort-xlarge` (builds burst here) | 200 GiB build cache |
| `pr-choreo` | 3 × `best-effort-large` | 3 × `best-effort-xlarge` | 200 GiB build cache |
| `np-observe`, `pr-observe` | 3 × `best-effort-large` | 3 × `best-effort-2xlarge` | 1–2 TiB on vSAN (OpenSearch, Prometheus) |
| `<team>-dv` (VKS) | 1 × `best-effort-medium` | 2 × `best-effort-large` | Team requests |
| `<team>-dv` (AKS) | Free tier | System pool 1 × D2ds_v5; apps 2 × D4ds_v5, autoscale to 0 overnight | Team requests |
| `<team>-te` | 3 control-plane VMs (VKS) or Standard tier (AKS) | 3 × medium | Team requests |
| `<team>-pr` | 3 control-plane VMs (VKS) or Standard tier, 3 zones (AKS) | 3 × large minimum, across zones | Team requests |

**Totals for this example:**

- VKS: 13 clusters, 33 control-plane VMs and 37 worker VMs. Production uses only the production vSphere capacity.
- AKS: 5 clusters, about 23 nodes, counting system pools (three nodes, across zones, in production).

## 10. Bootstrap Order

For each stack, independently. Production is built from a `platform` tag that is already running in non-production.

1. **`<stack>-platform`**: apply a `Cluster` to the Supervisor; install the components in Section 5.1 at pinned versions; configure the stack's identity, Supervisor tokens and Key Vault access; point Argo CD at the `platform` tag and `platform-<stack>`.
2. **`<stack>-choreo` and `<stack>-observe`**: created by the platform tenant's `KubernetesCluster` objects. OCM installs the baseline and the OpenChoreo charts. The observability plane and the workflow plane register with the OpenChoreo control plane, just as data planes do.
3. **Tenants**: one `Tenant` per team creates its control plane namespace, quota, Argo CD application, `ManagedClusterSet`, OpenChoreo namespace and environments.
4. **Team clusters**: synced from each team repository's folder for that stack; registered as `DataPlane`s; `Ready` when both agents are connected.

## 11. Growing Beyond This Size

| Signal | Change |
|---|---|
| More than about 25 teams per stack | Split the stack's OCM hub and Argo CD onto their own cluster, or shard Argo CD |
| Build queue time keeps growing | Move the workflow plane from `choreo` to its own `<stack>-build` cluster |
| More than about 30 days of logs, or high cardinality | Scale `observe` out, or send platform metrics to the organisation's central monitoring |
| A second Azure region | Add AKS team clusters in that region from the same stack; a separate stack is not needed |
| A team needs several clusters per environment | Raise its quota; names become `<team>-<env>-<n>` |

## 12. References

- [Self-Service Kubernetes Platform proposal](Self-Service-Kubernetes-Platform-Proposal.md): Sections 5.4 (isolation), 8.4 (OCM), 9 (OpenChoreo) and 12 (operating model)
- OpenChoreo Helm charts: `openchoreo-control-plane`, `openchoreo-data-plane`, `openchoreo-workflow-plane`, `openchoreo-observability-plane`
- Open Cluster Management: https://open-cluster-management.io
- Cluster API Provider Azure: https://capz.sigs.k8s.io
