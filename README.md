# AWS Learning Log

Tracking my hands-on AWS learning while migrating [Hypothesis Log](https://github.com/Sfuborisw) from Vercel/Render/Supabase to AWS. Goal: build real deployment experience (not just docs) and leave a visible commit trail as a portfolio signal.

## Why this repo exists

- Every AWS concept gets tied to an actual deployment step on Hypothesis Log
- Weekly logs in `/logs` — what I did, what broke, what it cost
- `/architecture` — notes on the target AWS setup and how it evolves
- `/cost-tracking` — billing discipline, so nothing surprises me

## Roadmap

### Tier 1 — core, tied directly to deploying Hypothesis Log
- [ ] IAM — root account locked down, IAM user/role created for daily work
- [ ] S3 — host the React static build
- [ ] CloudFront — CDN + HTTPS in front of S3
- [ ] EC2 — backend running via docker-compose (first pass)
- [ ] ECS Fargate — same backend, containerized properly (second pass)
- [ ] RDS (PostgreSQL) — migrate DB off Supabase, learn VPC/security groups

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
