# Backend Product Requirements Plan (PRP)
## Real-Time System Usage Dashboard - Backend

---

## 1) Backend Summary

Build a robust, scalable backend system that collects, stores, processes, and serves system usage metrics from one or more hosts. The backend comprises an agent (deployed on monitored hosts), an ingest pipeline, a time-series data store, an API layer, and an alerting engine. It provides RESTful APIs consumed by the frontend and supports real-time metric delivery, historical queries, alerting, and reporting.

**Elevator pitch:**
"Collect metrics from hundreds of hosts, store efficiently, query fast, alert intelligently—all while scaling horizontally and maintaining <500ms API latency."

---

## 2) Backend Goals & Non-Goals

### Goals
- **[REQ-BE-001]** High-throughput metric ingestion: ≥1M datapoints/min aggregate
- **[REQ-BE-002]** Low-latency API: p95 <500ms for typical queries (last 1h, ~100k points)
- **[REQ-BE-003]** Horizontal scalability: shard by host tag/region
- **[REQ-BE-004]** Reliable alerting: threshold and duration-based rules with noise reduction
- **[REQ-BE-005]** Data retention: 7d raw, 30–90d downsampled (configurable)
- **[REQ-BE-006]** Security: TLS, RBAC, hashed secrets, per-tenant isolation (if multi-tenant)
- **[REQ-BE-007]** Observability: emit metrics, logs, and health endpoints

### Non-Goals (v1)
- **[REQ-BE-NG-001]** Long-term retention (>90 days) or compliance archiving
- **[REQ-BE-NG-002]** Automated remediation or auto-scaling actions
- **[REQ-BE-NG-003]** Full SIEM/SOC features (deep log analytics, tracing)
- **[REQ-BE-NG-004]** Custom query language (use predefined metric names)

---

## 3) Users & Backend Use Cases

### Primary Consumers
- **Frontend SPA**: Queries host metadata, timeseries, and alerts
- **Agents**: Push metrics to ingest endpoints
- **External Systems**: Receive alert webhooks

### Key Backend Use Cases
- **[REQ-BE-UC-001]** Agent sends CPU/GPU/Memory metrics every 5s → Ingest validates and stores
- **[REQ-BE-UC-002]** Frontend requests last 1h of CPU data for a host → API queries time-series DB and returns JSON
- **[REQ-BE-UC-003]** Alert rule triggers (CPU >90% for 10m) → Send email and webhook
- **[REQ-BE-UC-004]** Admin schedules weekly report → Backend generates PDF/CSV and emails
- **[REQ-BE-UC-005]** Agent loses network → Buffers locally, retries when reconnected

---

## 4) Backend Scope

### Components

#### 4.1 Agent
- **[REQ-AGENT-001]** Lightweight collector running on each monitored host
- **[REQ-AGENT-002]** Collects metrics: CPU, Memory, Disk, Network, GPU (if available), Processes
- **[REQ-AGENT-003]** Collection interval: 5s (configurable via backend config push)
- **[REQ-AGENT-004]** Secure push to ingest endpoint (TLS 1.2+)
- **[REQ-AGENT-005]** Local buffering: up to 5 minutes of metrics on network failure
- **[REQ-AGENT-006]** Heartbeat: sends alive signal every 30s
- **[REQ-AGENT-007]** Resource limits: <1% CPU, <50MB memory

#### 4.2 Ingest Pipeline
- **[REQ-INGEST-001]** Stateless HTTP/gRPC gateway (auto-scale)
- **[REQ-INGEST-002]** Validates incoming metrics (schema, timestamp sanity)
- **[REQ-INGEST-003]** Enriches with host metadata (tags, zone)
- **[REQ-INGEST-004]** Batch writes to time-series DB (e.g., 1000 points/batch)
- **[REQ-INGEST-005]** Idempotent: deduplicates duplicate submissions
- **[REQ-INGEST-006]** Backpressure handling: safe drop policy for overflow (emit warnings)

#### 4.3 Time-Series Data Store
- **[REQ-STORE-001]** Hot storage: in-memory or SSD-backed (e.g., InfluxDB, TimescaleDB, Prometheus)
- **[REQ-STORE-002]** Retention: raw data 7 days, downsampled data 30–90 days
- **[REQ-STORE-003]** Downsampling: 1m avg for <24h, 5m avg for >24h, 1h avg for >7d
- **[REQ-STORE-004]** Sharding: by host tag/region for horizontal scale
- **[REQ-STORE-005]** Replication: 3x for HA (optional v1.1)

