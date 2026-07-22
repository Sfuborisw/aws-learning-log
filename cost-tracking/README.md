# Cost tracking

## One-time setup (do before touching any service)
- [ ] AWS Budgets: $1 monthly budget with email alert
- [ ] Cost Anomaly Detection: enabled
- [ ] Billing preferences: Free Tier usage alerts enabled

## Weekly habit
Check before closing the laptop each session:
- EC2 > Instances — anything running that shouldn't be?
- EC2 > Volumes — any unattached EBS volumes?
- EC2 > Elastic IPs — any unattached (these bill even when idle)?
- VPC > NAT Gateways — these bill ~$33/month just for existing; delete if not needed
- RDS — any test instances left running 24/7 unnecessarily?

## Log
| Date | Bill so far | Anything unexpected | Action taken |
|------|-------------|----------------------|---------------|
|      |             |                      |               |
