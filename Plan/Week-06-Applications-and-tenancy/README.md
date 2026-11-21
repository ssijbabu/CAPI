# Week 6 · Applications and tenancy

Tue 29 Dec 2026 to Tue 5 Jan 2027 · Alpha · days 26–30

[Plan overview](../Implementation-Plan.md) · [← Week 5](../Week-05-Template-engine/README.md) · [Week 7 →](../Week-07-Releases-and-delivery/README.md)

**Goal:** Onboarding an application creates its hub namespace, cluster set bindings and cluster namespaces.

**Exit check (day 30):** A pull request to `fleet-np` onboards `app-checkout`; its namespace appears on the dv clusters with quota and network policy.

| Day | Date | A · Controllers | B · Platform | C · Experience |
|---|---|---|---|---|
| [26](Day-026-2026-12-29.md) | Tue 29 Dec | Application controller: create `app-<name>`, labels, RoleBindings for the owner's groups. | Environments select SSP-provisioned clusters by label; check the sets SSP created. | Backstage Application page: owner, namespace, cluster sets. |
| [27](Day-027-2026-12-30.md) | Wed 30 Dec | Create a `ManagedClusterSetBinding` per allowed set; only SSP holds `managedclustersets/bind`. | Sample `Environment` and `PromotionPath` objects in `fleet-np`. | Onboarding guide (docs). |
| [28](Day-028-2026-12-31.md) | Thu 31 Dec | `ApplicationType` renders the `ns-<app>` ManifestWork: namespace, `ssp-app-<app>`, Role, quota, network policy, Pod Security labels. | Extend the end-to-end harness with application helpers; first test: onboard an application. | Backstage template "Onboard application" opens a pull request to `fleet-np`. |
| [29](Day-029-2027-01-04.md) | Mon 4 Jan | Application deletion: finalizers, removal of bindings and namespace works. | Split the end-to-end suites: cluster lifecycle (nightly) and applications (every pull request). | Review the template against the onboarding guide. |
| [30](Day-030-2027-01-05.md) | Tue 5 Jan | Fix-ups from review. | Flaky-test triage; cache kind images in CI. | Demo. |
