# Week 8 · Status and overrides

Wed 13 Jan 2027 to Tue 19 Jan 2027 · Alpha · days 36–40

[Plan overview](../Implementation-Plan.md) · [← Week 7](../Week-07-Releases-and-delivery/README.md) · [Week 9 →](../Week-09-Promotion/README.md)

**Goal:** Teams see per-cluster health and can tune each environment.

**Exit check (day 40):** `ReleaseBinding.status` shows health per cluster, and Flux waits for it.

| Day | Date | A · Controllers | B · Platform | C · Experience |
|---|---|---|---|---|
| [36](Day-036-2027-01-13.md) | Wed 13 Jan | Status feedback rules: Deployment availability and well-known status. | Scrape hub metrics: `ManagedCluster`, add-on and `ManifestWork` conditions. | Backstage per-cluster health view. |
| [37](Day-037-2027-01-14.md) | Thu 14 Jan | Aggregate the MWRS summary and per-cluster feedback into `ReleaseBinding.status`; `Ready` condition. | Flux `Kustomization` health checks wait on SSP `Ready`. | `kubectl ssp status` per cluster. |
| [38](Day-038-2027-01-15.md) | Fri 15 Jan | `state: Undeploy`; cascade deletion of the bundle. | End-to-end: undeploy, redeploy, delete. | Status reference (docs). |
| [39](Day-039-2027-01-18.md) | Mon 18 Jan | Overrides: parameters, environment variables, files; per-environment schema validation. | Performance baseline: 10 clusters × 50 components. | Backstage bindings tab with override diff. |
| [40](Day-040-2027-01-19.md) | Tue 19 Jan | Fix-ups. | Record baseline numbers in the repo. | Demo and retro. |
