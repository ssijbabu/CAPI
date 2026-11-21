# Week 1 · Foundations and local cluster management

Mon 23 Nov 2026 to Fri 27 Nov 2026 · Alpha · days 1–5

[Plan overview](../Implementation-Plan.md) · [Week 2 →](../Week-02-Cluster-provisioning-local/README.md)

**Goal:** A hub that can create local clusters with Cluster API and import them into OCM, and the empty SSP repository.

**Exit check (day 5):** A local cluster created by hand with Cluster API is imported into OCM automatically and becomes `Available`.

| Day | Date | A · Controllers | B · Platform | C · Experience |
|---|---|---|---|---|
| [1](Day-001-2026-11-23.md) | Mon 23 Nov | Create the `ssp` repository: Java 25, Quarkus, Java Operator SDK, Maven; lint and unit-test CI. | Script the local hub: one kind cluster running the OCM hub (with `ClusterImporter`), Cluster API and the Docker provider. | Set up the docs site skeleton and a glossary of SSP terms, clusters first. |
| [2](Day-002-2026-11-24.md) | Tue 24 Nov | Write `ClusterType` and `KubernetesCluster` types; generate CRDs. | Add the CAPD `ClusterClass` `ssp-docker`; create one local cluster by hand with `clusterctl`. | Scaffold the `kubectl ssp` plugin (Picocli, GraalVM native binary) with `version` and contexts. |
| [3](Day-003-2026-11-25.md) | Wed 25 Nov | Status model for `KubernetesCluster`: phases, conditions, printer columns. | Import the hand-made cluster with OCM's `ClusterImporter`; document the naming rule. | Scaffold the Backstage frontend and backend plugins; read the hub through the Kubernetes API. |
| [4](Day-004-2026-11-26.md) | Thu 26 Nov | Provider interface in Java: `local` now, `aks` and `vks` later. | Install Flux on the hub and run the Gitea fixture with `platform` and `fleet-np` (with a `clusters/` folder). | Design portal sign-in: Entra ID in shared environments, a local OIDC provider in kind. |
| [5](Day-005-2026-11-27.md) | Fri 27 Nov | Cluster API review with the whole team; write ADR-001 (cluster provisioning). | Write ADR-002 (hub layout: OCM, Cluster API, Flux on one hub); demo prep. | Write ADR-003 (interfaces); demo prep. |

## Decisions this week

- Day 2: Decision due today: OQ 8 (API group name), before the first CRDs.
