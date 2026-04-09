# dynatrace-ai-workspace

An AI-powered observability workspace for Dynatrace — combining GitHub Copilot, the Dynatrace MCP server, and the [dynatrace-for-ai](https://github.com/Dynatrace/dynatrace-for-ai) skills framework to accelerate incident triage, root cause analysis, and day-to-day observability workflows.

> **What this gives you:** Ask Copilot natural language questions about your Dynatrace environment and get accurate, production-aware answers — powered by verified domain knowledge, live API access, and pre-built investigation workflows.

---

## What's Inside

```
dynatrace-ai-workspace/
├── .github/
│   ├── copilot-instructions.md   # Auto-loaded workspace rules every Copilot session
│   └── prompts/                  # 6 slash command investigation workflows
├── .agents/skills/               # 13 Dynatrace domain knowledge files
├── .claude/skills/               # Symlinks for Claude Code compatibility
├── .vscode/
│   └── mcp.json                  # Dynatrace MCP server configuration
└── skills-lock.json              # Locked skill versions
```

---

## Prerequisites

| Tool | Purpose |
|---|---|
| [VS Code](https://code.visualstudio.com/) | Editor with Copilot Chat |
| [GitHub Copilot](https://github.com/features/copilot) | AI assistant (subscription required) |
| [Node.js](https://nodejs.org/) v18+ | Required for skills installer and MCP server |
| [dtctl](https://github.com/dynatrace-oss/dtctl) | Dynatrace CLI for notebook verification and resource management |
| A Dynatrace environment | `https://<env>.apps.dynatrace.com` |

---

## Setup

### 1. Clone the workspace

```bash
git clone git@github.com:<your-username>/dynatrace-ai-workspace.git
cd dynatrace-ai-workspace
code .
```

### 2. Update skills to latest

```bash
npx skills add dynatrace/dynatrace-for-ai
npx skills add dynatrace-oss/dtctl
```

> Run this command any time Dynatrace releases new skills.

### 3. Configure your Dynatrace environment

Edit `.vscode/mcp.json` and replace the placeholder URLs with your environment:

```json
{
  "servers": {
    "my-env-mcp": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@dynatrace-oss/dynatrace-mcp-server@latest", "--stdio"],
      "env": {
        "DT_ENVIRONMENT": "https://<your-env>.apps.dynatrace.com"
      }
    }
  }
}
```

### 4. Reload VS Code

Press `Cmd+Shift+P` → `Developer: Reload Window`

Copilot will authenticate via browser SSO when you first use a prompt.

### 5. Install and authenticate dtctl

```bash
# macOS — direct install (no Homebrew required)
curl -fsSL https://raw.githubusercontent.com/dynatrace-oss/dtctl/main/install.sh | bash

# Authenticate against your environment
dtctl auth login --context my-env \
  --environment "https://<your-env>.apps.dynatrace.com"

# Verify
dtctl doctor
```

---

## Skills

Skills are domain knowledge files that teach Copilot how Dynatrace works — correct DQL syntax, field names, query patterns, and investigation workflows. They load automatically when relevant.

| Skill | What It Covers |
|---|---|
| `dt-dql-essentials` | DQL syntax, common pitfalls, query patterns — **load before any DQL** |
| `dt-obs-problems` | Davis Problems, root cause analysis, impact assessment |
| `dt-obs-logs` | Log queries, filtering, pattern analysis, error classification |
| `dt-obs-tracing` | Distributed traces, spans, failure detection, log correlation |
| `dt-obs-services` | RED metrics, SLA tracking, runtime-specific monitoring (Java, .NET, Node.js, Python, PHP, Go) |
| `dt-obs-hosts` | Host and process metrics, CPU, memory, disk, containers |
| `dt-obs-kubernetes` | Pods, workloads, nodes, labels, ingress, PVCs |
| `dt-obs-aws` | EC2, RDS, Lambda, ECS/EKS, VPC, cost optimization |
| `dt-obs-frontends` | RUM, Web Vitals, user sessions, mobile crashes |
| `dt-app-dashboards` | Dashboard JSON creation and modification |
| `dt-app-notebooks` | Notebook creation and analytics workflows |
| `dt-migration` | Classic entity DQL → Smartscape migration |
| `dtctl` | CLI commands for managing Dynatrace resources from the terminal |

---

## Prompts

Prompts are pre-built investigation workflows available as Copilot slash commands. Type `/` in Copilot Chat to see them.

| Prompt | When to Use |
|---|---|
| `/health-check` | Routine service health — performance, problems, deployments, vulnerabilities |
| `/daily-standup` | Morning team report across multiple services with today vs yesterday comparison |
| `/investigate-error` | "Something is wrong with this service" — error-focused investigation |
| `/troubleshoot-problem` | Deep 7-step investigation into a specific Dynatrace problem |
| `/incident-response` | Full production incident triage — all active problems, prioritized by business impact |
| `/performance-regression` | Did my deployment cause a slowdown? Before vs after comparison with trace analysis |

### Investigation Workflow

The prompts follow a structured drill-down pattern:

```
/health-check          →  flag concerns
/troubleshoot-problem  →  investigate a specific problem
/investigate-error     →  drill into error patterns
/incident-response     →  full triage when things are on fire
```

---

## Key Concepts

### Why Skills Matter

Copilot without skills will guess DQL syntax — and get it wrong. For example, it might use `event.status == "OPEN"` (doesn't exist) instead of `event.status == "ACTIVE"`, or `log.level` instead of `loglevel`. The skills encode the corrections for known failure modes before Copilot writes a single query.

### How MCP Works

The Dynatrace MCP server gives Copilot live API access to your environment. When you run `/health-check`, Copilot calls the MCP server to execute real DQL queries and return live data — not cached or synthetic results.

### The Investigation Rule

**Always start with problems, never with broad log searches.** Broad log queries without a problem context will hit Dynatrace's 500GB scan limit and return zero results. The prompts enforce this automatically.

---

## Optional: dtctl CLI

[dtctl](https://github.com/dynatrace-oss/dtctl) is a kubectl-style CLI for Dynatrace that complements this workspace — giving you terminal-level access to run DQL queries, manage workflows, edit dashboards, and more.

```bash
# macOS (direct install)
curl -fsSL https://raw.githubusercontent.com/dynatrace-oss/dtctl/main/install.sh | bash

# Authenticate
dtctl auth login --context my-env \
  --environment "https://<your-env>.apps.dynatrace.com"

# Verify
dtctl doctor

# Example commands
dtctl get workflows
dtctl query 'fetch dt.davis.problems | filter event.status == "ACTIVE" | limit 5'
```

---

## Keeping Up to Date

```bash
# Update all skills to latest
npx skills add dynatrace/dynatrace-for-ai
npx skills add dynatrace-oss/dtctl

# Commit the updates
git add .
git commit -m "Update skills to latest"
git push
```

---

## Related Resources

- [dynatrace-for-ai](https://github.com/Dynatrace/dynatrace-for-ai) — Skills and prompts source repo
- [dtctl](https://github.com/dynatrace-oss/dtctl) — Dynatrace CLI for humans and AI agents
- [Dynatrace MCP Server](https://docs.dynatrace.com/docs/shortlink/dynatrace-mcp-server) — Official MCP server docs
- [Agent Skills spec](https://agentskills.io) — The open standard this workspace follows

---

## License

Skills and prompts sourced from [dynatrace-for-ai](https://github.com/Dynatrace/dynatrace-for-ai) and [dtctl](https://github.com/dynatrace-oss/dtctl) are Apache-2.0 licensed.
