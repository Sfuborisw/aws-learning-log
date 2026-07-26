# Phase 2 — Backend on EC2 (Docker) + private RDS PostgreSQL

Moved the Hypothesis Log backend off Render and onto AWS: a containerized FastAPI app running on EC2, talking to a private RDS PostgreSQL instance that is not reachable from the public internet. The database was migrated from Supabase with its real data intact.

## What was built

- **RDS PostgreSQL 17** (`hypothesis-log-db`, db.t4g.micro, ca-central-1), **Publicly accessible: No**. Lives in the default VPC, reachable only from inside that VPC.
- **EC2 instance** (`hypothesis-log-backend`, t3.micro, Ubuntu 24.04) running Docker. The FastAPI app is built from a Dockerfile and run via docker-compose, listening on port 8001.
- **Security group design** — the core of this phase:
  - EC2 SG (`hypothesis-log-ec2-sg`): inbound SSH (22) from my IP only, and TCP 8001 from anywhere (the public API port).
  - RDS SG: inbound PostgreSQL (5432) whose **source is the EC2 security group**, not an IP range. Only members of the EC2 SG can reach the database.
- **Database migration** — dumped the `public` schema from Supabase with `pg_dump`, transferred the dump to EC2, and restored into RDS. 3 tables, 6 hypotheses, intact.

## Architecture

```
Public internet
    |
    |  HTTP :8001  (allowed by EC2 SG inbound rule)
    v
EC2 instance (t3.micro, Ubuntu 24.04)
  Docker container: FastAPI + uvicorn on 0.0.0.0:8001
    |
    |  Postgres :5432  (allowed because RDS SG inbound
    |                   source = EC2 SG — SG-to-SG reference)
    v
RDS PostgreSQL 17 (db.t4g.micro, Publicly accessible: No)
  tables: hypotheses, signals, hypothesis_signals
```

Both EC2 and RDS are in the same VPC (`vpc-02bcb8306046df39e`), which is what lets the instance reach a private database.

## Services and concepts used

- **AWS:** EC2, RDS (PostgreSQL), VPC, security groups, key pairs
- **Networking / security:** SG-to-SG references (RDS trusts the EC2 SG, not an IP), private database with no public access, SSH key-pair auth, scoping SSH to a single IP
- **Containers:** Dockerfile (layer-cached deps), docker-compose, port mapping, env injection via `.env`
- **Database migration:** `pg_dump --schema=public`, `scp` over the SSH key, `psql -f` restore into RDS
- **Backend config:** SQLAlchemy reading `DATABASE_URL` from env (no code change to switch DB targets), psycopg 3 driver string (`postgresql+psycopg://`)

## Design decisions worth defending in an interview

**Why the RDS security group references the EC2 SG instead of an IP.**
The rule says "allow 5432 from anything in `hypothesis-log-ec2-sg`", not "allow 5432 from 35.182.244.56". If the EC2 instance reboots and its private IP changes, the rule still holds — membership is by security group, not address. This is the idiomatic AWS pattern and it's what keeps the database reachable by the app while closed to everything else.

**Why RDS is not publicly accessible.**
The database has no public endpoint at all. The only path in is from inside the VPC, through the SG rule above. The restore and all verification were done from the EC2 instance for exactly this reason — even my own laptop can't reach the database directly, which is the point.

**Why the app needed no code change to move databases.**
`database.py` reads `settings.database_url` (from the `DATABASE_URL` env var) and branches on the URL scheme — SQLite locally, Postgres in production. Supabase and RDS are both Postgres, so switching from one to the other is a single env-var change. This is the payoff of reading connection details from the environment instead of hardcoding them.

## Debugging notes

- **`pg_dump` version mismatch.** Ubuntu shipped pg_dump 16; Supabase runs Postgres 17.6, and pg_dump refuses to dump a newer server. Fixed by installing `postgresql-client-17` from the PGDG apt repo and calling `/usr/lib/postgresql/17/bin/pg_dump` explicitly.
- **Supabase is IPv6-only on its direct connection.** The WSL2 box has no IPv6 egress, so the direct host was unreachable. Fixed by using Supabase's Session pooler endpoint (IPv4, port 5432 — not the Transaction pooler on 6543, which doesn't support pg_dump).
- **The dump initially pulled Supabase's internal schemas** (`auth`, `storage`, `realtime`). Re-dumped with `--schema=public` to get only the app's own tables, which is what a plain RDS instance can accept.
- **RDS SG rule looked present but wasn't.** The default SG only had its self-referencing "All traffic" rule; the PostgreSQL rule hadn't actually saved. `nc -zv <rds-endpoint> 5432` from EC2 was the clean way to prove the network path independently of any password issue.
- **psycopg 3 driver string.** `postgresql://` made SQLAlchemy look for psycopg2. Had to use `postgresql+psycopg://` to select the installed psycopg 3.

## Cost note

EC2 t3.micro and RDS db.t4g.micro are both small, but unlike Phase 1 they bill while running. Because this was a fixed sprint, everything was verified, screenshotted, then torn down (EC2 terminated, RDS deleted) rather than left running. The Dockerfile, compose file, and dump live in this repo, so the whole stack can be recreated quickly.

## Teardown checklist (run when done)
- [ ] EC2: Instance state -> Terminate instance
- [ ] RDS: Actions -> Delete (skip final snapshot for a throwaway demo; uncheck "create final snapshot")
- [ ] Confirm both gone from their consoles so nothing keeps billing
- [ ] Security groups and key pair can be left (they cost nothing) or cleaned up
