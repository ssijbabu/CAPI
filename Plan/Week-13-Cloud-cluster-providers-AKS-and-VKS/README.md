# Week 13 · Cloud cluster providers: AKS and VKS

Wed 17 Feb 2027 to Tue 23 Feb 2027 · Alpha · days 61–65

[Plan overview](../Implementation-Plan.md) · [← Week 12](../Week-12-Production-stack/README.md) · [Week 14 →](../Week-14-Shared-environments/README.md)

**Goal:** The same cluster requests now create AKS and VKS clusters.

**Exit check (day 65):** One AKS and one VKS cluster, requested by pull request, are provisioned, registered with OCM and `Ready`.

| Day | Date | A · Controllers | B · Platform | C · Experience |
|---|---|---|---|---|
| [61](Day-061-2027-02-17.md) | Wed 17 Feb | `aks` provider: render a CAPZ managed cluster topology with workload identity and the OIDC issuer enabled. | Install CAPZ on a test hub with workload identity (no client secret). | Cluster types guide: local, AKS, VKS. |
| [62](Day-062-2027-02-18.md) | Thu 18 Feb | `vks` provider: create the cluster on the vSphere Supervisor; SSP imports it into OCM once. | Supervisor access for the hub: a ServiceAccount in each team's vSphere Namespace; network path. | Backstage: provider shown per cluster. |
| [63](Day-063-2027-02-19.md) | Fri 19 Feb | Provider status mapping for CAPZ and VKS conditions. | Private AKS API servers reachable from the hub over the hybrid connection. | `kubectl ssp cluster create --type aks-standard` and `vks-medium`. |
| [64](Day-064-2027-02-22.md) | Mon 22 Feb | `ClusterType`s `aks-standard`, `aks-private`, `vks-small`, `vks-medium` in `platform`. | Nightly cloud test: request an AKS and a VKS cluster, wait for `Ready`, delete. | Backstage: cost estimate per type. |
| [65](Day-065-2027-02-23.md) | Tue 23 Feb | Fix-ups. | Nightly results review. | Demo. |