#### 4.4 API Layer
- **[REQ-API-001]** RESTful JSON APIs for frontend consumption
- **[REQ-API-002]** Authentication: Bearer token (JWT) or session cookie
- **[REQ-API-003]** Authorization: RBAC (Viewer/Admin roles)
- **[REQ-API-004]** Rate limiting: 60 req/min/user (Viewer), 30 req/min/user (Admin writes)
- **[REQ-API-005]** Caching: aggressive caching for read endpoints (1–5s TTL)
- **[REQ-API-006]** Pagination: limit/offset for large result sets
- **[REQ-API-007]** Real-time delivery: WebSocket or SSE for live updates

#### 4.5 Alerting Engine
- **[REQ-ALERT-001]** Rule evaluation: run every 30s against recent data
- **[REQ-ALERT-002]** Rule types: threshold (static), duration (e.g., CPU >90% for 10m)
- **[REQ-ALERT-003]** Scope: per-host or tag-based (e.g., role=db)
- **[REQ-ALERT-004]** States: firing, acknowledged, resolved
- **[REQ-ALERT-005]** Noise reduction: cooldown period (5m between same alert)
- **[REQ-ALERT-006]** Channels: email, generic webhook (Slack/Teams via webhook URL)
- **[REQ-ALERT-007]** Retry: 3 attempts with exponential backoff for webhook failures

#### 4.6 Reporting Service
- **[REQ-REPORT-001]** Scheduled reports: weekly summary (PDF/CSV) per tag or host group
- **[REQ-REPORT-002]** On-demand export: CSV for charts/tables
- **[REQ-REPORT-003]** Storage: temporary S3/blob store, signed URLs for download

---

## 5) Functional Requirements (Backend)

### 5.1 Host & Inventory Management
- **[REQ-HOST-001]** CRUD operations for hosts (manual add or API import)
- **[REQ-HOST-002]** Host attributes: id, hostname, ips[], tags{}, has_gpu, agent_version, last_seen_at
- **[REQ-HOST-003]** Host discovery: CSV import or lightweight network scan (optional v1.1)
- **[REQ-HOST-004]** Host status: derived from agent heartbeat and metric thresholds
  - **OK**: all metrics below Warn, heartbeat <2m ago
  - **Warn**: any metric in Warn range (70–90%)
  - **Crit**: any metric >90% for ≥1m or heartbeat lost >2m

### 5.2 Metrics Collection & Transport
- **[REQ-METRIC-001]** Agent collects metrics at 5s intervals (default, configurable)
- **[REQ-METRIC-002]** Supported metrics:
  - **CPU**: `cpu.utilization`, `cpu.load1`, `cpu.load5`, `cpu.load15`, `cpu.core.N` (per-core %)
  - **Memory**: `mem.used_pct`, `mem.used_mb`, `mem.available_mb`, `mem.cache_mb`, `mem.swap_pct`
  - **GPU**: `gpu.utilization`, `gpu.memory_used_mb`, `gpu.memory_total_mb`, `gpu.temperature_c`, `gpu.power_w`, `gpu.process.pid`
  - **Disk**: `disk.used_pct`, `disk.iops_read`, `disk.iops_write`, `disk.throughput_read_mbps`, `disk.throughput_write_mbps`
  - **Network**: `net.tx_mbps`, `net.rx_mbps`, `net.errors`, `net.drops`, `net.sockets_active`
  - **Processes**: `proc.top.cpu_pct`, `proc.top.mem_mb`, `proc.top.gpu_mem_mb` (top N processes)
- **[REQ-METRIC-003]** Metric format: `{ "host_id": "h-001", "metric": "cpu.utilization", "ts": "2025-11-13T08:50:00Z", "value": 85.2, "labels": {} }`
- **[REQ-METRIC-004]** Agent sends batches of up to 100 metrics per POST
- **[REQ-METRIC-005]** Transport: HTTPS POST to `/api/v1/ingest/metrics`

### 5.3 Data Retention & Downsampling
- **[REQ-RETENTION-001]** Raw data: 7 days (configurable to 3–14d)
- **[REQ-RETENTION-002]** Downsampled data:
  - **1-minute avg**: 7–30 days
  - **5-minute avg**: 30–90 days
- **[REQ-RETENTION-003]** Automated cleanup: daily cron job to purge expired data
- **[REQ-RETENTION-004]** Downsampling: background job runs hourly to aggregate raw data

