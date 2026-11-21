# Week 18 · Endpoints and traffic

Wed 24 Mar 2027 to Tue 30 Mar 2027 · Beta · days 86–90

[Plan overview](../Implementation-Plan.md) · [← Week 17](../Week-17-Endpoints/README.md) · [Week 19 →](../Week-19-Dependencies/README.md)

**Goal:** Protocols, external exposure and traffic splitting.

**Exit check (day 90):** A canary takes 10 % of traffic through weighted backends.

| Day | Date | A · Controllers | B · Platform | C · Experience |
|---|---|---|---|---|
| [86](Day-086-2027-03-24.md) | Wed 24 Mar | `external` visibility through the external gateway. | Flux commit status: report reconcile results back to Git commits. | Backstage: URLs per environment. |
| [87](Day-087-2027-03-25.md) | Thu 25 Mar | Weighted backends. | Automated isolation check between the stacks in CI. | Canary guide. |
| [88](Day-088-2027-03-26.md) | Fri 26 Mar | gRPC, TLS passthrough, WebSocket, TCP and UDP routes. | Automated version updates for platform components in `platform`. | Protocol examples in the docs. |
| [89](Day-089-2027-03-29.md) | Mon 29 Mar | Endpoint release and promotion export. | Hub resilience: control plane spread and etcd backups. | Backstage: traffic split view. |
| [90](Day-090-2027-03-30.md) | Tue 30 Mar | Fix-ups. | Resilience review. | Demo. |
