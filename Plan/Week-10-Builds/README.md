# Week 10 · Builds

Wed 27 Jan 2027 to Tue 2 Feb 2027 · Alpha · days 46–50

[Plan overview](../Implementation-Plan.md) · [← Week 9](../Week-09-Promotion/README.md) · [Week 11 →](../Week-11-Identity-and-secrets/README.md)

**Goal:** A push builds an image on the build cluster and deploys it to dv.

**Exit check (day 50):** Push to `main` → build on the build cluster → digest back through status feedback → new release in dv.

| Day | Date | A · Controllers | B · Platform | C · Experience |
|---|---|---|---|---|
| [46](Day-046-2027-01-27.md) | Wed 27 Jan | `BuildTemplate` and `BuildRun` types. | Request the build cluster through SSP (`local-medium`, label `ssp.io/plane=build`); Argo Workflows as an add-on. | Design `kubectl ssp build run` and `build logs`. |
| [47](Day-047-2027-01-28.md) | Thu 28 Jan | `BuildRun` → Argo `Workflow` delivered by `ManifestWork` to the build cluster, as the build ServiceAccount. | Registry in kind; plan ACR workload identity for shared environments. | Backstage builds tab. |
| [48](Day-048-2027-01-29.md) | Fri 29 Jan | Status feedback: workflow phase and image digest (JSON path); write the digest to the `Workload`. | Flux `notification-controller` receiver for pushes; fallback to polling. | `kubectl ssp build logs` through `cluster-proxy` (non-production). |
| [49](Day-049-2027-02-01.md) | Mon 1 Feb | Auto-build on push; a new digest creates a new release. | Dockerfile and buildpacks templates in the `platform` repository. | Builds guide (docs). |
| [50](Day-050-2027-02-02.md) | Tue 2 Feb | Fix-ups. | Build cache on the build cluster. | Demo. |

## Decisions this week

- Day 50: Decision due today: OQ 5 (production rebuild or approved copy).