### 5.4 Alerting Rules & Notifications
- **[REQ-ALERTRULE-001]** Rule schema:
  ```json
  {
    "id": "rule-001",
    "name": "CPU Hot",
    "expression": "cpu.utilization > 90",
    "duration": "10m",
    "scope": { "tags": { "role": "web" } },
    "channels": ["email:ops@example.com", "webhook:https://hooks.slack.com/..."],
    "enabled": true
  }
  ```
- **[REQ-ALERTRULE-002]** Evaluation: every 30s, query last 10m of data
- **[REQ-ALERTRULE-003]** Fire alert: if condition met for entire duration
- **[REQ-ALERTRULE-004]** Resolve alert: if condition no longer met for 2 evaluation cycles
- **[REQ-ALERTRULE-005]** Acknowledge: mark alert as acknowledged (manual user action)
- **[REQ-ALERTRULE-006]** Cooldown: 5m minimum between same alert re-firing

### 5.5 RBAC & Authentication
- **[REQ-RBAC-001]** Roles:
  - **Viewer**: read-only access (view hosts, metrics, alerts)
  - **Admin**: full access (manage hosts, rules, users, exports)
- **[REQ-RBAC-002]** Auth methods:
  - Email/password (bcrypt hashed, salted)
  - Optional: OIDC/OAuth (v1.1)
- **[REQ-RBAC-003]** Session management: JWT tokens (15m expiry, refresh token for 7d)
- **[REQ-RBAC-004]** API authorization: enforce role checks on write endpoints
- **[REQ-RBAC-005]** Audit log: record user actions (login, rule changes, host edits)

### 5.6 Reporting & Export
- **[REQ-EXPORT-001]** On-demand CSV export: query time-series for a metric, return CSV
- **[REQ-EXPORT-002]** Weekly scheduled report:
  - Aggregated stats (avg, p95, max) for CPU/GPU/Memory per host
  - PDF or CSV format
  - Email to configured recipients
- **[REQ-EXPORT-003]** Export storage: temporary S3 bucket, signed URLs (expire in 1h)

---

## 6) Non-Functional Requirements (Backend)

### 6.1 Performance
- **[REQ-BE-PERF-001]** Ingest throughput: ≥1M datapoints/min (aggregate across all ingest nodes)
- **[REQ-BE-PERF-002]** API latency:
  - p50: <200ms
  - p95: <500ms
  - p99: <1s
- **[REQ-BE-PERF-003]** Query performance: 1h window (~720 points/metric) returns in <300ms
- **[REQ-BE-PERF-004]** Alert evaluation: complete cycle in <10s (for all rules)

### 6.2 Scalability
- **[REQ-BE-SCALE-001]** Horizontal scaling: add ingest/query nodes to increase capacity
- **[REQ-BE-SCALE-002]** Sharding: partition data by host tag (e.g., zone=us-west, zone=eu-central)
- **[REQ-BE-SCALE-003]** Support ≥500 hosts in v1, ≥5000 hosts in v1.1
- **[REQ-BE-SCALE-004]** Metric cardinality: <10,000 unique metric names (prevent explosion)

### 6.3 Reliability
- **[REQ-BE-REL-001]** Uptime: ≥99.9% monthly (app UI/API)
- **[REQ-BE-REL-002]** Agent heartbeat loss: <0.1%/day
- **[REQ-BE-REL-003]** Idempotent ingest: duplicate metric submissions are deduplicated
- **[REQ-BE-REL-004]** Graceful degradation: if alert webhook fails, queue for retry (3x)

### 6.4 Security
- **[REQ-BE-SEC-001]** Transport: TLS 1.2+ for all client↔backend and agent↔backend
- **[REQ-BE-SEC-002]** Secrets: bcrypt hashing for passwords, rotateable API keys (admin only)
- **[REQ-BE-SEC-003]** Data isolation: per-tenant namespace (if multi-tenant enabled)
- **[REQ-BE-SEC-004]** Rate limiting: prevent abuse (60 req/min/user for reads, 30 req/min for writes)
- **[REQ-BE-SEC-005]** Input validation: reject malformed metrics, SQL injection protection
- **[REQ-BE-SEC-006]** HSTS: enforce HTTPS on all endpoints

### 6.5 Privacy
- **[REQ-BE-PRIV-001]** No PII by default (process names are operational data)
- **[REQ-BE-PRIV-002]** Optional: mask process names per policy (e.g., replace with hash)
- **[REQ-BE-PRIV-003]** Audit logs: retain 90 days, secure access

### 6.6 Observability
- **[REQ-BE-OBS-001]** Emit metrics:
  - Ingest rate, API latency, error rates, DB query time, alert evaluation time
