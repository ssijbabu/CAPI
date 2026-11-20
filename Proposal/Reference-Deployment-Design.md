# Reference Design: Deployment Topology

**Where every component runs, for a platform of 22 clusters**

| | |
|---|---|
| **Status** | Draft |
| **Date** | 2026-11-20 |
| **Companion to** | [Self-Service-Kubernetes-Platform-Proposal.md](Self-Service-Kubernetes-Platform-Proposal.md) (the *proposal*). This document turns its architecture into a concrete deployment: cluster inventory, components per cluster, Git layout, network flows and sizing. |

---

## 1. Summary

This reference design shows the platform at a realistic starting size: **four application teams, three environments each, two isolated stacks, 22 clusters in total.**

- **10 platform clusters**, five per stack, all VKS on premises, each with one job:
  - `platform`: the self-service control plane (Crossplane, Cluster API, CAPZ, ASO, Argo CD, Kyverno).
  - `fleet`: the OCM hub.
  - `choreo`: the OpenChoreo control plane and portal.
  - `build`: the OpenChoreo workflow plane.
  - `observe`: logs, metrics and alerts.
- **12 team clusters**: each team has one cluster per environment (`dv`, `te`, `pr`). They are a mix of VKS and AKS, according to each team's needs.
- **Git** is outside both stacks and is the only link between them.

Keeping each platform role on its own cluster means each one can be upgraded, scaled and recovered without touching the others. For example, a heavy build cannot slow the OpenChoreo API, and an OCM hub upgrade cannot interrupt cluster provisioning.

The numbers scale linearly. Each new team adds three team clusters, and the ten platform clusters stay the same (Section 11).

![Reference topology on an A4 landscape page: Git (repositories, GitHub Apps, pull request checks), Entra ID and people across the top; below, the non-production and production stacks separated by a no-network-path barrier, each showing its platform, fleet, choreo, build and observe clusters and every team cluster with its components, plus on-premises and Azure services; the connections inside each stack are listed at the bottom](diagrams/reference-topology.svg)

*The diagram is drawn for an A4 landscape page. Open the SVG on its own to zoom in.*

## 2. Teams and Environments in This Example

| Team | `dv` | `te` | `pr` | Why |
|---|---|---|---|---|
| payments | VKS | AKS | AKS | Customer-facing on Azure; cheap on-premises development |
| orders | VKS | VKS | VKS | Data must stay on premises |
| search | AKS | AKS | AKS | Uses Azure services throughout |
| ledger | VKS | VKS | VKS | Regulated; on premises only |

All twelve team clusters have the default `purpose: openchoreo-data-plane`. A team that chooses `self-managed` keeps the same cluster count. The only difference is that its clusters run a GitOps agent instead of the OpenChoreo data plane components (Section 5.6).

## 3. Cluster Inventory

### 3.1 Non-production stack: 13 clusters

| # | Cluster | Platform | Location | Created by | Role |
|---|---|---|---|---|---|
| 1 | `np-platform` | VKS | vSphere Namespace `np-platform` | Supervisor, directly (bootstrap) | Self-service control plane |
| 2 | `np-fleet` | VKS | vSphere Namespace `np-platform` | `np-platform` (`purpose: platform-fleet`) | OCM hub |
| 3 | `np-choreo` | VKS | vSphere Namespace `np-platform` | `np-platform` (`purpose: openchoreo-control-plane`) | OpenChoreo control plane and portal |
| 4 | `np-build` | VKS | vSphere Namespace `np-platform` | `np-platform` (`purpose: openchoreo-workflow-plane`) | OpenChoreo workflow plane (builds) |
| 5 | `np-observe` | VKS | vSphere Namespace `np-platform` | `np-platform` (`purpose: openchoreo-observability-plane`) | OpenChoreo observability plane, platform monitoring |
| 6 | `payments-dv` | VKS | vSphere Namespace `payments-np` | `np-platform` | Team data plane |
| 7 | `payments-te` | AKS | Subscription `sub-k8s-nonprod`, `rg-payments-te` | `np-platform` | Team data plane |
| 8 | `orders-dv` | VKS | vSphere Namespace `orders-np` | `np-platform` | Team data plane |
| 9 | `orders-te` | VKS | vSphere Namespace `orders-np` | `np-platform` | Team data plane |
| 10 | `search-dv` | AKS | `sub-k8s-nonprod`, `rg-search-dv` | `np-platform` | Team data plane |
| 11 | `search-te` | AKS | `sub-k8s-nonprod`, `rg-search-te` | `np-platform` | Team data plane |
| 12 | `ledger-dv` | VKS | vSphere Namespace `ledger-np` | `np-platform` | Team data plane |
| 13 | `ledger-te` | VKS | vSphere Namespace `ledger-np` | `np-platform` | Team data plane |

