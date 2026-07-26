# AWS Learning Log

Tracking my hands-on AWS learning while migrating [Hypothesis Log](https://github.com/Sfuborisw) from Vercel/Render/Supabase to AWS. Goal: build real deployment experience (not just docs) and leave a visible commit trail as a portfolio signal.

## ⚠️ Free plan sprint — account closes Aug 10, 2026

This AWS account is on the Free plan, which auto-closes when credits run out or on **Aug 10, 2026**, whichever comes first. Decision made: treat this as a fixed-length sprint rather than upgrading to Paid.

**Consequence: the live deployed URL will disappear on Aug 10.** Portfolio evidence must NOT rely on a live link. Instead, capture:
- Screenshots of the working app + AWS console (IAM, S3, CloudFront, EC2, RDS all configured)
- This repo's code, configs, and logs (these survive — they're not tied to the AWS account)
- A written case study summarizing what was built and why

See `SPRINT-PLAN.md` for the day-by-day breakdown.

## Why this repo exists

- Every AWS concept gets tied to an actual deployment step on Hypothesis Log
- Weekly logs in `/logs` — what I did, what broke, what it cost
- `/architecture` — notes on the target AWS setup and how it evolves
- `/cost-tracking` — billing discipline, so nothing surprises me

## Roadmap

### Tier 1 — core, tied directly to deploying Hypothesis Log
- [x] IAM — `boris-dev` user created (console + CLI access, MFA); root locked down and set aside
- [x] S3 — hosting the React static build (private bucket, `hypothesis-log-frontend-awslearning`)
- [x] CloudFront — CDN + HTTPS in front of S3 via OAC (`d2remctp8dpvdy.cloudfront.net`)
- [x] EC2 — FastAPI in Docker via docker-compose, port 8001 (t3.micro, Ubuntu 24.04)
- [ ] ECS Fargate — same backend, containerized properly (second pass)
- [x] RDS (PostgreSQL 17) — migrated off Supabase, private (no public access), SG-to-SG access from EC2

**Phase 1 complete** — frontend on private S3 + CloudFront (OAC). Write-up: [`architecture/phase-1-case-study.md`](architecture/phase-1-case-study.md).

**Phase 2 complete** — backend containerized on EC2 + private RDS PostgreSQL, migrated from Supabase. Write-up: [`architecture/phase-2-case-study.md`](architecture/phase-2-case-study.md). Deploy artifacts in [`infra/backend-deploy/`](infra/backend-deploy/).

### Tier 2 — once Tier 1 is live
- [ ] CloudWatch — logs + billing alarms
- [ ] Route53 — custom domain
- [ ] Lambda — rewrite one CLAW BOT scheduled job as Lambda + EventBridge

### Tier 3 — only if a job posting specifically asks for it
- [ ] DynamoDB / SQS / SNS / EKS / Step Functions

## Billing safety checklist (done once, at account setup)
- [ ] $1 AWS Budgets alert created
- [ ] Cost Anomaly Detection enabled
- [ ] Free Tier usage alerts enabled in Billing preferences
- [ ] Weekly resource audit habit (unattached EBS volumes, idle Elastic IPs, NAT Gateways)

## Related
- Live app: (add Hypothesis Log deployed URL once migrated)
- Portfolio: boris-wong.vercel.app
