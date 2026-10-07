# AWS Startup Advisor

A Claude Code plugin that recommends AWS reference architectures sized to a
startup's actual stage, team, and budget — not generic Well-Architected
boilerplate.

## What it provides

- **Skill** `aws-startup-advisor` — triggers automatically when you ask about
  choosing, sizing, or growing into an AWS architecture (e.g. "what AWS setup
  should we use for our MVP", "when should we move to containers").
- **Command** `/aws-advisor <workload description>` — explicitly invoke the
  advisor for a specific workload.
- **Reference patterns** (`skills/aws-startup-advisor/references/patterns.md`)
  covering five baseline architectures: static site, serverless API,
  container web app, traditional EC2 web app, and data/event pipeline —
  each with the AWS services involved, early-stage sizing, common failure
  modes, and compliance notes.

## Installing

From within a Claude Code session with this plugin's marketplace added:

```
/plugin install aws-startup-advisor
```

Or point Claude Code at this repository directly as a plugin source.

## Usage

Just describe the workload:

> We're pre-launch, two engineers, expecting spiky traffic from a Product
> Hunt launch. What should our AWS setup look like?

Or invoke explicitly:

```
/aws-advisor SaaS app, steady B2B traffic, team already uses Docker, needs SOC 2 eventually
```
