# Week 4 · Application API foundations

Mon 14 Dec 2026 to Fri 18 Dec 2026 · Alpha · days 16–20

[Plan overview](../Implementation-Plan.md) · [← Week 3](../Week-03-Cluster-registration-and-day-2/README.md) · [Week 5 →](../Week-05-Template-engine/README.md)

**Goal:** Application kinds reviewed; delivery proven on SSP-provisioned clusters.

**Exit check (day 20):** A hand-written `ManifestWorkReplicaSet`, applied by Flux, runs nginx on two SSP-provisioned clusters.

| Day | Date | A · Controllers | B · Platform | C · Experience |
|---|---|---|---|---|
| [16](Day-016-2026-12-14.md) | Mon 14 Dec | Write `Application`, `Environment` and `PromotionPath` types. | Enable the `ManifestWorkReplicaSet` feature gate; fan out a hand-written MWRS through a `Placement`. | Backstage: link clusters to their teams in the catalog. |
| [17](Day-017-2026-12-15.md) | Tue 15 Dec | Write `ClusterComponentType` and `ClusterTrait` types with CEL validation markers. | Cluster health dashboards on the hub: Cluster API and OCM conditions. | Backstage: per-cluster add-on health. |
| [18](Day-018-2026-12-16.md) | Wed 16 Dec | Write `Component`, `Workload`, `ComponentRelease` and `ReleaseBinding` types; generate CRDs. | Prove executor limits: a `ManifestWork` running as a namespaced ServiceAccount cannot write elsewhere. | Backstage: cluster request status, from pull request to `Ready`. |
| [19](Day-019-2026-12-17.md) | Thu 17 Dec | Application API review with the whole team; write ADR-004 (application API). | Write ADR-005: application delivery through `ManifestWorkReplicaSet` and executors. | Weekly summary for stakeholders. |
| [20](Day-020-2026-12-18.md) | Fri 18 Dec | Fix-ups across cluster and application code. | Demo prep. | Demo. |