### 3.2 Production stack: 9 clusters

| # | Cluster | Platform | Location | Created by | Role |
|---|---|---|---|---|---|
| 14 | `pr-platform` | VKS | vSphere Namespace `pr-platform` (production Supervisor) | Supervisor, directly (bootstrap) | Self-service control plane |
| 15 | `pr-fleet` | VKS | vSphere Namespace `pr-platform` | `pr-platform` | OCM hub |
| 16 | `pr-choreo` | VKS | vSphere Namespace `pr-platform` | `pr-platform` | OpenChoreo control plane and portal |
| 17 | `pr-build` | VKS | vSphere Namespace `pr-platform` | `pr-platform` | OpenChoreo workflow plane (production rebuilds) |
| 18 | `pr-observe` | VKS | vSphere Namespace `pr-platform` | `pr-platform` | OpenChoreo observability plane, platform monitoring |
| 19 | `payments-pr` | Private AKS | Subscription `sub-k8s-prod`, `rg-payments-pr` | `pr-platform` | Team data plane |
| 20 | `orders-pr` | VKS | vSphere Namespace `orders-pr` | `pr-platform` | Team data plane |
| 21 | `search-pr` | Private AKS | `sub-k8s-prod`, `rg-search-pr` | `pr-platform` | Team data plane |
| 22 | `ledger-pr` | VKS | vSphere Namespace `ledger-pr` | `pr-platform` | Team data plane |

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

Every cluster carries the **baseline** in 5.6. The platform clusters add their own components.

### 5.1 `np-platform` and `pr-platform`: self-service control plane

| Namespace | Component | Purpose |
|---|---|---|
| `crossplane-system` | Crossplane v2; `function-go-templating`, `function-environment-configs`, `function-auto-ready`; `provider-kubernetes` | `KubernetesCluster` and `Tenant` APIs. `provider-kubernetes` writes VKS clusters to the Supervisor, and OCM objects (`ManagedCluster`, `ManagedClusterAddOn`, `ManifestWork`) to the `fleet` cluster. |
| `capi-system`, `capz-system` | Cluster API core, CAPZ | AKS lifecycle (`aks-standard`, `aks-private` ClusterClasses) |
| `azureserviceoperator-system` | Azure Service Operator v2 | Resource groups, VNets, NSGs, identities, role assignments, Log Analytics |
| `kyverno` | Kyverno | Admission guardrails for requests and releases |
| `argocd` | Argo CD | Syncs Git into this cluster; installs and configures the OCM hub on `fleet`; applies OpenChoreo configuration and releases to `choreo` |
| `team-<team>` (×4) | Team `KubernetesCluster` XRs, CAPI `Cluster` objects, ASO objects, kubeconfig secrets (platform-only) | One per team |
| `platform` | Platform tenant: `KubernetesCluster` for `fleet`, `choreo`, `build` and `observe` | Platform-owned clusters |

### 5.2 `np-fleet` and `pr-fleet`: OCM hub

| Namespace | Component | Purpose |
|---|---|---|
| `open-cluster-management`, `open-cluster-management-hub` | OCM cluster manager: registration (with auto-import), work, placement, add-on manager | Registers every cluster in the stack; delivers add-ons and `ManifestWork`s; feeds status back |
| `open-cluster-management-hub` | Policy framework (propagator), cluster-proxy, managed-serviceaccount | Fleet policy and compliance; audited break-glass with short-lived credentials |
| `<cluster>` (one per managed cluster) | `ManagedCluster` namespace: add-on configs, `ManifestWork`s, import secret | Created by Crossplane from `np-platform` / `pr-platform` |
| `platform-addons` | `ClusterManagementAddOn`s, `Placement`s, add-on configurations | Baseline and OpenChoreo charts, targeted by platform, environment and plane |

