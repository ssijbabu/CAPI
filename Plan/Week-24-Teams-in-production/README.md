# Week 24 · Teams in production

Wed 5 May 2027 to Tue 11 May 2027 · Beta · days 116–120

[Plan overview](../Implementation-Plan.md) · [← Week 23](../Week-23-Alerts/README.md) · [Week 25 →](../Week-25-Beta-hardening-and-exit/README.md)

**Goal:** Three teams in production, one shared cluster.

**Exit check (day 120):** Three teams run in pr; two of them share `shared-te`; klusterlets use SVIDs in kind and non-production.

| Day | Date | A · Controllers | B · Platform | C · Experience |
|---|---|---|---|---|
| [116](Day-116-2027-05-05.md) | Wed 5 May | Support onboarding of teams two and three. | Onboard two more teams in both stacks. | Training sessions. |
| [117](Day-117-2027-05-06.md) | Thu 6 May | Fix gaps found. | Shared cluster isolation test with two teams. | Collect feedback. |
| [118](Day-118-2027-05-07.md) | Fri 7 May | Fix gaps found. | If the OCM release with SPIFFE agent identity is out, move non-production agents to it; otherwise record the gap. | Fix portal issues. |
| [119](Day-119-2027-05-10.md) | Mon 10 May | Production releases for teams two and three. | Production capacity review with the Operators. | Update guides. |
| [120](Day-120-2027-05-11.md) | Tue 11 May | Fix-ups. | Retro. | Retro. |

## Decisions this week

- Day 116: Decision due today: OQ 7 (shared clusters in production).
