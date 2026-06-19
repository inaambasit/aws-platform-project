# FinOps Operating Model: AWS Platform Project

How cloud cost would be governed if this platform ran in a real organisation. This page shows the FinOps Framework as a repeating loop (Inform, Optimize, Operate), not a one-off cost cut.

## Tagging policy (Inform)

Five mandatory cost allocation tags on every resource:

| Tag | Purpose |
|---|---|
| Project | Groups spend to a deliverable or product |
| Environment | dev, staging, prod, for lifecycle and rightsizing decisions |
| Owner | Names the accountable person |
| CostCenter | Maps spend to a budget line |
| ManagedBy | Records the provisioning method, here Terraform |

Rules:
- Tags are applied at provisioning through Terraform, not added after the fact.
- Cost allocation tags are activated in the Billing console so they appear in Cost Explorer and the CUR.
- Target: untagged spend below 1% of total. Anything above is triaged in the monthly review.

## Budgets and alerts (Operate)

- An AWS Budget is set before any spend, with an alert threshold at a defined monthly figure.
- Alerts notify the owner on forecast-to-exceed, not only on breach, so action is possible before the spend lands.
- For a non-production stack the threshold is deliberately low, so any accidental always-on resource is caught fast.

## Anomaly review (Operate)

- Cost is reviewed on a fixed cadence (weekly for an active build phase, monthly when stable).
- Any unexpected line item or trend break is logged, root-caused, and either accepted with a reason or actioned.
- The most common anomaly on this class of stack is an orphaned resource after an incomplete teardown, for example a NAT gateway or Elastic IP left running.

## Showback (Inform)

- A monthly showback report breaks total spend down by tag: Project, Environment, Owner, CostCenter.
- Showback makes spend visible to the owner without enforcing a hard chargeback, which suits a small team or single-owner portfolio context.
- The report carries the daily spend trend and the untagged percentage alongside the breakdown.

## Optimisation backlog (Optimize)

- Optimisation levers are tracked as a ranked backlog, not handled ad hoc. See finops-savings-opportunity.md for the live list.
- Each item carries: estimated saving, effort, risk, owner, decision.
- Items are reviewed each cycle. Adopted, deferred or rejected, each with a reason.

## Decision log (Operate)

Every material cost decision is recorded so the rationale survives:

| Date | Decision | Rationale | Owner |
|---|---|---|---|
| [date] | [e.g. single NAT gateway in dev] | [reduced AZ resilience accepted for non-prod] | Inaam |

## Teardown and ephemeral environment control (Operate)

- Non-production stacks are ephemeral: stood up for active work, torn down after.
- Teardown is verified, not assumed. Post-destroy checks confirm no EKS cluster, no load balancers, no active NAT gateways, no Elastic IPs, no EC2 worker nodes, and an empty Terraform state.
- This control is the single largest cost lever on a non-production workload and is evidenced in the teardown records under docs/.

## The loop

Inform feeds Optimize feeds Operate, then back to Inform with better data each cycle:

1. Inform: tag, allocate, report, measure untagged spend and unit cost.
2. Optimize: rank and action the savings backlog, record the trade-offs.
3. Operate: budgets, anomaly review, showback, decision log, teardown control.

The point of the model is repeatability: cost stays governed continuously, rather than being cleaned up once.
