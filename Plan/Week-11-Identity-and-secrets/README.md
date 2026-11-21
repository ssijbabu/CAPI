# Week 11 · Identity and secrets

Wed 3 Feb 2027 to Tue 9 Feb 2027 · Alpha · days 51–55

[Plan overview](../Implementation-Plan.md) · [← Week 10](../Week-10-Builds/README.md) · [Week 12 →](../Week-12-Production-stack/README.md)

**Goal:** SPIFFE identities issued in the stack; no stored secrets in application paths.

**Exit check (day 55):** SPIRE issues SVIDs on every cluster; a signed image is admitted and an unsigned one is rejected; no Kubernetes Secret in application namespaces.

| Day | Date | A · Controllers | B · Platform | C · Experience |
|---|---|---|---|---|
| [51](Day-051-2027-02-03.md) | Wed 3 Feb | Spike: keyless image signing with the build's SVID. | SPIRE server on the hub; SPIRE agent as an OCM add-on (`AddOnTemplate`); trust domain `np.ssp.internal`. | Identity overview (docs). |
| [52](Day-052-2027-02-04.md) | Thu 4 Feb | Signing step in the build templates. | CSR registration driver with one-hour client certificates; verify rotation. | SR-5 inventory: every remaining credential and its plan. |
| [53](Day-053-2027-02-05.md) | Fri 5 Feb | Signature verification on every cluster, delivered as an add-on policy. | SPIRE OIDC discovery provider; Entra federated credential in a test tenant. | Backstage: show signature and provenance per release. |
| [54](Day-054-2027-02-08.md) | Mon 8 Feb | End-to-end check: no Kubernetes Secret in `app-*` or application namespaces. | Secrets Store CSI driver add-on; workload identity on a dev AKS cluster. | Secrets guide (docs). |
| [55](Day-055-2027-02-09.md) | Tue 9 Feb | Fix-ups. | Close OQ 2 (VKS bootstrap) or record the decision. | Demo. |

## Decisions this week

- Day 51: Decision due today: OQ 1 (Git host authentication).
- Day 53: Decision due today: OQ 6 (SR-9 scope for platform changes).
- Day 55: Decisions due today: OQ 2 (VKS bootstrap) and OQ 4 (VKS image pulls).
