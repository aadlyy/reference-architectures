# AWS Startup Reference Patterns

Five baseline patterns. Each entry: when it fits, the core services, the
early-stage sizing, failure modes to watch for, and compliance notes.

## 1. Static site / JAMstack

**Fits**: marketing sites, docs, landing pages, SPAs whose backend is a
third-party API or a thin serverless API from pattern 2.

**Services**: S3 (origin) + CloudFront (CDN/TLS) + Route 53 (DNS) +
ACM (certificates). Optional: Lambda@Edge or CloudFront Functions for
redirects/headers.

**Early-stage sizing**: single S3 bucket, CloudFront default cache behavior,
no origin failover. This is already highly available by default — don't add
anything.

**Failure modes**: cache invalidation forgotten after deploy (stale content);
S3 bucket made public incorrectly instead of via CloudFront OAC.

**Compliance notes**: encryption in transit is free via ACM/CloudFront;
enable S3 server-side encryption and bucket versioning for audit trails.

## 2. Serverless API

**Fits**: spiky or low/unpredictable traffic, small team wanting ~zero idle
cost, event-driven workloads (webhooks, scheduled jobs), MVPs validating
demand before committing to infra.

**Services**: API Gateway (HTTP API) + Lambda + DynamoDB (or Aurora
Serverless v2 if relational access patterns are needed) + S3 for any
blobs/uploads. Cognito if auth is needed and there's no existing identity
provider.

**Early-stage sizing**: on-demand DynamoDB billing, default Lambda
concurrency, no provisioned concurrency (accept cold starts until p99
latency is actually a complaint).

**Failure modes**: DynamoDB single-table design done without understanding
access patterns first (redesign is painful later); Lambda timeout/memory
left at defaults for workloads that need more of either; N+1 Lambda
invocations driving cost up unexpectedly.

**Compliance notes**: audit logging via CloudTrail + API Gateway access logs;
DynamoDB encryption at rest is on by default; VPC-attach Lambda only if it
must reach private resources (adds cold start cost — don't do it by default).

## 3. Container web app

**Fits**: steady traffic, team already using Docker, need for portability
across environments, workloads with long-running connections (WebSockets)
that don't fit the Lambda execution model well.

**Services**: ALB (load balancer) + ECS Fargate (or EKS only once the team
has genuine multi-service/multi-team scale needs) + RDS (Postgres/MySQL) +
ElastiCache if caching is needed. ECR for images.

**Early-stage sizing**: Fargate service with 1-2 tasks, single-AZ RDS
db.t3/t4g instance, no read replica, no auto-scaling policy beyond a basic
CPU target.

**Failure modes**: choosing EKS for a single small service (operational
overhead far exceeds the benefit below ~5-10 services); RDS connection
exhaustion from Fargate tasks without connection pooling (add RDS Proxy or
an app-level pool before scaling task count).

**Compliance notes**: Fargate tasks get per-task IAM roles (no shared
instance role); enable RDS encryption at rest at creation time (cannot be
added after the fact without a snapshot/restore); enable deletion protection
once there's real data.

## 4. Traditional web app (EC2)

**Fits**: team is new to AWS, workload is a standard monolithic CRUD app
(e.g. Rails/Django/Laravel), no appetite yet for container tooling.

**Services**: ALB + EC2 Auto Scaling Group + RDS. S3 for static
assets/uploads, CloudFront in front of the ALB if there's meaningful static
content.

**Early-stage sizing**: ASG with min=max=1 behind the ALB (still get health
checks and easy replacement on instance failure) single-AZ RDS instance.
Scale the ASG to 2+ instances across AZs only once there's an uptime
commitment to customers.

**Failure modes**: AMIs/instances configured by hand and undocumented
("pet" servers) — use a launch template built from a script or Packer even
at 1-instance scale, so replacing a dead instance doesn't require someone's
memory; security groups left wide open (0.0.0.0/0) on the instance instead
of restricting to the ALB's security group.

**Compliance notes**: same RDS encryption-at-creation caveat as pattern 3;
patch EC2 instances via SSM Patch Manager rather than manual SSH sessions,
both for audit trail and so it scales when there's more than one instance.

## 5. Data / event pipeline

**Fits**: ingesting events, logs, or clickstream at volume; batch analytics;
anything separate from the request/response application path above, even if
it feeds the same product.

**Services**: Kinesis Data Streams or SQS (ingestion) → S3 (landing zone) →
Glue (catalog/ETL) → Athena (ad hoc queries) or Redshift Serverless (if
query volume/concurrency justifies a warehouse). EventBridge for
service-to-service event routing instead of point-to-point integrations.

**Early-stage sizing**: S3 + Athena is almost always the right starting
point — stand up Redshift only once query concurrency or latency on Athena
becomes a real problem, not preemptively.

**Failure modes**: building a Redshift cluster for a dataset that's a few
GB/month (Athena-on-S3 is both cheaper and less operational burden at that
scale); no partitioning scheme on the S3 landing zone, making every Athena
query an expensive full scan.

**Compliance notes**: S3 bucket policies + Lake Formation (once more than a
couple of teams query the same data) for access control; encrypt the
landing zone bucket and set a lifecycle policy before volume makes the
backlog expensive to manage retroactively.