The hub manages itself as a cluster, so it receives the baseline too. It holds no cloud credentials; the import secret it uses is copied in by Crossplane for the one-time import and then deleted.

### 5.3 `np-choreo` and `pr-choreo`: OpenChoreo control plane

| Namespace | Component | Purpose |
|---|---|---|
| `openchoreo-control-plane` | `openchoreo-control-plane` chart: controller manager, OpenChoreo API, cluster gateway, event forwarder | OpenChoreo API and controllers; the gateway that every plane agent connects to |
| `openchoreo-control-plane` | Backstage portal (part of the same chart) | Developer portal; also hosts the `KubernetesCluster` template and the OCM plugin (proposal Section 9.4) |
| `kgateway-system` | Gateway API CRDs and kgateway | Ingress for the API, portal and cluster gateway |
| `<team>` (×4) | OpenChoreo namespaces: Projects, Components, Environments, the team's `DataPlane`s, releases and bindings | Delivered by OCM and Argo CD from Git |

OpenChoreo is configured to sign users in through **Entra ID**, instead of the identity provider it bundles. The `WorkflowPlane` and `ObservabilityPlane` registrations for `build` and `observe` live here.

### 5.4 `np-build` and `pr-build`: OpenChoreo workflow plane

| Namespace | Component | Purpose |
|---|---|---|
| `openchoreo-workflow-plane`, `argo` | `openchoreo-workflow-plane` chart: Argo Workflows and a cluster agent | Runs builds; the agent connects out to the `choreo` cluster gateway |
| `openchoreo-workflow-plane` | Build cache (PVC on vSAN), Git read access, registry push secret | The only push credential for the stack's registry |
| `openchoreo-workflow-plane` | Image signing and build provenance | Every image is signed and attested to its commit |

In non-production, `np-build` builds from source and the release bot (`np-release-bot`) exports releases to Git. In production, `pr-build` rebuilds the commit named in a promotion pull request, and `pr-build-bot` adds the production digest to that pull request (proposal Section 9.3).

### 5.5 `np-observe` and `pr-observe`: observability

| Namespace | Component | Purpose |
|---|---|---|
| `openchoreo-observability-plane` | `openchoreo-observability-plane` chart: observer, OpenTelemetry Collector, Prometheus, OpenSearch, and a cluster agent | Application logs, metrics, traces and alerts, shown in OpenChoreo |
| `platform-monitoring` | Prometheus (remote-write receiver), Alertmanager, Grafana | Platform metrics: Crossplane, CAPI, CAPZ, ASO, OCM, Kyverno, OpenChoreo and cluster health; the alerts in proposal Section 12.5 |

OpenSearch and Prometheus use persistent volumes on vSAN. This is the heaviest platform cluster for storage.

### 5.6 Every cluster: the baseline (delivered by OCM)

| Namespace | Component | Purpose |
|---|---|---|
| `open-cluster-management-agent`, `-addon` | OCM klusterlet and add-on agents (policy, cluster-proxy, managed-serviceaccount) | Registration, add-ons, policy, break-glass; outbound to the stack's `fleet` hub |
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

| Tool | `platform` | `fleet` | `choreo` | `build` | `observe` | Team clusters | Outside clusters |
|---|:-:|:-:|:-:|:-:|:-:|:-:|---|
| Crossplane and functions, provider-kubernetes | ● | | | | | | |
| Cluster API core, CAPZ | ● | | | | | | |
| Azure Service Operator | ● | | | | | | |
| Supervisor CAPI and VKS | | | | | | | vSphere (VMware-owned) |
| Argo CD | ● | | | | | | |
| Kyverno, for requests and releases | ● | | | | | | |
| OCM hub (cluster manager, add-on manager, Policy, cluster-proxy, managed-serviceaccount) | | ● | | | | | |
| OCM klusterlet and add-on agents | ● | ● | ● | ● | ● | ● | |
| OpenChoreo control plane and portal | | | ● | | | | |
| OpenChoreo workflow plane (Argo Workflows) | | | | ● | | | |
| OpenChoreo observability plane | | | | | ● | | |
| OpenChoreo data plane (agent, kgateway) | | | | | | ● (default purpose) | |
| Platform monitoring (Prometheus, Alertmanager, Grafana) | | | | | ● | | |
| Kyverno, for the in-cluster baseline | ● | ● | ● | ● | ● | ● | |
| cert-manager, External Secrets, external-dns, OTel Collector | ● | ● | ● | ● | ● | ● | |
| Git hosting and pull request checks | | | | | | | Git (Section 7) |
| Container registry, Key Vault | | | | | | | Azure, one per stack |
| Entra ID | | | | | | | Shared tenant |