- **[REQ-BE-OBS-002]** Structured logs: JSON format, log levels (INFO, WARN, ERROR)
- **[REQ-BE-OBS-003]** Health endpoints: `/health` (liveness), `/ready` (readiness)
- **[REQ-BE-OBS-004]** Tracing: optional distributed tracing (OpenTelemetry) in v1.1

---

## 7) Data Model (Backend)

### 7.1 Entities

#### Host
```json
{
  "id": "host-001",
  "hostname": "web-01.example.com",
  "ips": ["10.0.1.5", "172.16.0.10"],
  "tags": { "role": "web", "zone": "us-west", "gpu": "true" },
  "has_gpu": true,
  "agent_version": "v1.0.2",
  "last_seen_at": "2025-11-13T08:50:00Z",
  "status": "ok"
}
```

#### MetricPoint
```json
{
  "host_id": "host-001",
  "metric_name": "cpu.utilization",
  "timestamp": "2025-11-13T08:50:00Z",
  "value": 85.2,
  "labels": {}
}
```
**GPU example**:
```json
{
  "host_id": "host-001",
  "metric_name": "gpu.memory_used_mb",
  "timestamp": "2025-11-13T08:50:00Z",
  "value": 8192,
  "labels": { "gpu_index": "0" }
}
```

#### AlertRule
```json
{
  "id": "rule-001",
  "name": "CPU Hot",
  "expression": "cpu.utilization > 90",
  "duration": "10m",
  "scope": { "tags": { "role": "web" } },
  "channels": ["email:ops@example.com", "webhook:https://..."],
  "enabled": true
}
```

#### AlertEvent
```json
{
  "id": "event-001",
  "rule_id": "rule-001",
  "host_id": "host-001",
  "state": "firing",
  "started_at": "2025-11-13T08:45:00Z",
  "ended_at": null,
  "acknowledged_by": null,
  "severity": "critical"
}
```

#### User
```json
{
  "id": "user-001",
  "email": "alice@example.com",
  "password_hash": "$2a$10$...",
  "role": "Admin",
  "status": "active"
}
```

### 7.2 Metric Naming Convention
- **Pattern**: `<category>.<subcategory>.<unit>`
- **Examples**:
  - `cpu.utilization` (%)
  - `cpu.load1` (float)
  - `mem.used_pct` (%)
  - `gpu.memory_used_mb` (MB)
  - `disk.iops_read` (ops/s)
  - `net.tx_mbps` (Mbps)
  - `proc.top.cpu_pct` (% for top process)

---

## 8) API Specification (Backend)

### 8.1 Authentication

#### POST /api/v1/auth/login
**Request**:
```json
{
  "email": "alice@example.com",
  "password": "***"
}
```
**Response**:
```json
{
  "token": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "role": "Admin",
  "expires_at": "2025-11-13T09:05:00Z"
}
```

#### POST /api/v1/auth/logout
**Request**: Bearer token in header  
**Response**: `{ "status": "ok" }`

---

### 8.2 Hosts

#### GET /api/v1/hosts
**Query params**:
- `tags`: e.g., `role:web,zone:us-west` (AND logic)
- `status`: `ok`, `warn`, `crit`
- `limit`, `offset`: pagination

**Response**:
```json
{
  "hosts": [
    {
      "id": "host-001",
      "hostname": "web-01.example.com",
      "ips": ["10.0.1.5"],
      "tags": { "role": "web", "zone": "us-west" },
      "status": "warn",
      "last_seen_at": "2025-11-13T08:50:00Z",
      "metrics_summary": {
        "cpu_pct": 85.2,
        "mem_pct": 72.1,
        "gpu_pct": null,
        "disk_pct": 45.3,
        "net_tx_mbps": 120.5
      }
    }
  ],
  "total": 150,
  "limit": 50,
  "offset": 0
}
```

#### GET /api/v1/hosts/{id}
**Query params**:
- `range`: `15m`, `1h`, `6h`, `24h`, `7d`

**Response**:
```json
{
  "host": { "id": "host-001", "hostname": "web-01.example.com", ... },
  "metrics": {
    "cpu.utilization": [
      { "ts": "2025-11-13T07:50:00Z", "value": 45.2 },
      { "ts": "2025-11-13T07:51:00Z", "value": 47.8 }
    ],
    "mem.used_pct": [ ... ],
    "gpu.utilization": [ ... ]
  }
}
```

#### POST /api/v1/hosts
**Request** (Admin only):
```json
{
  "hostname": "db-02.example.com",
  "ips": ["10.0.2.10"],
  "tags": { "role": "db", "zone": "us-east" }
}
```
**Response**: `{ "id": "host-002", "status": "created" }`

