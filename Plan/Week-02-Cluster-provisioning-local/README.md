# Week 2 · Cluster provisioning (local)

Mon 30 Nov 2026 to Fri 4 Dec 2026 · Alpha · days 6–10

[Plan overview](../Implementation-Plan.md) · [← Week 1](../Week-01-Foundations-and-local-cluster-management/README.md) · [Week 3 →](../Week-03-Cluster-registration-and-day-2/README.md)

**Goal:** A `KubernetesCluster` in Git becomes a running local cluster.

**Exit check (day 10):** A pull request to `fleet-np` creates a local cluster that reaches the `Registering` phase with ready nodes.

| Day | Date | A · Controllers | B · Platform | C · Experience |
|---|---|---|---|---|
| [6](Day-006-2026-11-30.md) | Mon 30 Nov | Cluster controller: `KubernetesCluster` → namespace and Cluster API `Cluster` with the type's `ClusterClass` topology. | `ClusterType`s `local-small` and `local-medium` in the `platform` fixture. | `kubectl ssp cluster list` and `get`. |
| [7](Day-007-2026-12-01.md) | Tue 1 Dec | Admission webhook for `KubernetesCluster`: type, environment, version, team, names. | End-to-end harness (JUnit 5, Fabric8 client, kind); first test: request a cluster. | Backstage clusters list from `KubernetesCluster`. |
| [8](Day-008-2026-12-02.md) | Wed 2 Dec | Map Cluster API conditions to phases and conditions. | Run the end-to-end suite in CI with kind and the Docker provider. | `kubectl ssp cluster create` writes the request to a `fleet-np` branch and opens a pull request. |
| [9](Day-009-2026-12-03.md) | Thu 3 Dec | Labels on Cluster API objects (team, environment, stack, type); requests only from Git. | `fleet-np` `clusters/` folder with code owners for platform engineers. | Backstage cluster page: phase, version, nodes, conditions. |
| [10](Day-010-2026-12-04.md) | Fri 4 Dec | Fix-ups; unit test coverage for the cluster controller. | End-to-end: request through Git reaches ready nodes in under ten minutes. | Demo. |

## Decisions this week

- Day 9: Decision due today: OQ 10 (who approves cluster requests in non-production).
