# Week 3 · Cluster registration and day 2

Mon 7 Dec 2026 to Fri 11 Dec 2026 · Alpha · days 11–15

[Plan overview](../Implementation-Plan.md) · [← Week 2](../Week-02-Cluster-provisioning-local/README.md) · [Week 4 →](../Week-04-Application-API-foundations/README.md)

**Goal:** Provisioned clusters join OCM, get their add-ons, and can be scaled, upgraded and deleted from Git.

**Exit check (day 15):** Three local clusters (`team-dv`, `team-te`, `shared-te`) requested by pull request are `Ready` in OCM with add-ons; one is scaled, upgraded and deleted with nothing left behind.

| Day | Date | A · Controllers | B · Platform | C · Experience |
|---|---|---|---|---|
| [11](Day-011-2026-12-07.md) | Mon 7 Dec | Registration: create the `ManagedCluster` with labels and cluster set; OCM's importer joins it. | SSP creates cluster sets on demand: `team-<name>` and `shared-<env>`. | "Requesting a cluster" guide. |
| [12](Day-012-2026-12-08.md) | Tue 8 Dec | Add-ons: platform `Placement`s over all SSP clusters; `Ready` only when the type's add-ons are healthy. | Add-on placements in `platform` (policy placeholder now; SPIRE and telemetry later). | Backstage "Request a cluster" template opens a pull request to `fleet-np`. |
| [13](Day-013-2026-12-09.md) | Wed 9 Dec | Scale and upgrade: size and version changes update the Cluster API topology. | End-to-end: scale a local cluster, then upgrade its Kubernetes version. | `kubectl ssp cluster scale` and `upgrade` (write pull requests). |
| [14](Day-014-2026-12-10.md) | Thu 10 Dec | Delete: finalizer removes the `ManagedCluster`, then the Cluster API `Cluster`, then the namespace. | End-to-end: deleting a cluster leaves no containers, objects or Secrets. | Cluster lifecycle guide. |
| [15](Day-015-2026-12-11.md) | Fri 11 Dec | Fix-ups. | Request `team-dv`, `team-te` and `shared-te` through `fleet-np`; these are the development clusters for the following weeks. | Demo. |

## Decisions this week

- Day 11: Decision due today: OQ 9 (Cluster API kubeconfig Secrets and SR-5).