#### DELETE /api/v1/hosts/{id}
**Response**: `{ "status": "deleted" }`

---

### 8.3 Metrics Query

#### GET /api/v1/metrics/query
**Query params**:
- `host_id`: `host-001`
- `metric`: `cpu.utilization`
- `range`: `1h`, `6h`, `24h`, `7d`
- `labels`: (optional) JSON object, e.g., `{"gpu_index":"0"}`

**Response**:
```json
{
  "metric": "cpu.utilization",
  "host_id": "host-001",
  "datapoints": [
    { "ts": "2025-11-13T08:00:00Z", "value": 45.2 },
    { "ts": "2025-11-13T08:01:00Z", "value": 47.8 }
  ],
  "resolution": "1m"
}
```

---

### 8.4 Top Processes

#### GET /api/v1/processes/top
**Query params**:
- `host_id`: `host-001`
- `metric`: `cpu`, `mem`, `gpu_mem`
- `limit`: `10` (default)

**Response**:
```json
{
  "processes": [
    {
      "pid": 1234,
      "name": "python3",
      "cpu_pct": 95.2,
      "mem_mb": 512,
      "gpu_mem_mb": 2048,
      "gpu_index": 0
    }
  ]
}
```

---

### 8.5 Alerts

#### GET /api/v1/alerts/rules
**Response**:
```json
{
  "rules": [
    {
      "id": "rule-001",
      "name": "CPU Hot",
      "expression": "cpu.utilization > 90",
      "duration": "10m",
      "scope": { "tags": { "role": "web" } },
      "channels": ["email:ops@example.com"],
      "enabled": true
    }
  ]
}
```

#### POST /api/v1/alerts/rules (Admin)
**Request**:
```json
{
  "name": "GPU Saturated",
  "expression": "gpu.utilization > 85",
  "duration": "5m",
  "scope": { "tags": { "gpu": "true" } },
  "channels": ["webhook:https://hooks.slack.com/..."]
}
```
**Response**: `{ "id": "rule-002", "status": "created" }`

#### DELETE /api/v1/alerts/rules/{id} (Admin)
**Response**: `{ "status": "deleted" }`

#### GET /api/v1/alerts/events
**Query params**:
- `state`: `firing`, `acknowledged`, `resolved`
- `host_id`: (optional)
- `since`: ISO8601 timestamp

**Response**:
```json
{
  "alerts": [
    {
      "id": "event-001",
      "rule_name": "CPU Hot",
      "host_id": "host-001",
      "state": "firing",
      "started_at": "2025-11-13T08:45:00Z",
      "severity": "critical"
    }
  ]
}
```

#### POST /api/v1/alerts/events/{id}/acknowledge
**Request**: `{ "user": "alice@example.com" }`  
**Response**: `{ "status": "acknowledged" }`

---

### 8.6 Reports & Export

#### POST /api/v1/reports/export
**Request**:
```json
{
  "format": "csv",
  "host_id": "host-001",
  "metric": "cpu.utilization",
  "range": "24h"
}
```
**Response**:
```json
{
  "download_url": "https://storage.example.com/exports/cpu-001.csv?sig=...",
  "expires_at": "2025-11-13T09:50:00Z"
}
```

---

### 8.7 Ingest (Agent → Backend)

#### POST /api/v1/ingest/metrics
**Request** (Agent authenticated via API key):
```json
{
  "host_id": "host-001",
  "metrics": [
    { "metric": "cpu.utilization", "ts": "2025-11-13T08:50:00Z", "value": 85.2 },
    { "metric": "mem.used_pct", "ts": "2025-11-13T08:50:00Z", "value": 72.1 }
  ]
}
```
**Response**: `{ "accepted": 2, "rejected": 0 }`

---

### 8.8 Error Responses (Consistent Format)
```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Invalid or expired token",
    "details": {}
  }
}
```

**Error codes**:
- `UNAUTHORIZED` (401)
- `FORBIDDEN` (403)
- `NOT_FOUND` (404)
- `RATE_LIMIT_EXCEEDED` (429)
- `INTERNAL_ERROR` (500)

---

### 8.9 Rate Limiting
- **Header**: `X-RateLimit-Remaining: 45`
- **Response on exceed**: `429 Too Many Requests`

---

## 9) Alerting Rules (Starter Set)