## 7. Git

Git is the only thing both stacks read, and the only path from non-production to production. It is hosted outside both stacks, in one GitHub organisation (GitHub Enterprise).

### 7.1 Repositories

| Repository | Owner | Contents | Read by | Written by |
|---|---|---|---|---|
| `platform` | Platform team | XRDs, Compositions and render tests, ClusterClasses, Kyverno policies, OCM add-ons and fleet policies, OpenChoreo component types, traits and workflows | Both stacks' Argo CD, **by tag only** | Platform engineers, by pull request; released as `vX.Y.Z` |
| `platform-nonprod` | Platform team | Non-production stack: platform tag in use, bootstrap values, `platform-defaults`, tenants, platform tenant's clusters | `np-platform` Argo CD | Platform engineers |
| `platform-pr` | Platform team (production approvers) | The same, for production | `pr-platform` Argo CD | Platform engineers, with production approval |
| `team-payments`, `team-orders`, `team-search`, `team-ledger` | Each team | `nonprod/clusters/`, `nonprod/releases/te/`, `pr/clusters/`, `pr/releases/` | `np-platform` Argo CD reads `nonprod/`; `pr-platform` Argo CD reads `pr/` | Team members; release bots (7.3) |
| Application source (for example `payments-api`) | Each team | Code and Dockerfile or buildpack settings | `np-build` and `pr-build` (clone at a commit) | Team members |

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
- **Builds** start from a Git webhook to the stack's OpenChoreo API, through an endpoint that only accepts GitHub's published webhook ranges. If inbound webhooks are not allowed, the `build` cluster polls instead.
- **Nothing in one stack ever reads or writes the other stack.** The only way something from non-production can reach production is as a commit in Git.

## 8. Network Flows

All flows stay within one stack. Every flow between the stacks is denied.

| From | To | Port | Direction | Why |
|---|---|---|---|---|
| `platform` | Supervisor API | 443 | On premises | Create and update VKS clusters (`provider-kubernetes`) |
| `platform` | Azure Resource Manager, Entra ID | 443 | Outbound to internet or private link | CAPZ, ASO |
| `platform` | AKS API servers in the stack | 443 | Over hybrid connectivity to private endpoints | CAPI reads node state |
| `platform` (Crossplane, Argo CD) | `fleet` API server | 6443 | On premises | `ManagedCluster`, add-on and `ManifestWork` objects; hub installation |
| `platform` (Argo CD) | `choreo` API server | 6443 | On premises | Apply OpenChoreo configuration and releases from Git |
| `fleet` | New cluster API servers in the stack | 443 / 6443 | On premises; AKS over hybrid connectivity | One-time OCM import |
| `platform` | Git | 443 | Outbound | Argo CD sync |
| Every cluster | `fleet` hub API (OCM) | 443 | Outbound from the cluster; AKS reaches on premises over hybrid connectivity | Klusterlet and add-on agents |
| Team clusters, `build`, `observe` | `choreo` cluster gateway | 443 (WSS) | Outbound from the cluster | OpenChoreo cluster agents |
| Every cluster | `observe` | 4317 (OTLP), 443 | Outbound | Logs and metrics |
| Every cluster | Key Vault (private endpoint) | 443 | Outbound | External Secrets |
| `build` | Git; registry | 443 | Outbound | Clone at commit; push built images |
| Team clusters | Registry (private endpoint) | 443 | Outbound | Pull images |
| GitHub | `choreo` webhook endpoint | 443 | Inbound, GitHub ranges only (optional) | Build triggers |
| People | Portal, AKS API (Entra ID), VKS API (vSphere SSO) | 443, 6443 | Inbound from corporate network | Use and break-glass |
| **Either stack** | **The other stack** | **any** | **Denied** | Isolation |

**Dependency to plan for.** The production stack's AKS clusters must be able to reach `pr-fleet` and `pr-choreo` on premises. In the other direction, `pr-platform` must reach the private AKS API servers. Both paths use the production hybrid connection, so it must be resilient (redundant ExpressRoute or VPN). Running workloads are not affected if it fails, but changes and promotions stop.

