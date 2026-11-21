# Week 22 · Observability

Wed 21 Apr 2027 to Tue 27 Apr 2027 · Beta · days 106–110

[Plan overview](../Implementation-Plan.md) · [← Week 21](../Week-21-Resources-release-lines-and-adoption/README.md) · [Week 23 →](../Week-23-Alerts/README.md)

**Goal:** Teams see logs, metrics and traces by component.

**Exit check (day 110):** `kubectl ssp logs` and the portal show a component's logs and metrics across every cluster it runs on.

| Day | Date | A · Controllers | B · Platform | C · Experience |
|---|---|---|---|---|
| [106](Day-106-2027-04-21.md) | Wed 21 Apr | Namespace labels for application, component, environment and cluster. | `telemetry` add-on: OpenTelemetry collectors, mTLS with SVIDs. | Observability guide outline. |
| [107](Day-107-2027-04-22.md) | Thu 22 Apr | Observer API: queries by application and component. | `np-observe`: OTel gateway, Prometheus, OpenSearch. | Backstage logs and metrics views. |
| [108](Day-108-2027-04-23.md) | Fri 23 Apr | Observer API authorization from hub RBAC. | `pr-observe` mirrored. | `kubectl ssp logs` and `metrics` through the observer. |
| [109](Day-109-2027-04-26.md) | Mon 26 Apr | Fleet health metrics exported from the hub. | Retention and sizing. | Observability guide. |
| [110](Day-110-2027-04-27.md) | Tue 27 Apr | Fix-ups. | Check the OCM release status; if the SPIFFE release is out, try it in kind. | Demo. |
