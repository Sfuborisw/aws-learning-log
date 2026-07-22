# Target architecture — Hypothesis Log on AWS

## Current (before migration)
- Frontend: Vercel
- Backend: Render (FastAPI)
- Database: Supabase (Postgres)

## Target (on AWS)
- Frontend: S3 (static React build) + CloudFront (CDN/HTTPS)
- Backend: EC2 running docker-compose first, then re-deployed on ECS Fargate
- Database: RDS PostgreSQL, in a private subnet, only reachable from the backend's security group

## Security notes
- No root account usage for daily work — IAM user/role only
- RDS not publicly accessible; backend connects via VPC security group rule
- Secrets (DB password, API keys) via environment variables / AWS Secrets Manager, never committed to this repo

## Open questions / decisions log
- [ ] EC2 vs ECS Fargate first — decided: EC2 first for speed, Fargate as the "do it properly" second pass
- [ ] Custom domain via Route53 — decide once app is stable on AWS
