# Week 2 — NAT Gateway Cost Analysis

**Environment:** dev
**Date:** 2026-04-21
**Decision:** 2 NAT Gateways (HA) with aggressive destroy discipline

## The trade-off

NAT Gateway pricing in eu-west-2 (as of April 2026):
- $0.045/hour per NAT Gateway
- $0.045/GB data processing charge

## Three options considered

| Option                        | Monthly cost (idle) | Availability                          | When to use                                |
|-------------------------------|---------------------|---------------------------------------|--------------------------------------------|
| 1 NAT in single AZ            | ~£27                | ❌ Cross-AZ SPOF — if that AZ fails, all private subnets lose egress | Cheap dev/learning only                    |
| 1 NAT per AZ (this build)     | ~£55                | ✅ True AZ-level resilience            | Production pattern                         |
| Self-managed NAT instances    | ~£15                | ⚠️ Lower resilience, operational burden | Very cost-sensitive, non-critical workloads |

## Why we chose 2 NATs in dev

Two reasons:

1. **Learning value.** The production HA pattern is the one worth building muscle memory around. A single-NAT dev doesn't teach the routing/failure model properly.

2. **Cost is controlled by session discipline, not architecture.**
   - If left running 24/7: ~£55/month
   - If built and destroyed per session (~2–3 hours active): **~£0.50–£1.00 per session**
   - Across 12 project weeks with disciplined `terraform destroy`: **estimated £15–25 total**

## How the HA pattern is implemented

- NAT Gateway 1 → eu-west-2a public subnet
- NAT Gateway 2 → eu-west-2b public subnet
- `private-rt-1` routes `0.0.0.0/0` via NAT 1
- `private-rt-2` routes `0.0.0.0/0` via NAT 2
- `db-rt-1` and `db-rt-2` follow the same per-AZ pattern

Each AZ has an independent egress path. Failure of one AZ does not cascade.

## Variables exposed in the VPC module

`single_nat_gateway = true` collapses to 1 NAT across all AZs (cheap mode for staging if ever needed). Default is `false` to match production pattern.

## Untagged spend

All 23 resources carry the 5-tag FinOps scheme via `default_tags`. Expected untagged spend in Cost Explorer for this stack: 0%. Will validate in Week 10 (CUR pipeline).

## Production recommendation (if this were a real company)

- Keep 2 NATs per VPC in production
- For large-scale egress (>1 TB/month), evaluate VPC endpoints for S3/ECR/DynamoDB to bypass NAT data processing charges — meaningful saving
- Consider AWS PrivateLink for SaaS egress patterns
- Monitor `NATGatewayBytesOutToDestination` CloudWatch metric for anomalies
