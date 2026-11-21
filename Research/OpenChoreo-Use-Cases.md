# OpenChoreo: Capabilities and Use Cases

**A study of what OpenChoreo does, who it serves, and how it fits the self-service platform**

| | |
|---|---|
| **Status** | Draft |
| **Date** | 2026-11-20 |
| **Source studied** | `openchoreo/openchoreo` at commit `12dce69e2` (version `1.4.0-m.2`): API types, docs, samples, Helm charts, the `occ` CLI, the MCP server and the AI agents |
| **Related** | [Self-Service Kubernetes Platform proposal](../Proposal/Self-Service-Kubernetes-Platform-Proposal.md), [Reference deployment design](../Proposal/Reference-Deployment-Design.md) |

---

## 1. Summary

OpenChoreo is an open-source internal developer platform for Kubernetes and a CNCF Sandbox project. It was originally built by WSO2. It gives three groups of people their own view of the same system:

- **Developers** describe *what* to run: projects, components, endpoints, dependencies, and promotions between environments.
- **Platform engineers** decide *how* it runs. They write templates: component types, traits, resource types, project types, endpoint types and workflows. They also define environments, pipelines, planes and access rules.
- **SREs and operators** see logs, metrics, traces, alerts, incidents, costs and audit trails through the same abstractions developers use. Optional AI agents help with root cause analysis and cost.

Everything is a Kubernetes custom resource (`openchoreo.dev/v1alpha1`). The control plane turns developer intent and platform templates into rendered Kubernetes manifests, and an agent on each target cluster applies them.

This study found **53 distinct use cases** (28 for developers, 16 for platform engineers, 9 for SREs), listed in Section 4. Most of them are directly useful to our platform. A few need care in our design, mainly because production is isolated and Git is the only link between environments (Section 5).

## 2. How OpenChoreo Is Built

### 2.1 Planes

| Plane | Runs | Notes |
|---|---|---|
| **Experience** | Backstage-based portal (optional since the latest release), `occ` CLI, REST API, MCP server, GitOps (`kubectl` / Argo CD), AI agents | Every surface uses the same API and authorization |
| **Control plane** | Controllers, OpenChoreo API, cluster gateway, event forwarder | Renders manifests; holds no workload traffic |
| **Data plane** | Applications, a gateway (kgateway with the Gateway API), the cluster agent | One or many clusters, registered as `DataPlane` or `ClusterDataPlane` |
| **Workflow plane** | Argo Workflows and a cluster agent | Builds and general automation |
| **Observability plane** | Observer API, OpenTelemetry Collector, OpenSearch, Prometheus, a cluster agent; optional RCA and FinOps agents | Logs, metrics, traces, alerts, incidents |

Every plane's agent connects **outbound** to the control plane's cluster gateway over a WebSocket with mutual TLS. No plane needs inbound access from the control plane.

### 2.2 The resource model

| Group | Kinds | Who writes them |
|---|---|---|
| Developer abstractions | `Project`, `Component`, `Workload`, `ReleaseLine`, `Endpoint`, `Resource` | Developers |
| Immutable snapshots | `ComponentRelease`, `ResourceRelease`, `EndpointRelease`, `ProjectRelease` | The control plane (from the abstractions above) |
| Bindings (per environment) | `ReleaseBinding`, `ResourceReleaseBinding`, `EndpointReleaseBinding`, `ProjectReleaseBinding` | Developers, platform engineers or GitOps |
| Output | `RenderedRelease` | The control plane; applied to a data plane or the observability plane |
| Templates (namespaced or cluster-wide) | `ComponentType`, `Trait`, `ResourceType`, `ProjectType`, `EndpointType`, `Workflow` | Platform engineers |
| Runs | `WorkflowRun` | Developers, automation |
| Infrastructure | `Environment`, `DeploymentPipeline`, `DataPlane`, `WorkflowPlane`, `ObservabilityPlane` (each also cluster-scoped) | Platform engineers |
| Configuration and security | `SecretReference`, `AuthzRole`, `AuthzRoleBinding` (and cluster variants) | Platform engineers |
| Alerting | `ObservabilityAlertRule`, `ObservabilityAlertsNotificationChannel` | Developers (through a trait), platform engineers |

**The core flow:**

```text
Project ──► Component ──► Workload (image, env, files, endpoints, dependencies)
                 │
                 └──► ComponentRelease (frozen: type + traits + workload)
                           └──► ReleaseBinding (one per environment, with overrides)
                                     └──► RenderedRelease ──► data plane
```

