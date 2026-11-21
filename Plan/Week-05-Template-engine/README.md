# Week 5 · Template engine

Mon 21 Dec 2026 to Mon 28 Dec 2026 · Alpha · days 21–25

[Plan overview](../Implementation-Plan.md) · [← Week 4](../Week-04-Application-API-foundations/README.md) · [Week 6 →](../Week-06-Applications-and-tenancy/README.md)

**Goal:** Render a component type with traits into Kubernetes manifests, offline.

**Exit check (day 25):** `kubectl ssp render` turns a `web-service` component with an autoscaler trait into manifests.

| Day | Date | A · Controllers | B · Platform | C · Experience |
|---|---|---|---|---|
| [21](Day-021-2026-12-21.md) | Mon 21 Dec | Spike: OpenChoreo's template syntax on `cel-java`; port its template tests as fixtures (OQ 3). | Lay out the Git fixture repositories as in the KEP. | Design `kubectl ssp scaffold` from type parameter schemas. |
| [22](Day-022-2026-12-22.md) | Tue 22 Dec | Decide OQ 3; create the `template` module with the CEL environment and evaluation limits. | One Flux `Kustomization` per repository, each impersonating a scoped ServiceAccount. | Backstage entity provider: `Application` → System. |
| [23](Day-023-2026-12-23.md) | Wed 23 Dec | Render resource templates: `${…}` expressions, `includeWhen`, `forEach`; table-driven tests. | Hub RBAC baseline for `app-*` namespaces; Entra structured authentication stub. | Implement `kubectl ssp scaffold component`. |
| [24](Day-024-2026-12-24.md) | Thu 24 Dec | Traits: create and JSON-patch resources; embedded traits; trait schemas. | `ValidatingAdmissionPolicy`: only SSP may create `Placement` and MWRS in `app-*`. | Backstage entity provider: `Component` → Component. |
| [25](Day-025-2026-12-28.md) | Mon 28 Dec | Code review and hardening of the engine; per-reconcile cost budget. | Policy tests for the admission policy. | Wire `kubectl ssp render`; demo. |

## Decisions this week

- Day 22: Decision due today: OQ 3 (template engine).
