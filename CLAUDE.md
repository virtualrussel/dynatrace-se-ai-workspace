# Dynatrace AI Workspace — Session Briefing

## Environment

| | |
|---|---|
| **Default MCP server** | `production-mcp` → https://guu84124.apps.dynatrace.com |
| **Fallback MCP server** | `sprint-mcp` → https://bon05374.sprint.apps.dynatracelabs.com |

To target a specific environment for a session:
```
"Use the sprint-mcp server for all queries in this session"
```

## Global Rule

**Always start with problems — never run broad log searches.**
Broad queries without problem context hit Dynatrace's 500GB scan limit and return zero results.
All investigation workflows enforce this automatically.

## Prompts

Type `@` to access these slash commands:

| Prompt | When to use |
|---|---|
| `/health-check` | Routine service health — metrics, problems, deployments, vulnerabilities |
| `/daily-standup` | Morning report across services — today vs yesterday comparison |
| `/daily-standup-notebook` | Standup report + Dynatrace notebook creation + dtctl verification |
| `/investigate-error` | Error-focused investigation from a service name |
| `/troubleshoot-problem` | Deep 7-step investigation into a specific Dynatrace problem |
| `/incident-response` | Full triage of all active problems during a live incident |
| `/performance-regression` | Before vs after deployment comparison with rollback/hotfix recommendation |

## Skills

13 domain knowledge skills are installed in `.agents/skills/`. They load automatically when relevant — no manual loading required.

## dtctl

`dtctl` is a kubectl-style CLI for terminal-level verification and resource management. Use it to confirm what AI workflows create via MCP.

```bash
dtctl doctor                             # verify auth and connectivity
dtctl get notebooks                      # list notebooks
dtctl describe notebook "name"           # inspect structure
dtctl query 'fetch dt.davis.problems | filter event.status == "ACTIVE" | limit 5'
dtctl get workflows
dtctl config use-context production      # switch environments
dtctl config use-context sprint
```

Two authenticated contexts are configured: `production` (default) and `sprint`.
