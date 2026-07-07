# Case 01 — Serverless API

## Problem Statement

A startup needs a REST API to manage user tasks. The team has no
infrastructure expertise and expects unpredictable traffic — from near
zero during weekends to spikes on weekday mornings.

## Requirements

**Functional**
- Create, read, update and delete tasks per user
- Each task has: title, description, status, due date
- API must be accessible over HTTPS

**Non-functional**
- Auto-scaling: handle traffic spikes without manual intervention
- Zero server management
- Cost proportional to actual usage (pay-per-request)
- High availability across at least two availability zones

## Constraints

- Team size: 2 engineers, no DevOps
- Budget: under $50/month at 100k requests/month
- Timeline: MVP in 3 weeks

## Architecture Overview

![Architecture Diagram](./architecture.png)

**Services chosen:**
- **API Gateway** — HTTPS endpoint, routing, throttling
- **Lambda** — business logic, scales automatically per request
- **DynamoDB** — NoSQL, serverless, pay-per-request pricing model
- **CloudWatch** — logs and metrics out of the box
- **IAM** — least-privilege roles for Lambda → DynamoDB access

## Documents

- [Architecture Decision Record](./ADR.md)
- [Cost Estimate](./cost-estimate.md)
