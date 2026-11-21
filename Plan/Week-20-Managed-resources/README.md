# Week 20 · Managed resources

Wed 7 Apr 2027 to Tue 13 Apr 2027 · Beta · days 96–100

[Plan overview](../Implementation-Plan.md) · [← Week 19](../Week-19-Dependencies/README.md) · [Week 21 →](../Week-21-Resources-release-lines-and-adoption/README.md)

**Goal:** Teams request infrastructure through resource types.

**Exit check (day 100):** A `Resource` of type `postgres` becomes ready, and the binding waits for it.

| Day | Date | A · Controllers | B · Platform | C · Experience |
|---|---|---|---|---|
| [96](Day-096-2027-04-07.md) | Wed 7 Apr | `Resource` and `ClusterResourceType` types. | ASO on the platform clusters reachable from the resource path. | Resources guide outline. |
| [97](Day-097-2027-04-08.md) | Thu 8 Apr | Render resources into `ManifestWork`; `readyWhen` through status feedback. | Postgres resource type in `platform` (Azure Database for PostgreSQL). | Backstage resources tab. |
| [98](Day-098-2027-04-09.md) | Fri 9 Apr | Outputs published to a ConfigMap; inject into workloads. | Entra passwordless sign-in for Postgres. | `kubectl ssp resources`. |
| [99](Day-099-2027-04-12.md) | Mon 12 Apr | Bindings wait for resources in the same environment. | Key Vault fallback through the CSI driver for password-only services. | Resources guide. |
| [100](Day-100-2027-04-13.md) | Tue 13 Apr | Fix-ups. | End-to-end on a real Azure subscription. | Demo. |