| Rule ID | Name | Expression | Duration | Scope | Channels |
|---------|------|------------|----------|-------|----------|
| rule-001 | CPU Hot | `cpu.utilization > 90` | 10m | `role!=gpu-only` | email, webhook |
| rule-002 | GPU Saturated | `gpu.utilization > 85` OR `gpu.memory_used_pct > 90` | 5m | `gpu=true` | email, webhook |
| rule-003 | Memory Pressure | `mem.used_pct > 90` | 10m | all | email |
| rule-004 | Disk Fill Warn | `disk.used_pct > 85` | static | all | email |
| rule-005 | Disk Fill Critical | `disk.used_pct > 95` | static | all | email, webhook |
| rule-006 | Agent Silent | no heartbeat for 2m | 2m | all | email, webhook |

---

## 10) Thresholds & Defaults (Backend)

### 10.1 Collection & Retention
- **Collection interval**: 5s (fleet override permitted)
- **Retention**: 7d raw, 60d downsampled (configurable)
- **Top processes**: Top 10 by CPU/Memory; GPU processes if GPUs present

### 10.2 Status Derivation
- **OK**: all tracked metrics below Warn, heartbeat <2m ago
- **Warn**: any metric in 70–90% range
- **Crit**: any metric >90% for ≥1m OR heartbeat lost >2m

---

## 11) Architecture (Backend Detail)

### 11.1 High-Level Flow
```
Agent → Ingest → Store → API → Frontend
                       ↓
                   Alert Engine
                       ↓
                  Email/Webhook
```

### 11.2 Component Details

#### Agent
- **Language**: Go or Rust (low overhead)
- **Deployment**: systemd service on Linux, launchd on macOS
- **Config**: YAML file with backend endpoint, API key, collection interval
- **Buffering**: SQLite local DB (5 minutes capacity)

#### Ingest
- **Framework**: Go (net/http), Python (FastAPI), or Node.js (Express)
- **Endpoint**: `/api/v1/ingest/metrics`
- **Validation**: schema check, timestamp within [now-1h, now+5m]
- **Batching**: aggregate 1000 points before DB write
- **Scaling**: horizontal (stateless, behind load balancer)

#### Store
- **Options**:
  - **InfluxDB**: native time-series, good query performance
  - **TimescaleDB**: PostgreSQL extension, SQL queries
  - **Prometheus**: built-in alerting (if using as store)
- **Schema**:
  - Measurement: `metrics`
  - Tags: `host_id`, `metric_name`, `labels`
  - Fields: `value`
  - Timestamp: nanosecond precision

#### API
- **Framework**: Go (Gin), Python (FastAPI), Node.js (Express)
- **Middleware**: auth, rate limit, CORS, logging
- **Caching**: Redis for frequently queried metrics (1–5s TTL)
- **WebSocket/SSE**: separate service or integrated (e.g., Socket.IO, Go channels)

#### Alert Engine
- **Execution**: cron job every 30s
- **Logic**:
  1. Fetch all enabled rules
  2. For each rule, query last 10m of data for scoped hosts
  3. Evaluate expression
  4. If condition met for entire duration → fire alert
  5. Send notifications (email + webhook)
- **State management**: store alert events in DB (PostgreSQL, MySQL)

#### Reporting Service
- **Scheduled**: cron job (weekly, Sunday midnight)
- **On-demand**: API endpoint triggers report generation
- **Generation**: Python (Pandas + matplotlib) or Go (chart libraries)
- **Storage**: S3/GCS, signed URLs

---

## 12) Security & Compliance (Backend)

### 12.1 Transport Security
- **[REQ-BE-SEC-TLS-001]** TLS 1.2+ for all HTTPS endpoints
- **[REQ-BE-SEC-TLS-002]** HSTS header: `Strict-Transport-Security: max-age=31536000`

### 12.2 Authentication & Authorization
- **[REQ-BE-SEC-AUTH-001]** Passwords: bcrypt hashed (cost=10)
- **[REQ-BE-SEC-AUTH-002]** API keys: random 32-byte tokens, rotateable (admin only)
- **[REQ-BE-SEC-AUTH-003]** JWT tokens: 15m expiry, refresh token for 7d
- **[REQ-BE-SEC-AUTH-004]** RBAC enforcement: check role on every write endpoint

### 12.3 Data Isolation
- **[REQ-BE-SEC-ISO-001]** Per-tenant namespace (if multi-tenant enabled)
- **[REQ-BE-SEC-ISO-002]** Query scoping: filter by tenant_id at DB layer

