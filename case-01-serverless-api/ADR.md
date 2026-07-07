# ADR — Case 01: Serverless API

## Context

A startup needs a REST API to manage user tasks. The team has no
infrastructure expertise, traffic is unpredictable, and the budget
is tight. The solution must require zero server management.

---

## Decision 1: Lambda over EC2

**Chosen:** AWS Lambda

**Alternatives considered:**
- EC2 (virtual machine)
- ECS Fargate (containers)

**Rationale:**

Lambda was chosen because the workload is request-driven and
intermittent. With EC2 or Fargate, the team would pay for idle
compute 24/7 even with low traffic. Lambda charges only per
invocation and scales automatically from 0 to thousands of
concurrent requests without any configuration.

**Trade-offs accepted:**
- Cold start latency (~100–300ms on first invocation after idle)
- Maximum execution time of 15 minutes per invocation
- Stateless by design — no in-memory session state

---

## Decision 2: DynamoDB over RDS

**Chosen:** Amazon DynamoDB (on-demand mode)

**Alternatives considered:**
- Amazon RDS (PostgreSQL)
- Amazon Aurora Serverless

**Rationale:**

The task entity has a simple, predictable structure. DynamoDB's
on-demand pricing model matches Lambda's pay-per-request philosophy
— no minimum database instance running 24/7. RDS requires a minimum
instance ($15–30/month even at zero traffic), which breaks the
cost-proportional requirement.

**Trade-offs accepted:**
- No complex SQL queries or joins
- Data modelling must be access-pattern-driven (not relation-driven)
- Less flexible for ad-hoc queries

---

## Decision 3: API Gateway (REST) over ALB or CloudFront

**Chosen:** Amazon API Gateway (REST API)

**Alternatives considered:**
- Application Load Balancer (ALB)
- API Gateway HTTP API (newer, cheaper)

**Rationale:**

API Gateway REST API provides built-in request validation, usage
plans, API keys, and a request/response mapping layer — useful for
a public-facing API. HTTP API would be cheaper (~70% less) but
lacks some of these features. ALB does not natively integrate with
Lambda without extra configuration.

**Note:** If cost becomes the primary concern at scale, migrating
to HTTP API is a low-effort change worth revisiting.

---

## Decision 4: IAM Role (least privilege) for Lambda → DynamoDB

**Chosen:** Dedicated IAM Role per Lambda function

**Rationale:**

Each Lambda function receives only the permissions it needs — for
example, the GET handler gets `dynamodb:GetItem` and `dynamodb:Query`
only, never `dynamodb:DeleteItem`. This follows the principle of
least privilege and limits blast radius in case of a security event.

---

## Summary

| Service | Decision | Primary reason |
|---|---|---|
| Lambda | ✅ Chosen | Pay-per-request, zero ops |
| DynamoDB on-demand | ✅ Chosen | No idle cost, simple schema |
| API Gateway REST | ✅ Chosen | Built-in auth + validation |
| IAM Role per function | ✅ Chosen | Least privilege security |
| EC2 | ❌ Rejected | Idle cost, ops overhead |
| RDS PostgreSQL | ❌ Rejected | Minimum instance cost |
