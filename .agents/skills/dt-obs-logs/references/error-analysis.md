# Error Analysis

Patterns for calculating error rates, identifying top errors, and trending error activity in Dynatrace log data.

## Error Rate Calculation

### Overall Error Rate

```dql
fetch logs, from:now() - 1h
| summarize
    total = count(),
    errors = countIf(status == "ERROR"),
    fatals = countIf(status == "FATAL")
| fieldsAdd
    error_rate = (errors * 100.0) / total,
    fatal_rate = (fatals * 100.0) / total
```

### Error Rate per Process Group

```dql
fetch logs, from:now() - 1h
| summarize
    total = count(),
    errors = countIf(status == "ERROR"),
    by: {process_group = dt.process_group.detected_name}
| fieldsAdd error_rate = (errors * 100.0) / total
| sort error_rate desc
```

### Error Rate Over Time (5-minute buckets)

```dql
fetch logs, from:now() - 2h
| summarize
    total_logs = count(),
    error_logs = countIf(status == "ERROR"),
    by: {time_bucket = bin(timestamp, 5m)}
| fieldsAdd error_rate = (error_logs * 100.0) / total_logs
| sort time_bucket asc
```

### Error Rate Trend per Service

```dql
fetch logs, from:now() - 4h
| summarize
    total = count(),
    errors = countIf(status == "ERROR"),
    by: {time_bucket = bin(timestamp, 15m), process_group = dt.process_group.detected_name}
| fieldsAdd error_rate = (errors * 100.0) / total
| filter errors > 0
| sort time_bucket asc, error_rate desc
```

## Top Error Messages

### Most Frequent Error Messages

```dql
fetch logs, from:now() - 24h
| filter status == "ERROR"
| summarize error_count = count(), by: {content}
| sort error_count desc
| limit 20
```

### Top Errors per Service

```dql
fetch logs, from:now() - 24h
| filter status == "ERROR"
| summarize
    error_count = count(),
    by: {process_group = dt.process_group.detected_name, content}
| sort error_count desc
| limit 30
```

### New Errors (Only in Recent Window)

Compare error content from two windows to find newly appearing errors:

```dql
fetch logs, from:now() - 2h
| filter status == "ERROR"
| filter timestamp >= now() - 30m
| summarize recent_count = count(), by: {content}
| sort recent_count desc
| limit 20
```

## Error Severity Breakdown

### FATAL vs ERROR Count

```dql
fetch logs, from:now() - 24h
| filter in(status, {"ERROR", "FATAL"})
| summarize count(), by: {status, process_group = dt.process_group.detected_name}
| sort status asc, `count()` desc
```

### Severity Distribution

```dql
fetch logs, from:now() - 6h
| summarize count(), by: {status}
| sort `count()` desc
```

## Exception Analysis

### Exception Rate by Type

```dql
fetch logs, from:now() - 1h
| filter status == "ERROR"
| filter matchesPhrase(content, "exception")
| summarize exception_count = count(), by: {content}
| sort exception_count desc
| limit 20
```

### NullPointerException Frequency

```dql
fetch logs, from:now() - 2h
| filter contains(toLower(content), "nullpointerexception")
| summarize count(), by: {process_group = dt.process_group.detected_name}
| sort `count()` desc
```

### HTTP 5xx Error Log Analysis

```dql
fetch logs, from:now() - 1h
| filter status == "ERROR"
| filter contains(content, "500") or contains(content, "503") or contains(content, "502")
| summarize count(), by: {process_group = dt.process_group.detected_name, content}
| sort `count()` desc
| limit 20
```

## Error Spike Detection

### Errors per Minute (Last Hour)

```dql
fetch logs, from:now() - 1h
| filter status == "ERROR"
| summarize error_count = count(), by: {time_bucket = bin(timestamp, 1m)}
| sort time_bucket asc
```

### Identify Spike Windows

```dql
fetch logs, from:now() - 2h
| summarize
    error_count = countIf(status == "ERROR"),
    by: {time_bucket = bin(timestamp, 5m)}
| filter error_count > 10
| sort error_count desc
```

## Comparing Error Counts Across Time Periods

### Today vs Yesterday Error Count

```dql
fetch logs, from:now() - 48h
| filter status == "ERROR"
| fieldsAdd period = if(timestamp >= now() - 24h, "today", else: "yesterday")
| summarize error_count = count(), by: {period, process_group = dt.process_group.detected_name}
| sort period asc, error_count desc
```
