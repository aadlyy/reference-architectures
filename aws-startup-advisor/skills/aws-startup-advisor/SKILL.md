---
name: aws-startup-advisor
description: Use when a startup or small team needs to choose, size, or justify an AWS architecture — e.g. "what AWS setup should we use for our MVP", "how do we make our EC2 app production-ready", "cheapest way to get HA on AWS", "when should we move to containers/Kubernetes", or "what do we need for SOC 2 on AWS". Walks through workload shape, stage, and constraints and recommends a concrete reference architecture plus the triggers for growing into the next one.
---

# AWS Startup Advisor

Recommend AWS architectures sized to where a startup actually is, not where it
might be in three years. Default to the smallest architecture that meets the
stated requirements; name the specific trigger that justifies the next step up.

## 1. Establish the shape of the workload

Before recommending anything, pin down (ask only what isn't already given):

- **Stage**: pre-launch/prototype, early customers, or scaling with revenue.
- **Traffic shape**: steady, spiky/seasonal, or event-driven (webhooks, batch).
- **Team**: who operates this — one engineer, a small platform team, no ops at all?
- **State**: stateless request/response, needs a relational DB, needs file/blob storage,
  needs long-running jobs or queues.
- **Availability bar**: can it go down for minutes without anyone caring, or is
  there an SLA / paying customers already?
- **Compliance**: none yet, or a named target (SOC 2, HIPAA, PCI) with a deadline.
- **Budget reality**: free-tier/bootstrapped, or funded with a real infra budget.

Don't ask all seven as a questionnaire — infer what you can from context and
confirm the two or three that actually change the recommendation.

## 2. Match to a pattern

Use `references/patterns.md` for the five baseline patterns (static site,
serverless API, container web app, traditional web app, data/event pipeline)
including the AWS services, when each fits, what it costs to run, and its
failure modes. Pick one pattern as the primary recommendation — don't present
all five as equally viable options unless the workload genuinely straddles two.

Rules of thumb:
- No backend logic beyond a form or redirect → static site pattern, not a server.
- Spiky/unpredictable traffic, team wants zero idle cost → serverless pattern.
- Steady traffic, team wants container portability or already uses Docker →
  container pattern.
- Team is AWS novices and workload is a standard CRUD app → traditional
  EC2 + managed RDS is fine; don't push them into ECS/Lambda just because
  it's newer.
- Anything ingesting events/logs/clickstream at volume, or batch analytics →
  data pipeline pattern, kept separate from the application pattern.

## 3. Right-size for stage, not for scale they don't have yet

For each pattern, recommend the pre-launch/early-stage configuration by
default:
- Single AZ, not multi-AZ, until there's a paying customer or an SLA.
- One managed database instance, not a cluster or read replicas, until
  queries or connections are actually the bottleneck.
- On-demand pricing until usage is predictable enough to commit to savings
  plans/reserved capacity (usually 2-3 months of stable billing data).
- Skip a service mesh, multi-account landing zone, or dedicated platform
  team tooling for a team under ~10 engineers unless compliance requires it.

State explicitly what would change the recommendation (e.g. "move to Multi-AZ
RDS once you have customers who'd notice an outage" or "add an ASG once
you've outgrown a single instance's CPU/memory"), so the advice includes the
growth path, not just the starting point.

## 4. Cost-optimize by default

Always mention, where relevant:
- AWS Free Tier coverage for the first 12 months (EC2 t2/t3.micro, RDS
  db.t3.micro, 5GB S3, etc.) — don't recommend paid alternatives to a free-tier
  service that covers the need.
- Autoscaling to zero/minimum rather than a fixed fleet, for anything with
  non-constant load.
- S3 lifecycle rules (move to Infrequent Access / Glacier) for anything kept
  "just in case."
- NAT Gateway cost is often the single biggest surprise line item for a
  startup VPC — point this out when the architecture includes private
  subnets reaching the internet, and suggest NAT instance or VPC endpoints
  as cheaper alternatives at small scale.

## 5. Security and compliance baseline

Recommend this baseline regardless of compliance target, since it's cheap to
do from day one and expensive to retrofit:
- IAM roles per service/workload, no long-lived access keys, no use of the
  root account for daily work.
- Infra as code (CloudFormation/CDK/Terraform) from the first resource, not
  retrofitted later — "click-ops" architectures are the ones startups most
  often can't reproduce or audit later.
- AWS Config + GuardDuty turned on early; both are inexpensive relative to
  the audit pain of enabling them only when a customer or auditor asks.
- Secrets in Secrets Manager/Parameter Store, never in code or plain env files
  committed to a repo.

If a compliance target is named (SOC 2, HIPAA, PCI), say which of the above
become hard requirements rather than good practice, and flag what the chosen
pattern is still missing for it (e.g. encryption at rest/in transit, audit
logging retention, network segmentation) — see `references/patterns.md` for
pattern-specific compliance notes.

## 6. Output format

Give:
1. The recommended pattern in one sentence, naming the core services.
2. A short bulleted architecture (what talks to what).
3. The specific stage-appropriate sizing (not the generic HA version).
4. 2-4 cost notes specific to this workload.
5. The concrete trigger(s) for moving to the next pattern or scaling it up.

Skip generic AWS Well-Architected Framework summaries — give the specific
recommendation for the workload described.
