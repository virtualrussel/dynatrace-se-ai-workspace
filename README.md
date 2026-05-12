# dynatrace-se-ai-workspace

An AI-powered observability workspace for Dynatrace that combines GitHub Copilot or Claude AI, the Dynatrace MCP server, dtctl, and the [dynatrace-for-ai](https://github.com/Dynatrace/dynatrace-for-ai) skills framework to accelerate incident triage, root cause analysis, and day-to-day observability workflows.

> **What this gives you:** Ask AI natural language questions about your Dynatrace environment and get accurate, production-aware answers. All powered by verified domain knowledge, live API access, and pre-built investigation workflows.

> **New here?** Start with [docs/ELI5.md](./docs/ELI5.md) for a quick setup, then read [docs/OVERVIEW.md](./docs/OVERVIEW.md) for the big-picture operating model.

---

## What's Inside

Recommended reading order: [docs/ELI5.md](./docs/ELI5.md) → [docs/OVERVIEW.md](./docs/OVERVIEW.md) → [ARCHITECTURE.md](./ARCHITECTURE.md).

```
dynatrace-se-ai-workspace/
├── README.md                     # Setup guide and quick reference
├── llms.txt                      # Machine-readable workspace summary for LLMs
├── docs/
│   ├── ELI5.md                   # Beginner-friendly 15-minute install guide
│   ├── OVERVIEW.md               # Newcomer guide: purpose, value, and operating model
│   └── CHEATSHEET.md             # Quick reference — workflows, outputs, dtctl, and key rules
├── ARCHITECTURE.md               # How the workspace is built and how components connect
├── CONTRIBUTING.md               # How to update skills, prompts, and MCP config
├── CLAUDE.md                     # Auto-loaded session briefing for Claude Code
├── skills-lock.json              # Locked skill versions
├── LICENSE
├── .gitignore
├── .github/
│   ├── copilot-instructions.md   # Auto-loaded session briefing for GitHub Copilot
│   └── prompts/                  # 7 investigation workflows
│       ├── health-check.prompt.md
│       ├── daily-standup.prompt.md
│       ├── daily-standup-notebook.prompt.md
│       ├── investigate-error.prompt.md
│       ├── troubleshoot-problem.prompt.md
│       ├── incident-response.prompt.md
│       └── performance-regression.prompt.md
├── .agents/skills/               # 16 Dynatrace domain skills
├── .claude/skills/               # Symlinks for Claude Code compatibility
├── .mcp.json                     # MCP server configuration for Copilot CLI
├── .vscode/
│   ├── mcp.json                  # MCP server configuration for VS Code Copilot
│   ├── extensions.json           # Recommended VS Code extensions
│   └── settings.json             # Workspace editor settings
└── demos/
    └── ai-observability-demo.md  # Demo script
```

| Tool | Purpose |
|---|---|
| [VS Code](https://code.visualstudio.com/) | Editor with Copilot/Claude Chat |
| [GitHub Copilot](https://github.com/features/copilot) | AI assistant (option 1) |
| [Claude Code](https://claude.ai/code) | AI assistant (option 2) |
| [Node.js](https://nodejs.org/) v18+ | Required to run the MCP server |
| [dtctl](https://github.com/dynatrace-oss/dtctl) | Dynatrace open-source CLI for agents & humans to manage observability resources (use v0.27.1 or newer) |
| A Dynatrace environment | `https://<env>.apps.dynatrace.com` or `https://<env>.sprint.apps.dynatracelabs.com` |

You must use one AI assistant path: **GitHub Copilot** or **Claude Code**.

---

## Setup

> **Dynatrace employees & partners:** This workspace is pre-configured for the 
> Dynatrace demo environment (`guu84124.apps.dynatrace.com`). No changes are
> required to run demonstrations against the production demo tenant. Clone the repo,
> run `dtctl auth login --context production --environment "https://guu84124.apps.dynatrace.com"`,
> reload VS Code, and authenticate via your Dynatrace SSO when prompted.

### Choose Your Frontend

This workspace works with:
- **GitHub Copilot** in VS Code (subscription required)
- **Claude Code** via web or desktop (Claude Pro or Team required)

Select your setup path below. Both receive the same skills, prompts, and MCP server access.

**GitHub Copilot Path** → Follow Steps 1–6 below. `.github/copilot-instructions.md` is auto-loaded at the start of each Copilot session.

**Claude Code Path** → Follow Steps 1–6 below. `CLAUDE.md` is auto-loaded at the start of each Claude Code session.

### 1. Clone the workspace

```bash
git clone https://github.com/virtualrussel/dynatrace-se-ai-workspace.git
cd dynatrace-se-ai-workspace
```

Then open the folder in VS Code via **File → Open Folder**.

### 2. Update skills to latest *(optional)*

If this is your first setup, skip this step and continue to Step 3.

Skills are already included in this repo — cloning gives you everything you need. Run this only when you want to pull the latest skill updates from Dynatrace:

```bash
npx skills add dynatrace/dynatrace-for-ai
npx skills add dynatrace-oss/dtctl
```

> See [Keeping Up to Date](#keeping-up-to-date) for when to run this.

### 3. Configure dtctl for the demo tenant

`dtctl` handles resource lifecycle (apply, delete, share, history, restore), workflow and analyzer execution, and bulk or CI/CD-style automation from the terminal. It is required for demo workflows in this workspace.

> Compatibility note: use `dtctl` v0.27.1 or newer.

```bash
# macOS / Linux — direct install (no package manager required)
curl -fsSL https://raw.githubusercontent.com/dynatrace-oss/dtctl/main/install.sh | bash

# Local desktop (macOS/Windows/Linux with keyring): OAuth login
dtctl auth login --context production \
  --environment "https://guu84124.apps.dynatrace.com"

# GitHub Codespaces / CI: token-based auth
dtctl config set-context production \
  --environment "https://guu84124.apps.dynatrace.com" \
  --token-ref production-token
dtctl config set-credentials production-token --token <YOUR_PLATFORM_TOKEN>

# Verify
dtctl doctor
```

Create your platform token at https://myaccount.dynatrace.com/platformTokens (Account Management portal — not the IAM Access Tokens page, which manages classic `dt0c01.*` tokens).

If you are in Codespaces and see `keyring probe failed` or `dbus-launch` errors, skip OAuth and use token-based auth.

### 4. Configure your sprint environment (optional)

The workspace is pre-configured with two MCP servers — the shared demo tenant
(`guu84124`) and a secondary sprint tenant (`bon05374`). The `bon05374` entry is
specific to the original author and you will need to replace it with your own tenant ID if you want
to connect a second environment.

> If you only need the shared demo tenant (`guu84124`), skip this section entirely. No additional configuration is required.

Complete all four steps below to configure your own secondary tenant. Skipping
any step will result in Copilot referencing a server that doesn't exist or
authenticating against the wrong environment.

#### Sprint Tenant Checklist

**Step 4.A — Update `.vscode/mcp.json`**

Replace `<your-tenant-id>` with your personal sprint tenant ID (e.g. `abc12345`):

```json
{
  "servers": {
    "production-mcp": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@dynatrace-oss/dynatrace-mcp-server@latest", "--stdio"],
      "env": {
        "DT_ENVIRONMENT": "https://guu84124.apps.dynatrace.com"
      }
    },
    "sprint-mcp": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@dynatrace-oss/dynatrace-mcp-server@latest", "--stdio"],
      "env": {
        "DT_ENVIRONMENT": "https://<your-tenant-id>.sprint.apps.dynatracelabs.com"
      }
    }
  }
}
```

**Step 4.B — Update `.mcp.json`**

`.mcp.json` is used by Copilot CLI and must stay in sync with `.vscode/mcp.json`.
Run this command to regenerate it from your updated `.vscode/mcp.json`:

```bash
jq "{mcpServers: .servers}" .vscode/mcp.json > .mcp.json
```

Verify the output looks correct before continuing:

```bash
cat .mcp.json
```

You should see both MCP server entries with your sprint tenant ID in place.

**Step 4.C — Update `.github/copilot-instructions.md` and `CLAUDE.md`**

Find the Environment table in both files and update the fallback server URL to match your tenant ID:

```
| **Fallback MCP server** | `sprint-mcp` → https://<your-tenant-id>.sprint.apps.dynatracelabs.com |
```

Both `.github/copilot-instructions.md` (GitHub Copilot) and `CLAUDE.md` (Claude Code) must be updated with your tenant ID or they will reference the original author's sprint environment.

**Step 4.D — Authenticate dtctl**

```bash
# Local desktop (macOS/Windows/Linux with keyring): OAuth login
dtctl auth login --context sprint \
  --environment "https://<your-tenant-id>.sprint.apps.dynatracelabs.com"

# GitHub Codespaces / CI: token-based auth
dtctl config set-context sprint \
  --environment "https://<your-tenant-id>.sprint.apps.dynatracelabs.com" \
  --token-ref sprint-token
dtctl config set-credentials sprint-token --token <YOUR_PLATFORM_TOKEN>
```

If OAuth fails with a keyring error (for example, `dbus-launch` not found), use the token-based method above.

Both contexts are now configured. Switch between them with:
```bash
dtctl config use-context production
dtctl config use-context sprint
```

### 5. Reload VS Code

Press `Cmd+Shift+P` → `Developer: Reload Window`

When you first use a prompt in Copilot Chat, a browser window will open for
Dynatrace SSO authentication. This is expected — complete the login and return
to VS Code. Subsequent sessions authenticate automatically.

### 6. Verify the connection

**GitHub Copilot users:** In Copilot Chat, type:

```
Using the production-mcp server, list the top 5 services by request volume in the last hour
```

**Claude Code users:** In Claude Code, type the same query or copy it from the GitHub Copilot instruction above.

If you see a table of services with request counts — you are live and ready to demo.

---

## Skills

Skills are domain knowledge files that teach Copilot how Dynatrace works — correct DQL syntax, field names, query patterns, and investigation workflows. They load automatically when relevant.

Skills follow the [Agent Skills specification](https://agentskills.io/specification) and use progressive disclosure:

1. Catalog - Agents load only `name` + `description` (~100 tokens per skill) to know what's available.
2. Instructions - When relevant, the full `SKILL.md` is loaded (<5000 tokens).
3. Resources - Detailed reference files in `references/` are loaded on demand.

| Skill | What It Covers |
|---|---|
| `dt-dql-essentials` | Core DQL syntax rules, common pitfalls, and query patterns — **load before writing any DQL** |
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

## Prompts

Prompts are pre-built investigation workflows available as slash commands.

- **GitHub Copilot:** Type `/` in Copilot Chat (see `.github/prompts/`)
- **Claude Code:** Type `@` followed by the prompt name (e.g. `@health-check`)

| Prompt | When to Use |
|---|---|
| `/health-check` | Routine service health — performance, problems, deployments, vulnerabilities |
| `/daily-standup` | Morning team report across multiple services with today vs yesterday comparison |
| `/daily-standup-notebook` | Standup report + Dynatrace notebook creation + dtctl verification |
| `/investigate-error` | "Something is wrong with this service" — error-focused investigation |
| `/troubleshoot-problem` | Deep 7-step investigation into a specific Dynatrace problem |
| `/incident-response` | Full production incident triage — all active problems, prioritized by business impact |
| `/performance-regression` | Did my deployment cause a slowdown? Before vs after comparison with trace analysis |

### Investigation Workflow

The prompts follow a structured drill-down pattern:

```
/daily-standup           →  spot anomalies across services
  → /health-check        →  confirm which service has issues
    → /investigate-error →  find the root cause
      → /troubleshoot-problem →  deep-dive a specific problem
```

---

## Key Concepts

### Why Skills Matter

Copilot without skills will guess DQL syntax — and get it wrong. For example, it might use `event.status == "OPEN"` (doesn't exist) instead of `event.status == "ACTIVE"`, or `log.level` instead of `loglevel`. The skills encode the corrections for known failure modes before Copilot writes a single query.

### How MCP Works

The Dynatrace MCP server gives Copilot live API access to your environment. When you run `/health-check`, Copilot calls the MCP server to execute real DQL queries and return live data — not cached or synthetic results.

### The Investigation Rule

**Always start with problems, never with broad log searches.** Broad log queries without a problem context will hit Dynatrace's 500GB scan limit and return zero results. The prompts enforce this automatically.

### MCP Configuration Files

This workspace maintains two MCP configuration files that must be kept in sync:

| File | Used By |
|---|---|
| `.vscode/mcp.json` | VS Code GitHub Copilot and Claude Code |
| `.mcp.json` | GitHub Copilot CLI |

When adding or updating MCP servers, always update both files. Regenerate `.mcp.json` from `.vscode/mcp.json` using:

```bash
jq "{mcpServers: .servers}" .vscode/mcp.json > .mcp.json
```

---

## dtctl CLI

[dtctl](https://github.com/dynatrace-oss/dtctl) is a kubectl-style CLI for Dynatrace that complements this workspace — giving you terminal-level access to run DQL queries, manage workflows, verify notebooks, and more.

```bash
# macOS / Linux — direct install (no package manager required)
curl -fsSL https://raw.githubusercontent.com/dynatrace-oss/dtctl/main/install.sh | bash

# Local desktop (macOS/Windows/Linux with keyring): OAuth login
dtctl auth login --context production \
  --environment "https://guu84124.apps.dynatrace.com"

# GitHub Codespaces / CI: token-based auth
dtctl config set-context production \
  --environment "https://guu84124.apps.dynatrace.com" \
  --token-ref production-token
dtctl config set-credentials production-token --token <YOUR_PLATFORM_TOKEN>

# Verify
dtctl doctor

# Example commands
dtctl get workflows
dtctl get notebooks
dtctl query 'fetch dt.davis.problems | filter event.status == "ACTIVE" | limit 5'
dtctl query --client-context "incident-response" 'fetch dt.davis.problems | filter event.status == "ACTIVE" | limit 5'
dtctl verify query --client-context "incident-response" 'fetch dt.davis.problems | limit 5'
dtctl get notebooks --filter 'name == "<notebook-name>"' --sort "-modificationInfo.lastModifiedTime"
dtctl get documents --add-fields "originExtensionId,labels,shareInfo.isShared"

# Switch between configured contexts
dtctl config use-context production
dtctl config use-context sprint
```

---

## Keeping Up to Date

```bash
# Update all skills to latest
npx skills add dynatrace/dynatrace-for-ai
npx skills add dynatrace-oss/dtctl

# Regenerate .mcp.json after any MCP server changes
jq "{mcpServers: .servers}" .vscode/mcp.json > .mcp.json

# Commit the updates
git add .
git commit -m "Update skills to latest"
git push
```

### dtctl Version Transition Record

- v0.26.x → v0.27.0 (May 2026, commit `f857a23`): document API query flags, settings addressing, hook syntax changes
- v0.27.0 → v0.27.1 (May 2026): `--filter` type constraint fix, `--add-fields` carry-through fix, `--mine` platform token fix, anomaly-detector round-trip, coloring guidance rewrite
- Current baseline: v0.27.1 or newer

Note: `skills-lock.json` records source and content hash snapshots for skills, not a separate semantic `dtctlVersion` field.

---

## Related Resources

- [ARCHITECTURE.md](./ARCHITECTURE.md) — How the workspace components connect
- [docs/ELI5.md](./docs/ELI5.md) — Beginner-friendly 15-minute quick start
- [docs/CHEATSHEET.md](./docs/CHEATSHEET.md) — Workflow picker and operational quick reference
- [docs/OVERVIEW.md](./docs/OVERVIEW.md) — Business and operator-oriented purpose guide
- [dynatrace-for-ai](https://github.com/Dynatrace/dynatrace-for-ai) — Skills and prompts source repo
- [dtctl](https://github.com/dynatrace-oss/dtctl) — Dynatrace CLI for humans and AI agents
- [Dynatrace MCP Server](https://docs.dynatrace.com/docs/shortlink/dynatrace-mcp-server) — Official MCP server docs
- [Agent Skills spec](https://agentskills.io) — The open standard this workspace follows

---

## License

Skills and prompts sourced from [dynatrace-for-ai](https://github.com/Dynatrace/dynatrace-for-ai) and [dtctl](https://github.com/dynatrace-oss/dtctl) are Apache-2.0 licensed.
