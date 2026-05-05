# Dynatrace SE AI Workspace — Session Briefing

## Environment

| | |
|---|---|
| **Default MCP server** | `production-mcp` → https://guu84124.apps.dynatrace.com |
| **Fallback MCP server** | `sprint-mcp` → https://bon05374.sprint.apps.dynatracelabs.com |

To target a specific environment for a session:
```
"Use the production-mcp server for all queries in this session"
```

## Global Rule

**Always start with problems — never run broad log searches.**
Broad queries without problem context hit Dynatrace's 500GB scan limit and return zero results.
All investigation workflows enforce this automatically.

## Prompts

Type `/` in Copilot Chat to access these slash commands:

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

Domain knowledge skills are installed in `.agents/skills/`. They load automatically when relevant — no manual loading required.
