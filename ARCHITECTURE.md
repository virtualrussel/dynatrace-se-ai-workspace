# Architecture Overview

This document explains what this workspace is, how it is built, and how the components work together. It is intended for anyone who wants to understand the solution before using it.

---

## What This Is

A pre-configured AI observability workspace that connects GitHub Copilot or Claude Code to Dynatrace, enabling natural language investigation of production systems from VS Code or the Claude Code CLI.

Instead of logging into Dynatrace, navigating dashboards, and writing queries manually, you type a slash command in Copilot Chat or Claude Code and receive structured, accurate, production-aware answers in seconds.

---

## The Problem It Solves

GitHub Copilot, or Claude Code, is a general-purpose AI assistant. Without domain-specific knowledge it will:
- Guess DQL syntax and get it wrong
- Use field names that don't exist (`log.level` instead of `loglevel`)
- Write queries that hit scan limits and return zero results
- Have no access to your live Dynatrace data

This workspace solves all four problems by combining three things: domain knowledge, live data access, and pre-built workflows.

---

## Architecture

```
┌───────────────────────────────────────────────────────────────────┐
│                    VS Code                                        │
│                                                                   │
│    ┌─────────────────┐           ┌──────────────────────────┐     │
│    │     AI Chat     │           │   Integrated Terminal    │     │
│    │                 │           │                          │     │
│    │  /health-check  │           │  dtctl get workflows     │     │
│    │  /troubleshoot  │           │  dtctl query "..."       │     │
│    │  /standup       │           │  dtctl describe notebook │     │
│    └────────┬────────┘           └────────────┬─────────────┘     │
│             │                                 │                   │
│    ┌────────▼────────┐                        │                   │
│    │  Agent Skills   │                        │                   │
│    │   (16 skills)   │                        │                   │
│    │ .agents/skills/ │                        │                   │
│    └────────┬────────┘                        │                   │
└─────────────┼─────────────────────────────────┼───────────────────┘
              │ MCP (stdio) + OAuth             │ HTTPS + OAuth
              ▼                                 ▼
┌───────────────────────────────────────────────────────────────────┐
│              Dynatrace Platform                                   │
│                                                                   │
│   guu84124.apps.dynatrace.com  (production)                       │
│   bon05374.sprint.apps.dynatracelabs.com  (sprint)                │
│                                                                   │
│   Grail data lakehouse — logs, spans, metrics, events             │
│   Dynatrace Intelligence — problem detection, root cause analysis │
│   Notebooks, Dashboards, Workflows                                │
└───────────────────────────────────────────────────────────────────┘
```

---

## The Five Components

