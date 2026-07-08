# AWS Solutions Architecture

Portfolio of solution architecture case studies designed for AWS,
documenting diagrams, architectural decisions, and cost estimates.

> Built as part of my journey toward AWS Solutions Architect Associate (SAA-C03).

---

## Cases

| # | Case | Services | Status |
|---|------|----------|--------|
| 01 | [Serverless API](./case-01-serverless-api/) | Lambda · API Gateway · DynamoDB · CloudWatch | ✅ Completed |
| 02 | Event-driven Pipeline | SQS · Lambda · S3 · Athena | 🔜 Planned |
| 03 | Containerized Microservices | ECS Fargate · RDS · ALB · ElastiCache | 🔜 Planned |

---

## About

I'm a Systems Analyst and .NET developer building my cloud architecture
portfolio from zero, studying AWS services hands-on and documenting every
decision as a real solutions architect would.

**Certifications in progress:** AWS SAA-C03

---

## Structure

Each case follows the same format:

- `README.md` — problem context and requirements
- `architecture.png` — AWS architecture diagram
- `ADR.md` — architecture decision record (why each service was chosen)
- `cost-estimate.md` — monthly cost breakdown for different usage scenarios
