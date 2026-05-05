# Pattern Analysis

Techniques for detecting patterns, parsing structured logs, and analyzing trends in Dynatrace log data.

## JSON / Structured Log Parsing

Many applications emit JSON-formatted log lines. Use `parse` to extract typed fields instead of relying on raw `content` string matching.

### Basic JSON Parsing

```dql
fetch logs, from:now() - 1h
| filter status == "ERROR"
| parse content, "JSON:log"
| fieldsAdd level = log[level], message = log[msg], error = log[error]
| fields timestamp, level, message, error
| sort timestamp desc
| limit 50
```

**Notes:**
- `parse content, "JSON:log"` creates a record field `log` — access nested values with `log[key]`
- Always `filter` before `parse` to reduce parsing overhead
- Works with any JSON-structured field, not just `content`

### Aggregate by Parsed Message Field

```dql
fetch logs, from:now() - 4h
| filter status == "ERROR"
| parse content, "JSON:log"
| fieldsAdd message = log[msg]
| summarize error_count = count(), by: {message}
| sort error_count desc
| limit 20
```

### Extract HTTP Status from JSON Logs

```dql
fetch logs, from:now() - 1h
| filter contains(content, "http_status")
| parse content, "JSON:log"
| fieldsAdd http_status = log[http_status]
| filter toInt(http_status) >= 500
| fields timestamp, http_status, content
| sort timestamp desc
| limit 50
```

### Extract Latency Values from JSON Logs

```dql
fetch logs, from:now() - 1h
| filter contains(content, "duration_ms")
| parse content, "JSON:log"
| fieldsAdd duration = toDouble(log[duration_ms])
| summarize
    avg_duration = avg(duration),
    p95_duration = percentile(duration, 95),
    p99_duration = percentile(duration, 99),
    by: {process_group = dt.process_group.detected_name}
| sort avg_duration desc
```

## Exception Pattern Detection

### Detect Multiple Exception Types

```dql
fetch logs, from:now() - 2h
| filter status == "ERROR"
| fieldsAdd
    has_exception = if(matchesPhrase(content, "exception"), true, else: false),
    has_timeout = if(matchesPhrase(content, "timeout"), true, else: false),
    has_npe = if(contains(toLower(content), "nullpointerexception"), true, else: false),
    has_oom = if(matchesPhrase(content, "OutOfMemory"), true, else: false)
| summarize
    total = count(),
    exception_count = countIf(has_exception == true),
    timeout_count = countIf(has_timeout == true),
    npe_count = countIf(has_npe == true),
    oom_count = countIf(has_oom == true),
    by: {process_group = dt.process_group.detected_name}
```

### Exception Occurrence Over Time

```dql
fetch logs, from:now() - 4h
| filter status == "ERROR"
| fieldsAdd has_exception = if(matchesPhrase(content, "exception"), true, else: false)
| summarize
    total_errors = count(),
    exceptions = countIf(has_exception == true),
    by: {time_bucket = bin(timestamp, 15m)}
| fieldsAdd exception_ratio = (exceptions * 100.0) / total_errors
| sort time_bucket asc
```

## Log Volume and Trend Analysis

### Log Volume Over Time

```dql
fetch logs, from:now() - 24h
| summarize log_count = count(), by: {time_bucket = bin(timestamp, 1h)}
| sort time_bucket asc
```

### Log Volume by Severity Over Time

```dql
fetch logs, from:now() - 6h
| summarize count(), by: {time_bucket = bin(timestamp, 30m), status}
| filter in(status, {"ERROR", "WARN", "INFO"})
| sort time_bucket asc
```

### Sudden Volume Spikes (per service)

```dql
fetch logs, from:now() - 2h
| summarize
    log_count = count(),
    by: {time_bucket = bin(timestamp, 5m), process_group = dt.process_group.detected_name}
| sort log_count desc
| limit 30
```

## Keyword and Phrase Mining

### Top Repeated Phrases in Errors

```dql
fetch logs, from:now() - 1h
| filter status == "ERROR"
| summarize count(), by: {content}
| sort `count()` desc
| limit 20
```

### Detect Specific Infrastructure Keywords

```dql
fetch logs, from:now() - 1h
| filter status == "ERROR"
| fieldsAdd
    db_related = if(contains(toLower(content), "database") or contains(toLower(content), "sql"), true, else: false),
    network_related = if(contains(toLower(content), "connection") or contains(toLower(content), "timeout"), true, else: false),
    auth_related = if(contains(toLower(content), "unauthorized") or contains(toLower(content), "forbidden"), true, else: false)
| summarize
    db_errors = countIf(db_related == true),
    network_errors = countIf(network_related == true),
    auth_errors = countIf(auth_related == true),
    by: {process_group = dt.process_group.detected_name}
| sort db_errors desc
```

## Log Parsing with GROK-Style Patterns

### Extract Fields Using DQL `parse` Patterns

```dql
fetch logs, from:now() - 1h
| filter contains(content, "GET") or contains(content, "POST")
| parse content, "LD:method SPACE LD:path SPACE 'HTTP/' LD:http_version SPACE INT:status_code"
| fields timestamp, method, path, status_code
| filter toInt(status_code) >= 400
| sort timestamp desc
| limit 50
```

### Extract IP Addresses

```dql
fetch logs, from:now() - 1h
| filter contains(content, "client")
| parse content, "LD 'client=' LD:client_ip SPACE LD"
| fields timestamp, client_ip, content
| sort timestamp desc
| limit 50
```

## Baseline Comparison

### Errors This Hour vs Previous Hour

```dql
fetch logs, from:now() - 2h
| filter status == "ERROR"
| fieldsAdd window = if(timestamp >= now() - 1h, "current_hour", else: "previous_hour")
| summarize error_count = count(), by: {window, process_group = dt.process_group.detected_name}
| sort window asc, error_count desc
```
