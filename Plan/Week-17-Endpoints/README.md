# Week 17 · Endpoints

Wed 17 Mar 2027 to Tue 23 Mar 2027 · Beta · days 81–85

[Plan overview](../Implementation-Plan.md) · [← Week 16](../Week-16-Alpha-hardening-and-exit/README.md) · [Week 18 →](../Week-18-Endpoints-and-traffic/README.md)

**Goal:** Components expose endpoints with a chosen visibility.

**Exit check (day 85):** An endpoint with `project` visibility is reachable inside the application only; `external` is reachable through the external gateway.

| Day | Date | A · Controllers | B · Platform | C · Experience |
|---|---|---|---|---|
| [81](Day-081-2027-03-17.md) | Wed 17 Mar | `Endpoint` type and controller. | Track OCM releases: record what the release with SPIFFE agent identity must provide, and its expected date. | Endpoints guide outline. |
| [82](Day-082-2027-03-18.md) | Thu 18 Mar | `ClusterEndpointType`: routes and network policy per visibility. | Baseline policy add-on on every cluster: Pod Security, required labels. | Backstage endpoints view. |
| [83](Day-083-2027-03-19.md) | Fri 19 Mar | `project` and `namespace` visibility. | Gateway API CRDs and kgateway as an add-on. | `kubectl ssp endpoints`. |
| [84](Day-084-2027-03-22.md) | Mon 22 Mar | `internal` visibility across clusters of an environment. | Internal gateway per environment. | Endpoints guide. |
| [85](Day-085-2027-03-23.md) | Tue 23 Mar | Fix-ups. | Policy add-on tests in the end-to-end suite. | Demo. |
