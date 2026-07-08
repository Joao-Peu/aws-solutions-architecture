# Cost Estimate — Case 01: Serverless API

Estimates calculated using the AWS Pricing Calculator (US East — N. Virginia).
All prices in USD. Free tier discounts not applied for realistic projection.
Source files exported directly from AWS Pricing Calculator on 2026-07-07.

---

## Scenarios

| Service | Low (10k req/mo) | Medium (100k req/mo) | High (1M req/mo) |
|---|---|---|---|
| AWS Lambda | $0.00 | $0.02 | $0.20 |
| Amazon API Gateway | $0.04 | $0.35 | $3.50 |
| Amazon DynamoDB | $0.01 | $0.07 | $0.81 |
| Amazon CloudWatch | $0.04 | $1.50 | $7.50 |
| **Total/month** | **$0.09** | **$1.94** | **$12.02** |

---

## Assumptions

- Region: US East (N. Virginia) — lowest AWS pricing tier
- Lambda: 128 MB memory, 200ms average duration per invocation
- DynamoDB: on-demand capacity mode, 3:1 read/write ratio
- CloudWatch: 0.5 GB logs at Low, 2 GB at Medium, 10 GB at High
- No data transfer costs included (same-region communication)
- No reserved capacity or savings plans applied

---

## Key Insights

**Cost scales linearly with usage** — the serverless model charges
only for what you consume, with no idle baseline cost.

**CloudWatch becomes the biggest cost driver at scale** — at 1M
requests/month, logging represents ~62% of the total bill. In
production, setting log retention policies and sampling strategies
would significantly reduce this cost.

**DynamoDB on-demand is ideal at low-to-medium traffic** — at high
scale, switching to provisioned capacity with auto-scaling could
reduce DynamoDB costs by up to 70%.

**Lambda is practically free** — even at 1M requests/month the cost
is only $0.20, demonstrating why serverless is the default choice
for event-driven workloads.

---

## Cost per Request

| Scenario | Total/month | Cost per request |
|---|---|---|
| Low | $0.09 | $0.000009 |
| Medium | $1.94 | $0.000019 |
| High | $12.02 | $0.000012 |

---

## Calculator Reference

Estimates exported from AWS Pricing Calculator on 2026-07-07.
JSON source files available in `calculator/` folder.