The same release-and-bind pattern applies to `Resource`, `Endpoint` and `Project`. Releases are immutable, so what is promoted is exactly what was tested.

## 3. Ways to Use It

| Surface | What it offers |
|---|---|
| **Portal** | Backstage-based: create components from templates, view deployments, builds, logs and metrics. It can now be left out at install time and its plugins used in an existing Backstage. |
| **`occ` CLI** | Every resource kind, plus: `apply`, `scaffold` (generate a Component or Project from its type), `component deploy` / `promote`, `workflow run`, `logs`, `exec`, `releaseline`, `resource promote`, `endpoint promote`, `tree` (rendered resources of a binding), `remote` (local development), `auditlogs`, `secret create`, contexts and logins |
| **REST API** | `/api/v1/namespaces/{ns}/…` for every resource; enforces promotion paths and authorization |
| **MCP server** | About 150 tools for AI assistants: create, update and query every resource kind; trigger builds; query logs, metrics, traces, events, alerts, incidents, costs, recommendations, DORA metrics and the audit trail |
| **GitOps** | All resources are plain YAML, so Argo CD or Flux can apply them. Writes made this way bypass the API's promotion-path check, by design. |
| **AI agents** | RCA agent (root cause analysis of alerts), FinOps agent (cost analysis on budget alerts), portal assistant "Perch" (read-only chat copilot) |

## 4. Use Cases

Each use case gives what it is, how OpenChoreo does it, and where in the repository it is shown.

### 4.1 Developers: build and deploy

| # | Use case | How | Shown in |
|---|---|---|---|
| D1 | **Deploy a pre-built image** | A `Component` plus a `Workload` naming the image; with `autoDeploy`, a release and a binding to the first environment are created automatically | `samples/from-image/*` |
| D2 | **Build from source** | `Component.spec.workflow` picks a build workflow: Dockerfile, Google Cloud Buildpacks, Paketo Buildpacks or Ballerina. `autoBuild` builds on every push, through webhooks. The build publishes the image and the `Workload`. | `samples/from-source/*`, `samples/getting-started/ci-workflows` |
| D3 | **Use an external CI** | Keep GitHub Actions (or another CI) and deploy images to OpenChoreo. The portal shows GitHub Actions runs today; registering workloads back from GitHub Actions is in progress. | `docs/integrations/github-actions.md` |
| D4 | **Pick a component type** | Service, web application, scheduled task, worker (defaults), or any type the platform team publishes. Workload kinds: deployment, statefulset, cronjob, job, proxy. | `samples/getting-started/component-types` |
| D5 | **Serve different protocols** | HTTP with path routing; HTTP routed from an OpenAPI schema (one route per path and method); gRPC (`GRPCRoute`); TLS passthrough by SNI (`TLSRoute`); WebSocket; TCP and UDP | `samples/component-types/*`, `samples/from-image/echo-websocket-service` |
| D6 | **Configure the application** | Environment variables and mounted files, either literal or from secrets; per-environment overrides on the `ReleaseBinding` | `samples/component-types/component-with-configs` |
| D7 | **Use secrets safely** | `SecretReference` maps keys in an external store (through External Secrets) into the workload; the value never sits in Git or the control plane | Reference guide, `SecretReference` |
| D8 | **Add behaviour with traits** | Attach traits the type allows: autoscaling (HPA, VPA), persistent volumes, health probes, alert rules. Traits create or patch resources. | `samples/getting-started/component-traits` |
| D9 | **Scaffold** | `occ component scaffold` / `occ project scaffold` generates valid YAML from a type and its traits | `occ` |

### 4.2 Developers: connect and expose

| # | Use case | How | Shown in |
|---|---|---|---|
| D10 | **Expose an endpoint with a chosen visibility** | An `Endpoint` routes to workload endpoints with a visibility of `project`, `namespace`, `internal` or `external`. The `EndpointType` decides how each visibility is rendered (routes, network policy). | `Endpoint`, `EndpointType` types |
| D11 | **Split traffic between versions** | An `Endpoint` has weighted backends, including backends on different release lines, for canary or blue-green releases | `Endpoint.spec.backendRefs[].weight` |
| D12 | **Call another component** | `Workload.spec.dependencies.endpoints[]` declares a dependency, including across projects; the resolved URL is injected into the caller, and the binding reports resolved and pending connections | Reference guide, `ReleaseBinding.status` |
| D13 | **Use managed infrastructure** | A `Resource` requests infrastructure from a `ResourceType` (for example Postgres, Valkey or NATS); its outputs (values, Secret or ConfigMap keys) become environment variables or files. The workload's binding waits until the resource is ready. | `samples/from-image/doclet`, `samples/getting-started/cluster-resource-types` |
| D14 | **Run a whole multi-service application** | A project of interconnected components | `samples/gcp-microservices-demo` |

