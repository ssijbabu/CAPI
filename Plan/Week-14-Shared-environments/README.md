# Week 14 · Shared environments

Wed 24 Feb 2027 to Tue 2 Mar 2027 · Alpha · days 66–70

[Plan overview](../Implementation-Plan.md) · [← Week 13](../Week-13-Cloud-cluster-providers-AKS-and-VKS/README.md) · [Week 15 →](../Week-15-Pilot-application/README.md)

**Goal:** Both stacks running on real clusters.

**Exit check (day 70):** The same demo as week 9, on VKS and AKS clusters instead of kind.

| Day | Date | A · Controllers | B · Platform | C · Experience |
|---|---|---|---|---|
| [66](Day-066-2027-02-24.md) | Wed 24 Feb | Controller hardening: leader election, rate limits, requeue back-off. | Create `np-hub` once (outside SSP); install OCM, Cluster API with CAPZ, Flux, SPIRE and SSP from `fleet-np`. | Start the Operator runbook. |
| [67](Day-067-2027-02-25.md) | Thu 25 Feb | Controller metrics and events. | Request `np-build`, `np-observe`, `payments-dv`, `payments-te`, `shared-te` through SSP; they join OCM automatically. | Deploy the Backstage plugins to the non-production portal with Entra ID. |
| [68](Day-068-2027-02-26.md) | Fri 26 Feb | Fixes from bring-up. | Create `pr-hub` once; request `pr-build`, `pr-observe` and `payments-pr` through `fleet-pr` with Operator approval. | Operator runbook: approving a release. |
| [69](Day-069-2027-03-01.md) | Mon 1 Mar | Upgrade test: SSP vN → vN+1 re-renders only changed bindings. | Entra federation for Flux, ACR and Key Vault in both stacks. | Start the SRE runbook. |
| [70](Day-070-2027-03-02.md) | Tue 2 Mar | Fix-ups. | Bring-up retro; update the reference design if needed. | Demo. |