### 1. Agent Skills
**Source:** [github.com/Dynatrace/dynatrace-for-ai](https://github.com/Dynatrace/dynatrace-for-ai) & [github.com/dynatrace-oss/dtctl](https://github.com/dynatrace-oss/dtctl) **Location:** `.agents/skills/`

Skills are markdown files containing domain-specific knowledge. They teach AI assistants how Dynatrace works including correct DQL syntax, field names, query patterns, and investigation workflows. They load automatically when relevant, using a three-tier progressive disclosure model:

```
Tier 1 — Catalog     Always loaded    ~100 tokens per skill
Tier 2 — SKILL.md    On demand        ~5,000 tokens
Tier 3 — references/ On demand        Deep reference detail
```

This means all 16 skills can be installed without performance penalty — the AI only loads what it needs for each specific query.

| Skill | Domain |
|---|---|
| `dt-dql-essentials` | Core DQL syntax rules, common pitfalls, and query patterns — load before writing any DQL |
| `dt-obs-problems` | DAVIS problem analysis, root cause identification, impact assessment, and correlation with other telemetry |
| `dt-obs-logs` | Log querying, filtering, pattern analysis, and error rate calculation |
| `dt-obs-tracing` | Distributed traces, spans, service dependencies, and request flow analysis |
| `dt-obs-services` | RED metrics (Rate, Errors, Duration) and runtime-specific telemetry for Java, .NET, Node.js, Python, PHP, and Go |
| `dt-obs-hosts` | Host and process metrics — CPU, memory, disk, network, containers, and process-level telemetry |
| `dt-obs-kubernetes` | Kubernetes cluster, pod, node, and workload monitoring — health, pod failures, OOMKills, scheduling, and security posture |
| `dt-obs-aws` | AWS cloud resources — EC2, RDS, Lambda, ECS/EKS, VPC, load balancers, S3, DynamoDB, SQS/SNS, and cost optimization |
| `dt-obs-azure` | Azure cloud resources — VMs, VMSS, SQL Database, Storage, AKS, App Service, Functions, VNet, Event Hubs, Container Apps, and Key Vault |
| `dt-obs-gcp` | GCP cloud resources — Compute Engine, GKE, Cloud Run, Pub/Sub, VPC networking, DNS, IAM, Secret Manager, and monitoring |
| `dt-obs-predictive-analytics` | Time series forecasting, capacity saturation planning, trend and anomaly detection across hosts, services, and infrastructure |
| `dt-obs-frontends` | RUM, Web Vitals, user sessions, mobile crashes, page performance, user interactions, and frontend errors |
| `dt-app-dashboards` | Create, modify, query, and analyze dashboard JSON — tiles, layouts, DQL queries, variables, and visualizations |
| `dt-app-notebooks` | Create, modify, query, and analyze notebook JSON — sections, DQL queries, visualizations, and markdown documentation |
| `dt-migration` | Migrate classic entity DQL and entityName/entityAttr/classicEntitySelector patterns to Smartscape equivalents |
| `dtctl` | CLI for Dynatrace resource lifecycle — apply, delete, share, history, restore; workflow execution; bulk and CI/CD automation |

---

### 2. MCP Server
**Source:** [github.com/dynatrace-oss/dynatrace-mcp](https://github.com/dynatrace-oss/dynatrace-mcp)
**Location:** `.vscode/mcp.json` (GitHub Copilot and Claude Code extension — VS Code) · `.mcp.json` (Claude Code CLI)

The Model Context Protocol (MCP) server is the live data bridge between Copilot and Dynatrace. When Copilot needs to answer a question about your environment, it calls the MCP server, which executes real API calls and DQL queries against your Dynatrace tenant and returns live results.

Two environments are configured as named servers:

```json
production-mcp  →  https://guu84124.apps.dynatrace.com        (production)
sprint-mcp  →  https://bon05374.sprint.apps.dynatracelabs.com  (sprint)
```

Authentication for a local MCP server uses OAuth browser SSO so no API tokens or credentials are stored in the workspace. To target a specific environment in a Copilot session:

```
"Use the production-mcp server for all queries"
```

---

### 3. Prompt Templates
**Source:** [github.com/Dynatrace/dynatrace-for-ai/prompts](https://github.com/Dynatrace/dynatrace-for-ai/tree/main/prompts)
**Location:** `.github/prompts/`

Prompts are pre-built investigation workflows saved as slash commands. They combine skills with structured instructions — telling the AI what to do, in what order, and with what guardrails. Type `/` in Copilot Chat or a Claude Code CLI session to see all available prompts.

**Locations:** `.github/prompts/` (Copilot Chat) · `.claude/commands/` (Claude Code CLI — symlinked from `.github/prompts/`, single source of truth)

| Prompt | Purpose | When to Use |
|---|---|---|
| `/health-check` | Service health snapshot | Routine morning check or before a deployment |
| `/daily-standup` | Multi-service report with today vs yesterday comparison | Team standup preparation |
| `/daily-standup-notebook` | Standup report + Dynatrace notebook + dtctl verification | Full documented standup workflow |
| `/investigate-error` | Error-focused investigation from a service name | "Something is wrong with this service" |
| `/troubleshoot-problem` | Structured 7-step deep dive into a specific problem | Known problem needing root cause |
| `/incident-response` | Full triage of all active problems by business impact | Active production incident |
| `/performance-regression` | Before vs after deployment comparison | Post-deployment validation |

#### The Investigation Workflow

Prompts are designed to chain together as an investigation deepens:

```
/daily-standup             Spot anomalies across services
       ↓
/health-check              Confirm which service has issues
       ↓
/investigate-error         Find the root cause
       ↓
/troubleshoot-problem      Deep-dive a specific problem
```

#### Key Guardrails Built Into Prompts

The `troubleshoot-problem` and `daily-standup-notebook` prompts encode operational rules learned from real usage:

- **Always start with problems** — never run broad log searches without problem context (hits 500GB scan limit)
- **No `#` or `--` comments in DQL** — invalid syntax that causes parse errors
- **`timeseries` uses `=` not `as`** for aliasing
- **`timeseries` filters use `==` with `by:` dimension** — not `contains()`
- **Array notation required** for computed fields after `timeseries`

---

### 4. Session Briefing Files
**Locations:** `.github/copilot-instructions.md` (GitHub Copilot) · `CLAUDE.md` (Claude Code — VS Code extension and CLI)

Both files are automatically loaded at the start of every AI session in this workspace. They act as a standing briefing — the AI already knows the default MCP environment, the investigation rule, and the available prompts before a single word is typed.

Each file contains:
- Default and fallback MCP server
- Global rule: always start with problems, never broad log searches
- Prompt directory — all 7 slash commands and when to use them
- Note that 16 skills are installed and load automatically

The two files are identical in content but kept separate because each tool requires a specific path:
- GitHub Copilot reads only `.github/copilot-instructions.md`
- Claude Code reads only `CLAUDE.md` at the repo root

---

### 5. dtctl CLI
**Source:** [github.com/dynatrace-oss/dtctl](https://github.com/dynatrace-oss/dtctl)
**Installation:** `/usr/local/bin/dtctl`

`dtctl` is a kubectl-style command-line tool for Dynatrace. It covers resource lifecycle operations (apply, delete, share, history, restore), workflow and analyzer execution, and bulk or CI/CD-style automation. Use it when operations aren't exposed via MCP or when scripting is the goal.

In this workspace, `dtctl` is commonly used for **verification** — confirming that notebooks and other artifacts created by AI workflows via MCP actually exist and are correctly structured in Dynatrace — but it is not limited to that role.

```bash
dtctl get notebooks                    # List all notebooks
dtctl describe notebook "name"         # Inspect notebook structure
dtctl query 'fetch dt.davis.problems   # Run DQL directly
  | filter event.status == "ACTIVE"
  | limit 5'
dtctl get workflows                    # List all workflows
dtctl doctor                           # Verify authentication and connectivity
```

Two authenticated contexts are configured:

```
production  (default)
sprint
```

Switch between them with:
```bash
dtctl config use-context production
dtctl config use-context sprint
```

---

## How It All Works Together

The same flow applies in Claude Code CLI: `CLAUDE.md` is loaded in place of `copilot-instructions.md`, and `/daily-standup-notebook` is invoked from the `claude` terminal session rather than Copilot Chat.

Here is the complete flow for a typical `/daily-standup-notebook` session:

```
1. You type /daily-standup-notebook in Copilot Chat

2. Copilot loads copilot-instructions.md
   → Knows to use production-mcp by default
   → Knows the investigation rules and DQL guardrails

3. Copilot loads relevant skills
   → dt-obs-services  (RED metrics)
   → dt-obs-problems  (active problems)
   → dt-app-notebooks (notebook structure)
   → dtctl            (verification commands)

4. Copilot calls production-mcp
   → Executes live DQL queries against production
   → Retrieves metrics, problems, and deployment data

5. Copilot generates the standup report
   → Today vs yesterday metric comparison
   → Active problems and incidents
   → Action items per service

6. Copilot creates a Dynatrace notebook via MCP
   → Executive summary
   → Per-service findings
   → Embedded live DQL queries
   → Prioritised remediation steps

7. dtctl verifies the notebook
   → dtctl get notebooks confirms it exists
   → dtctl describe notebook confirms structure
   → Shareable URL returned
```

---

## Keeping the Workspace Up to Date

Skills are versioned via `skills-lock.json`. Update to the latest skills before important demos or after Dynatrace releases new features:

```bash
npx skills add dynatrace/dynatrace-for-ai
npx skills add dynatrace-oss/dtctl
git add .
git commit -m "Update skills to latest — $(date +%Y-%m-%d)"
git push
```

Update `dtctl` by re-running the install script:

```bash
curl -fsSL https://raw.githubusercontent.com/dynatrace-oss/dtctl/main/install.sh | bash
```

For auditability, treat `skills-lock.json` hashes as the source-of-truth snapshot for committed skill content.

---

## Source References

| Component | Source |
|---|---|
| Dynatrace skills | [github.com/Dynatrace/dynatrace-for-ai](https://github.com/Dynatrace/dynatrace-for-ai) |
| Investigation prompts | [github.com/Dynatrace/dynatrace-for-ai/prompts](https://github.com/Dynatrace/dynatrace-for-ai/tree/main/prompts) |
| MCP server package | [github.com/dynatrace-oss/dynatrace-mcp](https://github.com/dynatrace-oss/dynatrace-mcp) |
| dtctl CLI + skill | [github.com/dynatrace-oss/dtctl](https://github.com/dynatrace-oss/dtctl) |