### 4.3 Developers: release and promote

| # | Use case | How | Shown in |
|---|---|---|---|
| D15 | **Promote through environments** | Bind the same immutable `ComponentRelease` to the next environment. The API only allows a release that is already bound in a source environment of the pipeline. | `DeploymentPipeline`, `occ component deploy` |
| D16 | **Tune per environment** | `ReleaseBinding` overrides: component type settings, trait settings, environment variables and files | `ReleaseBinding` |
| D17 | **Undeploy without deleting** | `ReleaseBinding.spec.state: Undeploy` | `ReleaseBinding` |
| D18 | **Run several release lines side by side** | A `ReleaseLine` builds another branch of the same component, with its own workload, releases and bindings, in the same environment. The default types name everything per line, so lines never collide. | `samples/release-lines` |
| D19 | **Promote infrastructure and endpoints separately** | `occ resource promote` and `occ endpoint promote` move a resource or endpoint release to the next environment | `occ` |

### 4.4 Developers: local development, debugging and AI

| # | Use case | How | Shown in |
|---|---|---|---|
| D20 | **Develop locally against real dependencies** | `occ remote` reads a workload's declared dependencies, tunnels each one (endpoints and resources, across projects) to localhost, and starts a shell with the right environment variables. Access is checked per dependency, and sessions expire after 30 minutes by default. | `samples/local-development` |
| D21 | **Inspect what was deployed** | `occ … tree` shows the rendered resources of a binding with their health; `get_resource_events` and `get_resource_logs` through MCP | `occ`, MCP |
| D22 | **Read logs and open a shell** | `occ component logs`, `occ component exec`, build logs per workflow run | `occ` |
| D23 | **Ask an AI assistant to deploy** | Through MCP: create a component from source, build it and promote it to production, step by step or in conversation | `samples/mcp/service-deployment` |
| D24 | **Ask an AI assistant to debug** | Diagnose build failures; analyse a cascading failure from logs and traces; get resource right-sizing advice | `samples/mcp/build-failures`, `log-analysis`, `resource-optimization` |

### 4.5 Developers and teams: self-service automation

| # | Use case | How | Shown in |
|---|---|---|---|
| D25 | **Run standalone automation** | A `Workflow` that is not tied to a component, run as a `WorkflowRun` with parameters | `samples/workflows` |
| D26 | **Create a repository** | Workflow that creates a GitHub or AWS CodeCommit repository | `samples/workflows/scm-create-repo` |
| D27 | **Provision cloud infrastructure with Terraform** | Workflows that create and destroy an Azure SQL database or AWS RDS Postgres, with remote Terraform state, and return a connection string | `samples/workflows/azure-sql-database`, `aws-rds-postgres` |
| D28 | **Generate reports** | Workflow that fetches GitHub statistics and builds a report | `samples/workflows/github-stats-report` |

### 4.6 Platform engineers: golden paths

| # | Use case | How | Shown in |
|---|---|---|---|
| P1 | **Publish component types** | `ComponentType` / `ClusterComponentType`: a parameter schema for developers, a per-environment schema, CEL validation rules, and Kubernetes resource templates with `${…}` expressions, `includeWhen` and `forEach`. It also lists which traits, workflows and endpoint types are allowed. | `docs/templating/*`, `samples/component-types` |
| P2 | **Enforce standards on every component of a type** | Embedded traits: the platform pre-configures trait instances that are always applied | `samples/component-types/component-with-embedded-traits` |
| P3 | **Offer optional capabilities** | `Trait` / `ClusterTrait` that create new resources or JSON-patch existing ones, with their own schemas | Reference guide |
| P4 | **Offer managed infrastructure** | `ResourceType`: templates for any Kubernetes manifest, `readyWhen` checks, named outputs, and a retain or delete policy | Reference guide |
| P5 | **Shape each project's namespace** | `ProjectType`: renders the project's namespace per environment (the "cell"), plus shared pieces such as network policies, quotas and baseline RBAC | `samples/getting-started/cluster-project-types` |
| P6 | **Adopt an existing namespace** | `ProjectReleaseBinding` in `Observe` mode maps an environment to an existing namespace and only reads it | `ProjectReleaseBinding` type |
| P7 | **Control how endpoints are exposed** | `EndpointType`: renders routes and network policy per visibility, and declares the resulting URLs | `EndpointType` type |
| P8 | **Customise builds** | Custom `Workflow` / `ClusterWorkflow` built from Argo `ClusterWorkflowTemplate` steps, for example adding a Spectral OpenAPI linter | `samples/workflows/custom-steps` |
| P9 | **Keep templates safe** | CEL cost limits per expression and per reconcile, loop and patch caps, and an optional render timeout | `docs/templating/cost-enforcement.md` |

