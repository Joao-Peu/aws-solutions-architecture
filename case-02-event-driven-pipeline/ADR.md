# ADR — Case 02: Event-driven Pipeline

## Context

A company receives CSV files from clients containing daily sales data.
Files need to be processed, transformed, and made available for analytical
queries. Volume is unpredictable and the team has no infrastructure expertise.
Processing must be decoupled from upload to avoid blocking the client.

---

## Decision 1: S3 over FTP or API Gateway for file ingestion

**Chosen:** Amazon S3 (input bucket)

**Alternatives considered:**
- FTP server on EC2
- API Gateway with binary payload

**Rationale:**

S3 is the natural entry point for file-based workloads on AWS. It provides
native HTTPS upload, scales to any file size and volume, and integrates
directly with event notifications — eliminating the need for polling.
FTP requires a server to manage. API Gateway has a 10 MB payload limit,
which breaks the 100 MB file requirement.

**Trade-offs accepted:**
- Clients need an S3-compatible upload mechanism (SDK or presigned URL)
- Additional IAM configuration required for secure client access

---

## Decision 2: SQS between S3 and Lambda (async decoupling)

**Chosen:** Amazon SQS (standard queue)

**Alternatives considered:**
- S3 → Lambda direct trigger (without SQS)
- Amazon EventBridge

**Rationale:**

A direct S3 → Lambda trigger works but lacks built-in retry logic and
dead-letter queue support. SQS adds durability: if Lambda fails to process
a file, the message stays in the queue and is retried automatically up to
a configurable number of times. It also decouples throughput — SQS absorbs
bursts of uploads without overwhelming Lambda concurrency limits.

EventBridge would add unnecessary complexity for this straightforward
file-processing pattern.

**Trade-offs accepted:**
- At-least-once delivery (duplicate processing possible — Lambda must be idempotent)
- Small additional latency (~1-2 seconds) between upload and processing start
- SQS standard queue does not guarantee order (FIFO queue available if needed)

---

## Decision 3: Lambda for transformation over Glue

**Chosen:** AWS Lambda

**Alternatives considered:**
- AWS Glue (ETL service)
- EC2 with a processing script

**Rationale:**

AWS Glue is powerful but has a minimum billing of 1 DPU (Data Processing Unit)
per job, with a 10-minute minimum charge — making it expensive for small,
frequent files. Lambda charges per invocation and handles files up to 100 MB
within its 15-minute timeout. For simple CSV transformation, Lambda is
significantly cheaper and faster to implement.

EC2 requires server management, which breaks the zero-ops requirement.

**Trade-offs accepted:**
- Lambda has a 512 MB ephemeral storage limit (sufficient for 100 MB CSV files)
- Complex multi-file joins or very large datasets would require Glue or EMR

---

## Decision 4: S3 + Athena over a traditional data warehouse

**Chosen:** S3 (output bucket) + Amazon Athena

**Alternatives considered:**
- Amazon Redshift
- RDS PostgreSQL

**Rationale:**

Redshift requires a minimum cluster running 24/7 ($180+/month), which is
disproportionate for a startup-scale analytics workload. Athena is serverless
and charges only per query ($5 per TB scanned). Storing transformed data in
Parquet format on S3 reduces scan costs by up to 87% compared to raw CSV,
making Athena extremely cost-effective at this scale.

**Trade-offs accepted:**
- Query latency is higher than Redshift (seconds vs milliseconds)
- No real-time data — queries reflect data as of last processed file
- Requires data to be organized in a query-friendly partition structure

---

## Decision 5: IAM Role per service (least privilege)

**Chosen:** Dedicated IAM Roles per service

**Rationale:**

Each service receives only the permissions it needs:
- Lambda: `s3:GetObject` on input bucket, `s3:PutObject` on output bucket,
  `sqs:ReceiveMessage`, `sqs:DeleteMessage`
- Athena: `s3:GetObject` on output bucket only

This limits blast radius in case of a security event and follows
AWS Well-Architected security best practices.

---

## Summary

| Service | Decision | Primary reason |
|---|---|---|
| S3 (input) | ✅ Chosen | Native file ingestion, event notifications |
| SQS | ✅ Chosen | Retry logic, burst absorption, decoupling |
| Lambda | ✅ Chosen | Pay-per-use, zero ops, handles 100 MB files |
| S3 (output) + Athena | ✅ Chosen | Serverless analytics, no minimum cost |
| IAM Role per service | ✅ Chosen | Least privilege security |
| S3 → Lambda direct | ❌ Rejected | No retry, no dead-letter queue |
| AWS Glue | ❌ Rejected | Minimum 10-min billing, overkill for CSV |
| Redshift | ❌ Rejected | $180+/month minimum, disproportionate cost |
