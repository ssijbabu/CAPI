# Week 9 · Promotion

Wed 20 Jan 2027 to Tue 26 Jan 2027 · Alpha · days 41–45

[Plan overview](../Implementation-Plan.md) · [← Week 8](../Week-08-Status-and-overrides/README.md) · [Week 10 →](../Week-10-Builds/README.md)

**Goal:** Promotion from dv to te is enforced at admission, Flux writes included.

**Exit check (day 45):** A te binding is rejected while the release is unhealthy in dv, and accepted once it is healthy.

| Day | Date | A · Controllers | B · Platform | C · Experience |
|---|---|---|---|---|
| [41](Day-041-2027-01-20.md) | Wed 20 Jan | Admission webhook: the target environment must be reachable by the `PromotionPath`. | Gitea: protected `main`, required review. | `kubectl ssp promote` (non-production) opens a pull request with the te binding. |
| [42](Day-042-2027-01-21.md) | Thu 21 Jan | Admission check: the release is `Ready` in the source environment. | End-to-end: promotion rejected while dv is unhealthy. | Backstage "Promote" button opens the pull request. |
| [43](Day-043-2027-01-22.md) | Fri 22 Jan | Prove the webhook applies to writes from Flux service accounts. | Webhook high availability; `failurePolicy: Fail`. | Promotion guide (docs). |
| [44](Day-044-2027-01-25.md) | Mon 25 Jan | Map binding rollout to OCM rollout strategies (`Progressive`, `maxConcurrency`). | End-to-end: progressive rollout across two te clusters. | `kubectl ssp history`. |
| [45](Day-045-2027-01-26.md) | Tue 26 Jan | Fix-ups. | CI time budget check. | Demo. |
