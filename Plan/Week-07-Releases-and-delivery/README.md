# Week 7 · Releases and delivery

Wed 6 Jan 2027 to Tue 12 Jan 2027 · Alpha · days 31–35

[Plan overview](../Implementation-Plan.md) · [← Week 6](../Week-06-Applications-and-tenancy/README.md) · [Week 8 →](../Week-08-Status-and-overrides/README.md)

**Goal:** A `ReleaseBinding` to dv runs a component on the dv clusters.

**Exit check (day 35):** A merge in the application repository runs `checkout-api` on `team-dv`, delivered by `ManifestWork`.

| Day | Date | A · Controllers | B · Platform | C · Experience |
|---|---|---|---|---|
| [31](Day-031-2027-01-06.md) | Wed 6 Jan | Release controller: `Component` + `Workload` → immutable `ComponentRelease` (content-hash name, immutability webhook). | Environment status: list the clusters each environment matches. | `kubectl ssp status <component>`. |
| [32](Day-032-2027-01-07.md) | Thu 7 Jan | Releases pin type and trait versions (snapshot in the release spec). | Webhook serving certificates: short-lived, rotated by cert-manager. | Backstage Component page: list of releases. |
| [33](Day-033-2027-01-08.md) | Fri 8 Jan | Binding controller: resolve the environment into a `Placement` in `app-<app>`. | Placement tests across team and shared cluster sets. | Component authoring guide (docs). |
| [34](Day-034-2027-01-11.md) | Mon 11 Jan | Binding controller: render the bundle into a MWRS with executor and rollout strategy. | End-to-end: a dv binding runs pods on `team-dv`. | `kubectl ssp tree`: rendered resources of a binding. |
| [35](Day-035-2027-01-12.md) | Tue 12 Jan | Fix-ups; idempotency tests (no update when the render is unchanged). | End-to-end: no update storm on re-sync. | Demo. |
