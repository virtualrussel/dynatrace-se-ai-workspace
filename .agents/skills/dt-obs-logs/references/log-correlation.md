# Log Correlation

Correlate log data with distributed traces, DAVIS problems, and other telemetry signals for unified investigation.

## Logs and Traces Correlation

Logs enriched with `trace_id` and `span_id` by OneAgent enable direct correlation with distributed traces.

### Find Logs Containing Trace Context

```dql
fetch logs, from:now() - 2h
| filter isNotNull(trace_id)
| fields timestamp, status, content, trace_id, span_id
| sort timestamp desc
| limit 50
```

### Find Logs for a Specific Trace

```dql
fetch logs, from:now() - 2h
| filter trace_id == "your-trace-id-here"
| fields timestamp, status, content, span_id
| sort timestamp asc
```

### Find Traces that Produced Error Logs

```dql
fetch spans, from:now() - 30m
| filter trace.id in [
    fetch logs
    | filter isNotNull(trace_id)
    | filter status == "ERROR"
    | fields toUid(trace_id)
]
| limit 50
```

**Note**: Subqueries in `in` statements have size limits. If the subquery result exceeds the limit, you'll get an `IN_KEYWORD_TABLE_SIZE` DQL error — pre-filter the logs more aggressively before the subquery.

### Find Traces Associated with Specific Log Content

```dql
fetch spans, from:now() - 30m
| filter trace.id in [
    fetch logs
    | filter isNotNull(trace_id)
    | filter contains(content, "payment failed")
    | fields toUid(trace_id)
]
| fields span.name, duration, http.status_code, trace.id
| sort duration desc
| limit 20
```

### Join Spans and Logs on Trace ID

```dql
fetch spans, from:now() - 1h
| join [ fetch logs | fieldsAdd trace.id = toUid(trace_id) ]
  , on: { trace.id }
  , fields: { content, status }
| fields timestamp, trace.id, span.id, span.name, status, content
| sort timestamp desc
| limit 100
```

### Find Spans Associated with Specific Span-Level Logs

```dql
fetch spans, from:now() - 30m
| filter span.id in [
    fetch logs
    | filter isNotNull(span_id)
    | filter contains(content, "slow query detected")
    | fields toUid(span_id)
]
| fields span.name, duration, db.statement, trace.id
| sort duration desc
| limit 20
```

## Logs and Problems Correlation

Correlate error logs with DAVIS-detected problems to understand what logs accompanied a problem event.

### Logs During Active Problems

Find error logs from services with active problems in the same window:

```dql
fetch logs, from:now() - 1h
| filter status == "ERROR"
| fieldsAdd process_group = dt.process_group.detected_name
| filter isNotNull(process_group)
| summarize error_count = count(), by: {process_group}
| sort error_count desc
```

Cross-reference with:

```dql
fetch dt.davis.problems, from:now() - 1h
| filter event.status == "ACTIVE"
| fields event.name, event.category, display_id, event.start
| sort event.start desc
```

### Logs in a Problem's Time Window

Investigate logs during a specific problem's active period by its start time:

```dql
fetch logs, from: "2024-01-15T10:00:00Z", to: "2024-01-15T11:00:00Z"
| filter status == "ERROR"
| fieldsAdd process_group = dt.process_group.detected_name
| filter contains(process_group, "payment")
| fields timestamp, status, content, process_group
| sort timestamp asc
| limit 200
```

### Error Log Spike Aligned with Problem Detection

```dql
fetch logs, from:now() - 4h
| filter status == "ERROR"
| summarize error_count = count(), by: {time_bucket = bin(timestamp, 5m)}
| sort time_bucket asc
```

Use alongside:

```dql
fetch dt.davis.problems, from:now() - 4h
| filter not(dt.davis.is_duplicate)
| fields event.name, event.start, event.end, event.status, event.category
| sort event.start asc
```

## Logs and Metrics Correlation

### Service Log Errors vs. Request Failure Rate

Identify services where log errors correspond to elevated request failure rates by querying them side by side.

**Step 1** — Error log count per service:

```dql
fetch logs, from:now() - 1h
| filter status == "ERROR"
| summarize log_errors = count(), by: {process_group = dt.process_group.detected_name}
| sort log_errors desc
```

**Step 2** — Trace-level failure rate for those services:

```dql
fetch spans, from:now() - 1h
| filter request.is_root_span == true
| summarize
    total = count(),
    failed = countIf(request.is_failed == true),
    by: {service = getNodeName(dt.smartscape.service)}
| fieldsAdd failure_rate = (failed * 100.0) / total
| sort failure_rate desc
```

## Kubernetes Context Correlation

If log fields include Kubernetes metadata, correlate logs to pods and namespaces:

### Logs by Namespace

```dql
fetch logs, from:now() - 1h
| filter status == "ERROR"
| filter isNotNull(k8s.namespace.name)
| summarize error_count = count(), by: {namespace = k8s.namespace.name, pod = k8s.pod.name}
| sort error_count desc
| limit 20
```

### Logs for a Specific Pod

```dql
fetch logs, from:now() - 1h
| filter k8s.pod.name == "payment-service-abc123"
| filter status == "ERROR"
| fields timestamp, content, status
| sort timestamp desc
| limit 100
```
