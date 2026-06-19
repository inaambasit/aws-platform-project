# FinOps Savings Opportunity: AWS Platform Project

Cost analysis and optimisation recommendations for the platform built in Phase 1, analysed on the real stack in region eu-west-2.

## Purpose

Turn a built-and-torn-down AWS EKS platform into a quantified FinOps case study: allocate the cost, find the waste, rank the levers by value, state the trade-offs, and record the decisions. This is the Optimize phase of the FinOps Framework applied to my own infrastructure, not a toy billing export.

## How figures were derived

All cost figures come from AWS Cost Explorer history for the period the stack was live, cross-checked against the CUR export to S3. Unit rates are taken from the AWS pricing for eu-west-2 at time of analysis. No figures are estimated where actuals exist. Where a lever depends on a configuration choice (number of NAT gateways, node count), the calculation is shown so it can be reproduced.

## The stack analysed

- Reusable VPC: public, private and database subnets with NAT
- EKS cluster and managed node group on t3.small (moved up from t3.micro after pod density limits)
- ECR repository for the application image
- AWS Load Balancer Controller via Helm with IRSA, fronting a live ALB
- Five-tag cost allocation: Project, Environment, Owner, CostCenter, ManagedBy
- Usage pattern: ephemeral. Stood up to build and validate, then torn down to zero spend.

## Baseline versus actual

The single largest saving was already realised through teardown discipline. A dev EKS stack left running 24/7 carries a meaningful fixed monthly cost even at near-zero traffic, driven by the EKS control plane, NAT gateways and node compute.

| Scenario | Monthly cost |
|---|---|
| If left running 24/7 (baseline) | £[BASELINE] |
| Actual, with ephemeral teardown | £[ACTUAL] |
| Saving from ephemeral operation | £[SAVING] ( [X]% ) |

This is the headline FinOps point: the biggest lever on a non-production stack is not running it when it is not needed.

## Optimisation recommendation backlog

Ranked by value. Saving figures fill from Cost Explorer; effort and risk are assessed now.

| Lever | Est. saving | Effort | Risk | Owner | Decision |
|---|---|---|---|---|---|
| Ephemeral operation (run only when needed) | £[__]/mo, the largest item | Low | Low | Inaam | Adopted |
| NAT gateway consolidation in non-prod (per-AZ to single) | ~50 to 66% of NAT fixed cost if running one per AZ | Low | Medium (reduced AZ resilience, acceptable in dev) | Inaam | [Decide] |
| Node group on spot for non-prod | up to ~60 to 90% on node compute | Low | Medium (interruptions, fine for dev) | Inaam | [Decide] |
| Node rightsizing and count to actual pod requirements | £[__]/mo | Medium | Low | Inaam | [Decide] |
| ECR lifecycle policy to expire old and untagged images | small, hygiene | Low | Low | Inaam | [Decide] |
| Reduce untagged spend below 1% | improves allocation accuracy | Low | Low | Inaam | [Decide] |

## Lever detail

### 1. Ephemeral operation
The control plane, NAT and idle nodes accrue cost whether or not the app serves traffic. For a portfolio or dev workload, running the stack only during active work, then tearing down, removes nearly all of it. This is already in practice and evidenced by the teardown records. Quantify it as baseline minus actual above.

### 2. NAT gateway consolidation
NAT gateways carry a fixed hourly charge plus a per-GB data processing charge, and in a small EKS dev stack they are typically the largest fixed line item after the control plane. If the build provisioned one NAT gateway per availability zone for resilience, a non-production environment can run a single NAT gateway and cut the fixed NAT charge proportionally. Trade-off: loss of cross-AZ NAT redundancy, which is acceptable in dev and not in prod. Confirm the NAT line item and gateway count in Cost Explorer, then state the saving.

### 3. Spot for the node group
The managed node group ran on-demand. For non-production workloads, spot capacity delivers a large discount on node compute. Trade-off: spot interruptions, which a dev or demo workload tolerates. State the on-demand node cost from Cost Explorer, apply the observed spot discount for the instance type in eu-west-2, and record the net saving.

### 4. Node rightsizing and count
The node group moved from t3.micro to t3.small to clear pod scheduling limits, which are driven by per-instance IP and ENI limits, not raw CPU or memory. Check actual node utilisation: a single small application can leave a t3.small lightly used. The lever is matching node count and size to real pod requirements, and using the Cluster Autoscaler or Karpenter to scale to need rather than provisioning flat. Trade-off: added operational complexity.

### 5. ECR lifecycle
Add a lifecycle policy to expire untagged and superseded images so registry storage does not creep. Small in absolute terms, but it signals cost hygiene and is a one-line policy.

### 6. Untagged spend
Cost allocation is only as good as tag coverage. Measure untagged spend as a percentage of total and drive it below 1% by enforcing the five mandatory tags at provisioning. This is the Inform-phase foundation that makes every other figure trustworthy.

## Unit economics

Beyond raw spend, express cost against a business unit so it connects to value:

- Cost per 1,000 requests served, or
- Cost per pod-hour

Unit cost (chosen metric): £[__] per [unit]. Computed as total stack cost for the period divided by [units served in the period].

## Headline result

Estimated [X]% reduction in monthly run cost versus an always-on baseline, with the optimisation backlog above quantifying a further £[__] of addressable saving on the running stack. Full method and evidence in this repo.

## Risks and trade-offs taken

- NAT consolidation and spot reduce resilience, accepted for non-production only.
- Rightsizing with autoscaling adds operational complexity, weighed against the saving.
- Ephemeral operation requires teardown discipline, evidenced by the runbook and post-destroy checks.

## Decisions

Record each lever decision with date and rationale here as they are made, so the case study doubles as a decision log entry feeding the operating model.