### 4.7 Platform engineers: environments, planes and tenancy

| # | Use case | How | Shown in |
|---|---|---|---|
| P10 | **Define environments and promotion paths** | `Environment` (references a data plane, marks production, optional gateway override) and `DeploymentPipeline` (source → targets) | `samples/platform-config` |
| P11 | **Run workloads on many clusters** | Register clusters as `DataPlane` (one namespace) or `ClusterDataPlane` (everyone); several CRs can share one physical cluster through `planeID` | Reference guide |
| P12 | **Separate builds and observability** | `WorkflowPlane` and `ObservabilityPlane` on their own clusters, each with its agent | Helm charts |
| P13 | **Multi-tenancy** | Namespaces act as organisations or tenants; namespaced and cluster-scoped variants of every template and plane | Reference guide |
| P14 | **Fine-grained access control** | `AuthzRole` lists actions (for example `component:create`, `resource:connect`); `AuthzRoleBinding` grants them to a JWT claim, optionally narrowed to a project or component, with `allow` or `deny`. The defaults are admin, platform-engineer, developer, SRE and readers, plus system roles for the agents and portal. | Helm values, reference guide |
| P15 | **Use the company identity provider** | Bring your own OIDC provider (for example Entra ID) instead of the bundled one | `docs/proposals/0186-integrate-external-idp.md` |
| P16 | **Manage secrets through the platform** | Optional secret management: a secret API writes to the plane's store, and `SecretReference` reads from it | `features.secretManagement` (off by default) |

### 4.8 SREs and operators

| # | Use case | How | Shown in |
|---|---|---|---|
| S1 | **Observe by component** | Logs, HTTP and resource metrics, traces, spans and events, filtered by project, component and environment | Observer API, MCP `query_*` tools |
| S2 | **Alert on logs or metrics** | Developers attach the alert-rule trait; platform engineers set email or webhook channels per environment, and can turn alerts, incident creation and AI analysis on or off per environment | `samples/component-alerts` |
| S3 | **Track incidents** | Alerts can open incidents, which can be queried and updated | MCP `query_incidents`, `update_incident` |
| S4 | **AI root cause analysis** | The RCA agent investigates an alert with read-only tools and writes a report; experimental auto-remediation can apply fixes | `agents/sre-agent`, `samples/from-image/url-shortener` |
| S5 | **Cost and right-sizing** | The FinOps agent analyses budget alerts; costs and recommendations can be queried | `agents/finops-agent`, MCP `query_costs`, `query_recommendations` |
| S6 | **Delivery metrics** | DORA metrics (deployment frequency and related measures) | MCP `query_dora_metrics` |
| S7 | **Audit** | Every state-changing call through the API, CLI or MCP is audited and queryable | `occ auditlogs`, MCP `query_audit_logs` |
| S8 | **Platform logs** | Logs of the platform's own components | `occ … platformlogs`, MCP `query_platform_logs` |
| S9 | **Read-only chat copilot** | "Perch" answers questions about platform state and cannot change anything; backend only so far | `agents/portal-assistant` |

## 5. Fit With Our Platform

How each group of use cases maps onto the design in the proposal and the reference deployment design.

