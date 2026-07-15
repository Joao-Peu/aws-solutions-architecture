# Cost Estimate — Case 02: Event-driven Pipeline

Estimates calculated using the AWS Pricing Calculator (US East — N. Virginia).
All prices in USD. Free tier discounts not applied for realistic projection.
Source files exported directly from AWS Pricing Calculator on 2026-07-15.

---

## Scenarios

| Service | Low (100 files/mo) | Medium (1k files/mo) | High (10k files/mo) |
|---|---|---|---|
| Amazon S3 | $0.03 | $0.08 | $0.35 |
| Amazon SQS | $0.00 | $0.00 | $0.04 |
| AWS Lambda | $0.00 | $0.00 | $0.02 |
| Amazon Athena | $0.00 | $0.02 | $0.25 |
| Amazon CloudWatch | $0.04 | $0.15 | $1.00 |
| **Total/month** | **$0.07** | **$0.25** | **$1.66** |

---

## Assumptions

- Region: US East (N. Virginia) — lowest AWS pricing tier
- File size: average 10 MB per CSV file
- Lambda: 512 MB memory, 5 seconds average duration per file
- Athena: 0.1 GB scanned per query, 50 queries at Low / 500 at Medium / 5k at High
- CloudWatch: 0.5 GB logs at Low, 2 GB at Medium, 10 GB at High
- SQS: 1 message per file (standard queue)
- No data transfer costs included (same-region communication)
- No reserved capacity or savings plans applied

---

## Key Insights

**This pipeline costs $0.07/month at low volume** — processing 100 files
with full observability, retry logic, and SQL query capability for less
than R$0.40/mês.

**SQS and Lambda are nearly free at any scale** — the cost drivers are
CloudWatch (logging) and S3 (storage accumulation over time).

**S3 storage grows over time** — unlike compute costs, stored data
accumulates. At High scale with 10k files/month, implementing S3
Lifecycle Policies to move older data to S3 Glacier is recommended,
reducing storage cost by up to 80%.

**Athena + S3 vs Redshift** — a traditional Redshift cluster would cost
$180+/month at minimum. This serverless alternative delivers the same
SQL query capability for under $2.00/month at High scale.

---

## Cost per File Processed

| Scenario | Total/month | Cost per file |
|---|---|---|
| Low | $0.07 | $0.0007 |
| Medium | $0.25 | $0.00025 |
| High | $1.66 | $0.000166 |

Cost per file actually decreases at scale — demonstrating the
efficiency of serverless event-driven architecture.

---

## Calculator Reference

Estimates exported from AWS Pricing Calculator on 2026-07-15.
JSON source files available in the [`calculator/`](./calculator/) folder.
