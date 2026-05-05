# Log Querying

Core patterns for fetching, filtering, and navigating Dynatrace log data with DQL.

## Key Fields

| Field | Description | Example Value |
|-------|-------------|---------------|
| `timestamp` | Log entry creation time | `2024-01-15T10:30:00Z` |
| `content` | Raw log message text | `"ERROR: Connection refused"` |
| `status` | Log severity level | `ERROR`, `WARN`, `INFO`, `DEBUG` |
| `dt.process_group.id` | Process group entity ID | `PROCESS_GROUP-abc123` |
| `dt.process_group.detected_name` | Human-readable process group name | `"payment-service"` |
| `trace_id` | Distributed trace correlation ID | `"abc123def456"` |
| `span_id` | Individual span correlation ID | `"789xyz"` |

## Basic Log Fetch

### Last N Logs

```dql
fetch logs, from:now() - 1h
| fields timestamp, status, content
| sort timestamp desc
| limit 50
```

### Logs with Entity Names

```dql
fetch logs, from:now() - 1h
| fields timestamp, status, content,
    process_group = dt.process_group.detected_name
| sort timestamp desc
| limit 100
```

## Severity Filtering

### Single Severity Level

```dql
fetch logs, from:now() - 1h
| filter status == "ERROR"
| fields timestamp, content, process_group = dt.process_group.detected_name
| sort timestamp desc
| limit 100
```

### Multiple Severity Levels

```dql
fetch logs, from:now() - 2h
| filter in(status, {"ERROR", "FATAL", "WARN"})
| fields timestamp, status, content, process_group = dt.process_group.detected_name
| sort timestamp desc
| limit 200
```

### Excluding a Severity Level

```dql
fetch logs, from:now() - 1h
| filter status != "INFO" and status != "DEBUG"
| fields timestamp, status, content
| sort timestamp desc
| limit 100
```

## Content Search

### Simple Substring Search

```dql
fetch logs, from:now() - 1h
| filter contains(content, "database")
| fields timestamp, content, status
| sort timestamp desc
| limit 50
```

### Case-Insensitive Search

```dql
fetch logs, from:now() - 1h
| filter contains(toLower(content), "nullpointerexception")
| fields timestamp, content, status
| sort timestamp desc
| limit 50
```

### Full-Text Phrase Search

```dql
fetch logs, from:now() - 1h
| filter matchesPhrase(content, "connection timeout")
| fields timestamp, content, process_group = dt.process_group.detected_name
| sort timestamp desc
| limit 50
```

### Multiple Keyword Search (OR)

```dql
fetch logs, from:now() - 1h
| filter contains(content, "exception") or contains(content, "timeout") or contains(content, "refused")
| fields timestamp, content, status
| sort timestamp desc
| limit 100
```

### Multiple Keyword Search (AND)

```dql
fetch logs, from:now() - 1h
| filter contains(content, "payment") and contains(content, "failed")
| fields timestamp, content, status
| sort timestamp desc
| limit 50
```

## Entity-Based Filtering

### Filter by Process Group Name

```dql
fetch logs, from:now() - 1h
| fieldsAdd process_group = dt.process_group.detected_name
| filter process_group == "payment-service"
| fields timestamp, status, content
| sort timestamp desc
| limit 100
```

### Filter by Process Group Name (Partial Match)

```dql
fetch logs, from:now() - 1h
| fieldsAdd process_group = dt.process_group.detected_name
| filter contains(process_group, "payment")
| fields timestamp, status, content, process_group
| sort timestamp desc
| limit 100
```

### Combined Entity + Severity + Content Filter

```dql
fetch logs, from:now() - 2h
| fieldsAdd process_group = dt.process_group.detected_name
| filter process_group == "payment-service"
| filter status == "ERROR"
| filter contains(content, "transaction")
| fields timestamp, content, status
| sort timestamp desc
| limit 50
```

## Time Range Patterns

### Last N Minutes

```dql
fetch logs, from:now() - 30m
| fields timestamp, status, content
| sort timestamp desc
| limit 100
```

### Absolute Time Range

```dql
fetch logs, from: "2024-01-15T10:00:00Z", to: "2024-01-15T11:00:00Z"
| fields timestamp, status, content
| sort timestamp desc
| limit 100
```

### Rolling Window Comparison

```dql
fetch logs, from:now() - 2h
| filter timestamp > now() - 1h
| filter status == "ERROR"
| summarize recent = count()
```

## Aggregations by Entity

### Log Count per Process Group

```dql
fetch logs, from:now() - 1h
| summarize log_count = count(), by: {process_group = dt.process_group.detected_name}
| sort log_count desc
```

### Error Count per Process Group

```dql
fetch logs, from:now() - 1h
| filter status == "ERROR"
| summarize error_count = count(), by: {process_group = dt.process_group.detected_name}
| sort error_count desc
```

### Log Count by Severity

```dql
fetch logs, from:now() - 1h
| summarize count(), by: {status}
| sort `count()` desc
```

## Performance Tips

- **Filter early**: Apply `filter` before `summarize` or `fieldsAdd` to reduce data volume
- **Use `contains()` before `parse`**: Pre-filter content before JSON parsing
- **Limit time ranges**: Avoid unbounded queries; always use a `from:` clause
- **Use `limit`**: Add `| limit N` to cap result size during exploration