| OpenChoreo use case | Fit | Notes for our design |
|---|---|---|
| D1–D9 (build and deploy) | **Strong** | These are the main reasons to adopt OpenChoreo. Builds run on `<stack>-build`. |
| D10–D12 (endpoints and dependencies) | **Strong** | Visibility levels and network policy follow the EndpointType. Recent releases add `project` visibility. |
| D13 (managed infrastructure) | **Strong, with work** | A `ResourceType` can render *any* manifest. On a team data plane it could create an ASO or Crossplane object, giving developers databases through OpenChoreo. The sample Postgres, Valkey and NATS types are for local use only. |
| D15–D17 (promotion) | **Needs adapting** | Inside non-production, promotion from `dv` to `te` uses `ReleaseBinding`s written from Git. To `pr`, the release is copied to the production stack through Git (proposal Section 9.3). OpenChoreo checks promotion paths only through its API, so our pull request rules and Kyverno replace that check. |
| D18–D19 (release lines, separate promotion) | **Strong** | Fits canary releases inside one environment. Resource and endpoint releases also need Git-driven promotion to `pr`. |
| D20 (`occ remote`) | **Non-production only** | Tunnels from a laptop into an environment. Enable it on the non-production stack; keep it **off** in production (`remoteConnect` is off by default). |
| D21–D24 (debugging, AI) | **Strong in non-production** | In production, give AI assistants read-only roles only, because MCP also has write tools. |
| D25–D28 (standalone workflows) | **Useful, but overlaps** | Terraform workflows overlap with Crossplane and ASO. Prefer the platform's `KubernetesCluster` and future Crossplane APIs for infrastructure. Use workflows for repository creation and reports. |
| P1–P9 (golden paths) | **Strong** | Ship them in the `platform` repository and release them by tag to both stacks. |
| P5–P6 (project types, observe mode) | **Strong** | Project types can stamp network policies and quotas into every team namespace. Observe mode helps bring existing applications in gradually. |
| P10–P13 (environments, planes, tenancy) | **Strong** | Matches our design: namespaced `DataPlane` per team cluster, one OpenChoreo per stack, `dv` and `te` environments in non-production and `pr` in production. |
| P14–P15 (access, identity) | **Strong** | Map Entra ID groups to roles. In production, give developers read-only roles; deployments come from Argo CD. |
| P16 (secret management) | **Optional** | We already use External Secrets with Key Vault; `SecretReference` works with that. |
| S1–S9 (operations) | **Strong** | Runs on `<stack>-observe`. Each stack has its own agents and data, which keeps production isolated. The RCA and FinOps agents need an LLM endpoint; decide which one is allowed in production. |

## 6. Maturity and Caveats

- **The APIs are `v1alpha1`** and change between releases. The studied code is a milestone build (`1.4.0-m.2`). Some fields are already deprecated, such as `Workload` endpoint visibility in favour of `Endpoint`. Pin releases and rehearse upgrades.
- **Several features are off by default:** secret management, rollout when referenced configuration changes, and `occ remote`.
- **Some features are early:**
  - The portal assistant is a backend only.
  - RCA auto-remediation is experimental.
  - The GitHub Actions deployment bridge is still a follow-up.
- **Sample infrastructure is not for production.** The getting-started `ClusterResourceType`s run databases as in-cluster StatefulSets.
- **The modular architecture is a proposal.** Modules such as Guard, Vault and AI Gateway are a direction, not shipped features (`docs/proposals/0482`).
- **Promotion checks live in the API.** Git-based writes bypass them, so our own checks must cover that (Section 5).
- **Templates are powerful.** Template authors can render any resource onto a data plane. Review component, resource and project types like code, and keep the CEL cost limits on.

## 7. Suggested Pilot Use Cases

Use these to prove the fit during Phase 6 of the proposal:

1. **D2 + D15:** build a service from source in `dv`, promote it to `te` through Git, then to `pr` with the production rebuild.
2. **D10 + D12:** two components in different projects, one calling the other with `project` and `internal` visibility, checked against network policy.
3. **D13:** a `ResourceType` that creates an Azure database through ASO, consumed by a workload as environment variables.
4. **D18 + D11:** a second release line with a 10% traffic split.
5. **P5:** a `ProjectType` that adds a default-deny network policy and a quota to every team namespace.
6. **S2 + S4:** an alert rule on a component, a webhook channel, and an RCA report in non-production.
7. **D20:** `occ remote` against `te` dependencies, with production access confirmed to be refused.

## 8. Sources in the Repository

| Area | Path |
|---|---|
| Resource kinds | `docs/resource-kind-reference-guide.md`, `api/v1alpha1/*_types.go` |
| Templating | `docs/templating/*` |
| Design proposals | `docs/proposals/0159` (plane separation), `0186` (external IdP), `0200` (developer concepts), `0245` (build plane), `0482` (modular architecture), `0537` (component types) |
| Samples | `samples/*` (each with a README) |
| CLI | `internal/occ/cmd/*` |
| MCP tools | `pkg/mcp/tools/*`, `internal/observer/mcp` |
| AI agents | `agents/sre-agent`, `agents/finops-agent`, `agents/portal-assistant` |
| Install | `install/helm/openchoreo-*` |
