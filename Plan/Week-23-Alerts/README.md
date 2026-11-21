# Week 23 · Alerts

Wed 28 Apr 2027 to Tue 4 May 2027 · Beta · days 111–115

[Plan overview](../Implementation-Plan.md) · [← Week 22](../Week-22-Observability/README.md) · [Week 24 →](../Week-24-Teams-in-production/README.md)

**Goal:** Alert rules from teams, channels from SREs.

**Exit check (day 115):** An alert from a team's trait reaches the SRE channel without anyone handling a credential.

| Day | Date | A · Controllers | B · Platform | C · Experience |
|---|---|---|---|---|
| [111](Day-111-2027-04-28.md) | Wed 28 Apr | `alert-rules` trait renders `PrometheusRule` and log rules. | Alertmanager per environment. | Alerts guide outline. |
| [112](Day-112-2027-04-29.md) | Thu 29 Apr | Deliver alert rules to the observability cluster by `ManifestWork`. | Notification channels read from Key Vault with workload identity. | Backstage alerts view. |
| [113](Day-113-2027-04-30.md) | Fri 30 Apr | Incidents opened from alerts (S3). | Channel configuration in the fleet repositories. | SRE runbook: alerts and incidents. |
| [114](Day-114-2027-05-03.md) | Mon 3 May | Per-environment switches for alerts and incidents. | Alert routing tests. | Alerts guide. |
| [115](Day-115-2027-05-04.md) | Tue 4 May | Fix-ups. | Record the OCM release status for the beta exit review. | Demo. |
