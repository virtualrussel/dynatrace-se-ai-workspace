# Dynatrace SE AI Workspace — Cheat Sheet

**Use this to pick the right workflow, not write DQL.**

---

## When to use which workflow

| Situation | Start here | What you get back |
|-----------|------------|-------------------|
| Something is broken right now | `/incident-response` | All active problems prioritized by business impact, with mitigation steps and a shareable incident report |
| You have a specific problem ID | `/troubleshoot-problem` | 7-step RCA: scoped logs → error classification → trace timeline → root cause hypothesis |
| Error rates spiked after a deploy | `/performance-regression` | Before/after metrics comparison, bottleneck span, and a rollback or hotfix recommendation |
| Investigating error noise from a service | `/investigate-error [service-name]` | Top 3 error patterns with example logs, related traces, and remediation suggestions |
| Routine service health check | `/health-check [service-name]` | RED metrics, active problems, recent deployments, top slow endpoints, vulnerabilities |
| Morning catch-up across all services | `/daily-standup` | Per-service health status, today vs yesterday comparison, and action items |
| Standup + notebook for sharing | `/daily-standup-notebook` | Same as `/daily-standup` plus a Dynatrace notebook created and verified via dtctl |

---

## Workflow chaining

Most investigations follow this escalation path:

```
/daily-standup           → spot anomalies
  → /health-check        → confirm which service
    → /investigate-error → find the root cause
      → /troubleshoot-problem → deep-dive a specific problem
```

For live incidents, skip straight to `/incident-response` — it triages all active problems at once.

For deploys that went wrong, use `/performance-regression` — it compares before/after and recommends rollback or hotfix.

---

## Skills — what they unlock

Skills are loaded automatically when relevant. You can also ask for one directly.

| Ask about... | Skill used |
|---|---|
| Service error rates, latency, throughput, RED metrics, runtime monitoring (Java, .NET, Node.js) | `dt-obs-services` |
| Kubernetes pods, workloads, nodes, OOMKills, scheduling, security posture | `dt-obs-kubernetes` |
| Host CPU, memory, disk, network, containers, process-level telemetry | `dt-obs-hosts` |
| Frontend performance, Web Vitals, RUM, user sessions, mobile crashes, frontend errors | `dt-obs-frontends` |
| Distributed traces, spans, service dependencies, request flow analysis | `dt-obs-tracing` |
| Log querying, filtering, pattern analysis, error rates | `dt-obs-logs` |
| Problem RCA, root cause identification, impact assessment, DAVIS correlation | `dt-obs-problems` |
| AWS — EC2, RDS, Lambda, ECS/EKS, VPC, S3, DynamoDB, SQS/SNS, cost optimization | `dt-obs-aws` |
| Azure — VMs, AKS, App Service, Functions, VNet, Event Hubs, Container Apps, Key Vault | `dt-obs-azure` |
| GCP — Compute Engine, GKE, Cloud Run, Pub/Sub, VPC, IAM, Secret Manager | `dt-obs-gcp` |
| Time series forecasting, capacity saturation, trend and anomaly detection | `dt-obs-predictive-analytics` |
| Dashboards — create, modify, query, or analyze | `dt-app-dashboards` |
| Notebooks — create, modify, query, or analyze | `dt-app-notebooks` |
| Resource lifecycle, workflow execution, bulk/CI-CD automation via CLI | `dtctl` |
| Writing any DQL query | `dt-dql-essentials` (always load first) |

---

## dtctl — terminal companion

`dtctl` is the CLI-side of this workspace (all dtctl examples below are terminal commands). Use it when you want to verify, query, or manage resources from the terminal rather than through chat.

Compatibility: use `dtctl` v0.27.1 or newer. v0.27.0 adds post-apply hooks, new document API query flags (--filter, --sort, --add-fields, --admin-access), and breaking changes to settings addressing.

| Task | Command |
|------|---------|
| Verify connection | `dtctl doctor` |
| Run a DQL query | `dtctl query 'fetch dt.davis.problems \| filter event.status == "ACTIVE"'` |
| Verify DQL syntax only | `dtctl verify query --client-context "health-check" 'fetch dt.davis.problems \| limit 5'` |
| Run a context-tagged DQL query | `dtctl query --client-context "health-check" 'fetch dt.davis.problems \| filter event.status == "ACTIVE"'` |
| List workflows | `dtctl get workflows` |
| List notebooks | `dtctl get notebooks` |
| Filter notebook lookup precisely | `dtctl get notebooks --filter 'name == "<notebook-name>"' --sort "-modificationInfo.lastModifiedTime"` |
| Include extra document metadata | `dtctl get documents --add-fields "originExtensionId,labels,shareInfo.isShared"` |
| Switch environments | `dtctl config use-context production` / `dtctl config use-context sprint` |

v0.27.1 notes:
- `dtctl doctor` may show a user-identity warning for platform tokens while still passing overall checks. Grant `app-engine:apps:run` scope to resolve it fully.
- `--filter` on `get dashboards`/`get notebooks` now correctly restricts by type (previously returned mixed document types).
- `--add-fields` fields are now fully carried through (was silently dropped in v0.27.0).
- `--mine` now works with platform tokens (previously crashed with a JWT parse error).
- Settings object operations should use `objectId` from `dtctl get settings -o json`, not legacy synthetic UID values.

The AI workflows and dtctl point at the same environment — use chat for investigation, dtctl for spot-checks and verification.

---

## When workflows stop short

These are expected behaviours, not errors:

| What you see | Why | What to do |
|---|---|---|
| "No active problems found" | No problems in last 7 days | Widen to recently closed problems, or check you're on the right environment |
| "No regression threshold exceeded" | `/performance-regression` found no signal | Trust the result — or re-run with a narrower timeframe if you suspect a specific window |
| "Could not correlate to a local file" | `/performance-regression` found the bottleneck span but no matching code | The slow code may be in a dependency or a service not in this workspace |
| Query returns 0 results | Filters too tight, or wrong entity | Verify the service name with `/health-check` first |
| 500GB scan limit hit | Log query too broad | A workflow should have scoped it — if writing your own query, add entity filter + tighter timeframe |

---

## Key rules

**Start with problems — never open-ended log searches.**
All workflows enforce this. If you skip it, you'll hit the 500GB scan limit.

**Scope everything to a problem timeframe.**
Workflows extract the timeframe automatically. If you're writing your own query, use ±5 min around the incident window.

**Let the workflow drive DQL — don't write queries from scratch.**
Ask: *"What were the error patterns during the last problem on [service]?"* — not *"Write a DQL query for..."*

---

## Session targeting

Default environment: `production-mcp` → https://guu84124.apps.dynatrace.com

To switch for the session:

This is a chat instruction to the AI assistant (not a terminal command).

```
# Switch to sprint for this session
"Use the sprint-mcp server for all queries in this session"

# Switch back to production for this session
"Use the production-mcp server for all queries in this session"
```

---

## Useful natural language prompts

These work well when you're not sure which workflow to reach for:

- *"Are there any active problems right now?"*
- *"What happened to [service] in the last hour?"*
- *"Show me the slowest endpoints for [service] since the last deployment"*
- *"Create a notebook summarising today's incidents"*
- *"What's the root cause of problem [ID]?"*
- *"Compare [service] performance before and after [deploy time]"*

---

**Default MCP:** production-mcp | **Last Updated:** May 5, 2026
