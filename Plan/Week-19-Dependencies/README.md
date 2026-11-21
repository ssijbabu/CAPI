# Week 19 · Dependencies

Wed 31 Mar 2027 to Tue 6 Apr 2027 · Beta · days 91–95

[Plan overview](../Implementation-Plan.md) · [← Week 18](../Week-18-Endpoints-and-traffic/README.md) · [Week 20 →](../Week-20-Managed-resources/README.md)

**Goal:** Components call each other by declared dependency.

**Exit check (day 95):** A caller receives the resolved URL of its dependency, and is refused when the target does not allow it.

| Day | Date | A · Controllers | B · Platform | C · Experience |
|---|---|---|---|---|
| [91](Day-091-2027-03-31.md) | Wed 31 Mar | Resolve `dependencies.endpoints` from the target's `ReleaseBinding.status`. | OCM upgrade rehearsal in kind: hub and klusterlets upgraded with SSP running. | Dependencies guide outline. |
| [92](Day-092-2027-04-01.md) | Thu 1 Apr | Inject resolved URLs into the caller's render. | End-to-end: a klusterlet registers with an SVID and no bootstrap token. | Backstage dependency graph. |
| [93](Day-093-2027-04-02.md) | Fri 2 Apr | Allowed-callers check for dependencies across applications. | Cluster removal runbook: detach a cluster and clean up its hub namespace. | `kubectl ssp deps`. |
| [94](Day-094-2027-04-05.md) | Mon 5 Apr | Report pending and resolved connections in status. | Network policy for allowed callers. | Dependencies guide. |
| [95](Day-095-2027-04-06.md) | Tue 6 Apr | Fix-ups. | End-to-end: SSP survives an OCM upgrade without re-rendering. | Demo. |