### 12.4 Audit Logging
- **[REQ-BE-SEC-AUDIT-001]** Log admin actions: rule changes, user edits, host deletions
- **[REQ-BE-SEC-AUDIT-002]** Retention: 90 days
- **[REQ-BE-SEC-AUDIT-003]** Format: JSON logs with user_id, action, timestamp, IP

### 12.5 Input Validation
- **[REQ-BE-SEC-VAL-001]** Reject malformed JSON
- **[REQ-BE-SEC-VAL-002]** SQL injection protection: use parameterized queries
- **[REQ-BE-SEC-VAL-003]** Metric name whitelist: only allow known metric names

---

## 13) Performance Targets (Backend)

| Metric | Target | Measurement |
|--------|--------|-------------|
| Ingest throughput | ≥1M datapoints/min | Load test with multiple agents |
| API latency (p50) | <200ms | Prometheus histogram |
| API latency (p95) | <500ms | Prometheus histogram |
| API latency (p99) | <1s | Prometheus histogram |
| Query time (1h window) | <300ms | Trace with APM tool |
| Alert evaluation cycle | <10s | Timer in alert engine |
| Agent CPU usage | <1% | Monitor on host |
| Agent memory usage | <50MB | Monitor on host |

---

## 14) Telemetry & Logging (Backend)

### 14.1 Metrics (Prometheus format)
- **Ingest**:
  - `ingest_requests_total` (counter)
  - `ingest_datapoints_total` (counter)
  - `ingest_errors_total` (counter)
- **API**:
  - `api_requests_total` (counter, labels: endpoint, status)
  - `api_latency_seconds` (histogram)
- **Alerts**:
  - `alert_evaluations_total` (counter)
  - `alert_notifications_sent_total` (counter, labels: channel)
- **DB**:
  - `db_query_duration_seconds` (histogram)

### 14.2 Logs (Structured JSON)
```json
{
  "timestamp": "2025-11-13T08:50:00Z",
  "level": "INFO",
  "service": "ingest",
  "message": "Accepted 100 datapoints from host-001",
  "host_id": "host-001",
  "datapoints": 100
}
```

### 14.3 Health Endpoints
- **GET /health**: `{ "status": "ok" }`
- **GET /ready**: `{ "status": "ready", "db": "ok", "cache": "ok" }`

---

## 15) Dependencies & Integrations (Backend)

### 15.1 External Services
- **Email provider**: SMTP (SendGrid, AWS SES, Mailgun)
- **Webhook delivery**: HTTP POST to Slack/Teams/generic URLs
- **Storage**: S3/GCS for report exports (optional: local filesystem)
- **OIDC/OAuth**: (optional v1.1) for enterprise SSO

### 15.2 Internal Dependencies
- **Time-series DB**: InfluxDB, TimescaleDB, or Prometheus
- **Relational DB**: PostgreSQL or MySQL (for hosts, users, rules, alerts)
- **Cache**: Redis (optional, for API response caching)
- **Message Queue**: (optional v1.1) Kafka/RabbitMQ for async processing

---

## 16) Backend Acceptance Criteria (v1)

- **[AC-BE-001]** Agent sends metrics every 5s; ingest accepts and stores 100% within 2s
- **[AC-BE-002]** API returns host list (50 hosts) in <500ms
- **[AC-BE-003]** Query 1h of CPU data (720 points) returns in <300ms
- **[AC-BE-004]** Alert rule evaluates every 30s; fires email within 1m of condition breach
- **[AC-BE-005]** Viewer role cannot delete hosts (403 Forbidden)
- **[AC-BE-006]** Admin can create/edit/delete alert rules via API
- **[AC-BE-007]** Weekly report generates and emails PDF within 5m of scheduled time
- **[AC-BE-008]** Agent buffers locally when network down; retries on reconnect (no data loss)
- **[AC-BE-009]** Rate limit: 61st request/min returns 429 error
- **[AC-BE-010]** Health endpoint returns 200 OK; ready endpoint checks DB connectivity

---

## 17) Backend Success Metrics

- **[METRIC-BE-001]** Ingest throughput: ≥1M datapoints/min sustained
- **[METRIC-BE-002]** API availability: ≥99.9% uptime/month
- **[METRIC-BE-003]** Alert false positive rate: <5%
- **[METRIC-BE-004]** Agent heartbeat loss: <0.1%/day
- **[METRIC-BE-005]** Query cache hit rate: ≥70%

---

