# Lessons Learned Log

## 2026-04-21 — Week 2: VPC Module + Dev Env

**What I built:** Reusable VPC module with 2 NAT Gateways, applied to dev env with S3 remote backend, verified in the console, then destroyed cleanly.

**What went well:**
- The remote backend worked properly and the dev environment picked up the S3 state without issues.
- Seeing the VPC, subnets, NAT Gateways and route tables in the AWS Console made the networking layout click properly.

**What surprised me / what I learned:**
- The route tables were the key part that made the HA NAT design make sense. Private route table 1 pointed to NAT 1 and private route table 2 pointed to NAT 2.
- I understood more clearly why 2 NAT Gateways cost more but remove the single point of failure across AZs.

**What I'd do differently next time:**
- I would read the Terraform plan even more slowly before applying so I can map every resource mentally before it is built.
- I want to test the single_nat_gateway option later in a separate environment so I can compare cost and resilience properly.

**Confidence level on topic:** Medium to high
