# Week 12 · Production stack

Wed 10 Feb 2027 to Tue 16 Feb 2027 · Alpha · days 56–60

[Plan overview](../Implementation-Plan.md) · [← Week 11](../Week-11-Identity-and-secrets/README.md) · [Week 13 →](../Week-13-Cloud-cluster-providers-AKS-and-VKS/README.md)

**Goal:** A second, isolated stack and the only path into it.

**Exit check (day 60):** te → `kubectl ssp release export` → Operator approval → signed merge → rebuild on pr-build → running in pr.

| Day | Date | A · Controllers | B · Platform | C · Experience |
|---|---|---|---|---|
| [56](Day-056-2027-02-10.md) | Wed 10 Feb | Release export format: frozen release, pr binding, evidence; schema version. | Second local hub `pr-hub` (OCM, Cluster API, Docker provider, Flux, SPIRE `pr.ssp.internal`); request `pr-build` and `team-pr` through `fleet-pr`. | `kubectl ssp release export --to pr`. |
| [57](Day-057-2027-02-11.md) | Thu 11 Feb | pr-hub accepts exported releases (marked as exported); schema check. | `releases-pr`: `CODEOWNERS` for Operators, branch protection; Flux `spec.verify`. | Export opens the pull request with the developer's own Git login. |
| [58](Day-058-2027-02-12.md) | Fri 12 Feb | Production rebuild: `BuildRun` from the recorded commit; bind after success. | pr-hub admission: only Flux service accounts write application kinds. | Pull request body with evidence: te health, provenance, target clusters. |
| [59](Day-059-2027-02-15.md) | Mon 15 Feb | Change windows on the pr `Environment`; bindings stay `Pending` outside them. | End-to-end: unsigned commit not applied; a person's write rejected. | Backstage production view (read-only), with pending bindings. |
| [60](Day-060-2027-02-16.md) | Tue 16 Feb | Fix-ups. | Isolation review: no route, trust or credential between stacks. | Demo. |