## 9. Sizing (Starting Point)

VM classes and Azure sizes follow the proposal's size mapping. Revisit them after the pilot.

| Cluster | Control plane | Workers | Storage |
|---|---|---|---|
| `np-platform`, `pr-platform` | 3 × `best-effort-large` | 3 × `best-effort-large` | 50 GiB per node |
| `np-fleet`, `pr-fleet` | 3 × `best-effort-large` | 3 × `best-effort-medium` | 50 GiB per node |
| `np-choreo`, `pr-choreo` | 3 × `best-effort-large` | 3 × `best-effort-large` | 50 GiB per node |
| `np-build` | 3 × `best-effort-large` | 4 × `best-effort-xlarge` (builds burst here) | 200 GiB build cache |
| `pr-build` | 3 × `best-effort-large` | 3 × `best-effort-xlarge` | 200 GiB build cache |
| `np-observe`, `pr-observe` | 3 × `best-effort-large` | 3 × `best-effort-2xlarge` | 1–2 TiB on vSAN (OpenSearch, Prometheus) |
| `<team>-dv` (VKS) | 1 × `best-effort-medium` | 2 × `best-effort-large` | Team requests |
| `<team>-dv` (AKS) | Free tier | System pool 1 × D2ds_v5; apps 2 × D4ds_v5, autoscale to 0 overnight | Team requests |
| `<team>-te` | 3 control-plane VMs (VKS) or Standard tier (AKS) | 3 × medium | Team requests |
| `<team>-pr` | 3 control-plane VMs (VKS) or Standard tier, 3 zones (AKS) | 3 × large minimum, across zones | Team requests |

**Totals for this example:**

- VKS: 17 clusters (10 platform, 7 team), 45 control-plane VMs and 49 worker VMs. Production uses only the production vSphere capacity.
- AKS: 5 clusters, about 23 nodes, counting system pools (three nodes, across zones, in production).

## 10. Bootstrap Order

For each stack, independently. Production is built from a `platform` tag that is already running in non-production.

1. **`<stack>-platform`**: apply a `Cluster` to the Supervisor; install the components in Section 5.1 at pinned versions; configure the stack's identity, Supervisor tokens and Key Vault access; point Argo CD at the `platform` tag and `platform-<stack>`.
2. **`<stack>-fleet`**: created by the platform tenant's `KubernetesCluster`. Argo CD installs the OCM hub on it, then `<stack>-platform` and `<stack>-fleet` are imported as the first managed clusters and receive the baseline.
3. **`<stack>-choreo`, `<stack>-build` and `<stack>-observe`**: created by the platform tenant. OCM installs the baseline and the OpenChoreo charts. The workflow plane on `build` and the observability plane on `observe` register with the OpenChoreo control plane on `choreo`, just as data planes do.
4. **Tenants**: one `Tenant` per team creates its control plane namespace, quota, Argo CD application, `ManagedClusterSet`, OpenChoreo namespace and environments.
5. **Team clusters**: synced from each team repository's folder for that stack; registered as `DataPlane`s; `Ready` when both agents are connected.

## 11. Growing Beyond This Size

| Signal | Change |
|---|---|
| More than about 25 teams per stack | Shard Argo CD; scale the `fleet` hub's controllers |
| Build queue time keeps growing | Add workers to `<stack>-build`, or add a second workflow plane cluster |
| More than about 30 days of logs, or high cardinality | Scale `observe` out, or send platform metrics to the organisation's central monitoring |
| A second Azure region | Add AKS team clusters in that region from the same stack; a separate stack is not needed |
| A team needs several clusters per environment | Raise its quota; names become `<team>-<env>-<n>` |

## 12. References

- [Self-Service Kubernetes Platform proposal](Self-Service-Kubernetes-Platform-Proposal.md): Sections 5.4 (isolation), 8.4 (OCM), 9 (OpenChoreo) and 12 (operating model)
- OpenChoreo Helm charts: `openchoreo-control-plane`, `openchoreo-data-plane`, `openchoreo-workflow-plane`, `openchoreo-observability-plane`
- Open Cluster Management: https://open-cluster-management.io
- Cluster API Provider Azure: https://capz.sigs.k8s.io