## 18) Backend Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Agent overhead on host | Skews metrics, user distrust | Keep CPU <1%, memory <50MB; use native APIs; sampling |
| Metric cardinality explosion | Storage/query slow | Label budget, downsample/rollups, reject unknown metrics |
| Ingest overload | Data loss, latency | Horizontal scaling, backpressure handling, safe drop policy |
| Alert fatigue | Ignored alerts | Sensible defaults, cooldowns, per-tag scoping |
| DB query slow | API timeout | Indexing, caching, downsampling, query optimization |
| Security exposure | Data leak | TLS, RBAC, hardened endpoints, rate limits, input validation |

---

## 19) Backend Milestones

### M1 – Prototype (4 weeks)
- **[BE-M1-001]** Agent POC: collect CPU/Mem, send to ingest endpoint
- **[BE-M1-002]** Ingest service: validate and store in time-series DB
- **[BE-M1-003]** API: GET /api/v1/hosts, GET /api/v1/hosts/{id}
- **[BE-M1-004]** Basic auth: email/password login

### M2 – Multi-host & GPU (4 weeks)
- **[BE-M2-001]** GPU metrics collection (NVIDIA SMI)
- **[BE-M2-002]** Top processes API
- **[BE-M2-003]** Host tagging and filtering
- **[BE-M2-004]** WebSocket/SSE for real-time updates

### M3 – Alerts & Reports (3 weeks)
- **[BE-M3-001]** Alert engine: rule evaluation, email/webhook delivery
- **[BE-M3-002]** Alert CRUD APIs
- **[BE-M3-003]** CSV export API
- **[BE-M3-004]** Weekly scheduled reports

### M4 – Hardening (3 weeks)
- **[BE-M4-001]** RBAC enforcement (Viewer/Admin roles)
- **[BE-M4-002]** Rate limiting middleware
- **[BE-M4-003]** Retention and downsampling automation
- **[BE-M4-004]** Security audit: TLS, input validation, audit logs

---

## 20) Open Questions (Backend)

- **[Q-BE-001]** Preferred time-series DB: InfluxDB, TimescaleDB, or Prometheus?
- **[Q-BE-002]** Downsampling strategy: backend job or DB-native (e.g., TimescaleDB continuous aggregates)?
- **[Q-BE-003]** Multi-tenancy in v1 or v1.1?
- **[Q-BE-004]** Air-gapped deployments: local email relay, no internet-based webhooks?
- **[Q-BE-005]** Agent auto-update mechanism or manual deployment?

---

## 21) Glossary (Backend Context)

- **Agent**: Lightweight metric collector running on monitored hosts
- **Ingest**: Backend service accepting and storing metrics
- **Time-series DB**: Database optimized for time-stamped data (e.g., InfluxDB)
- **Downsampling**: Reducing data resolution over time (e.g., 1m → 5m avg)
- **RBAC**: Role-based access control (Viewer, Admin)
- **Heartbeat**: Periodic alive signal from agent to backend
- **Idempotent**: Operation that produces same result if repeated (no duplicates)

---

## 22) Appendix — Backend Component Sizing (Example)

### v1 Target: 100 hosts, 5s collection interval
- **Datapoints/min**: 100 hosts × 12 metrics × 12 samples/min = 14,400/min
- **Ingest nodes**: 1 (can handle 1M/min)
- **Time-series DB**: 1 node (InfluxDB on 8GB RAM, 100GB SSD)
- **API nodes**: 2 (for HA, behind load balancer)
- **Alert engine**: 1 cron job (30s cycle)

### v1.1 Target: 500 hosts
- **Datapoints/min**: 500 × 12 × 12 = 72,000/min
- **Ingest nodes**: 2 (load balanced)
- **Time-series DB**: 2 nodes (replicated)
- **API nodes**: 3 (horizontal scale)

---

## 23) Testing Strategy (Backend)

### 23.1 Unit Tests
- **[TEST-BE-001]** Metric validation logic
- **[TEST-BE-002]** Alert rule expression parsing
- **[TEST-BE-003]** RBAC permission checks

### 23.2 Integration Tests
- **[TEST-BE-004]** Agent → Ingest → Store roundtrip
- **[TEST-BE-005]** API query accuracy (compare expected vs actual results)
- **[TEST-BE-006]** Alert rule evaluation and notification delivery

### 23.3 Load Tests
- **[TEST-BE-007]** Ingest throughput: send 1M datapoints/min, verify acceptance
- **[TEST-BE-008]** API latency: concurrent requests (100 users), measure p95

### 23.4 Security Tests
- **[TEST-BE-009]** Auth bypass attempts (401/403 responses)
- **[TEST-BE-010]** SQL injection (parameterized queries protect)
- **[TEST-BE-011]** Rate limit enforcement (429 on exceed)

---

**End of Backend PRP**
