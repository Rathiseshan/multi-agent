# PRP Split Mapping Document
## Real-Time System Usage Dashboard - Requirements Classification

This document shows how requirements from the original PRP.md were classified and distributed to Frontend_PRP.md and Backend_PRP.md.

---

## Summary Statistics

- **Original PRP**: 378 lines across 24 sections
- **Frontend PRP**: ~600 lines with 170+ requirements
- **Backend PRP**: ~850 lines with 250+ requirements
- **Total Requirement IDs**: 420+ (with granular breakdown)
- **Shared/Boundary Items**: API contracts, metric naming, alert states, error taxonomy

---

## Section-by-Section Mapping

| Original Section | Frontend | Backend | Shared/Boundary | Notes |
|-----------------|----------|---------|-----------------|-------|
| 1) Summary | ✓ | ✓ | - | Rewritten for each context |
| 2) Goals & Non-Goals | ✓ | ✓ | - | Separated by concern (UX vs. performance/scale) |
| 3) Users & Use Cases | ✓ | ✓ | - | Frontend: UI flows; Backend: API consumers |
| 4) Scope | ✓ | ✓ | Metric names | Frontend: views; Backend: collection/storage |
| 5) Out-of-Scope | ✓ | ✓ | - | Retained in both for context |
| 6) Success Metrics | ✓ | ✓ | - | Split: FE (load time), BE (throughput, uptime) |
| 7) UX & Visual Design | ✓ | - | - | 100% frontend (charts, themes, a11y) |
| 8) Functional Requirements | ✓ | ✓ | API contracts | FE: UI logic; BE: data processing, RBAC |
| 9) Non-Functional Requirements | ✓ | ✓ | - | FE: TTI, bundle size; BE: ingest rate, latency |
| 10) Data Model | - | ✓ | Metric naming | Backend owns schema |
| 11) API | - | ✓ | ✓ | Backend defines; Frontend consumes (documented in both) |
| 12) Alerting Rules | ✓ | ✓ | Alert states | FE: UI for rules; BE: evaluation engine |
| 13) Thresholds & Defaults | ✓ | ✓ | Threshold values | FE: visual thresholds; BE: status derivation |
| 14) Architecture | ✓ | ✓ | - | High-level in both; detailed in BE |
| 15) Security & Compliance | ✓ | ✓ | Auth flow | FE: CSP, XSS; BE: TLS, RBAC, encryption |
| 16) Performance Targets | ✓ | ✓ | - | FE: chart render; BE: API latency, ingest |
| 17) Telemetry & Logging | ✓ | ✓ | - | FE: client-side; BE: server metrics |
| 18) Dependencies | ✓ | ✓ | - | FE: libraries; BE: DB, email, webhooks |
| 19) Risks & Mitigations | ✓ | ✓ | - | Context-specific risks |
| 20) Milestones | ✓ | ✓ | - | Aligned timelines, separated deliverables |
| 21) Acceptance Criteria | ✓ | ✓ | - | FE: UI/UX; BE: API, agent, alerts |
| 22) Open Questions | ✓ | ✓ | - | Context-specific questions |
| 23) Glossary | ✓ | ✓ | - | Terms relevant to each domain |
| 24) Appendix | ✓ | ✓ | - | FE: widget inventory; BE: sizing examples |

---

## Requirement ID Classification

### Frontend Requirements (REQ-FE-xxx)

#### Goals & UX
- **[REQ-FE-001]** Fast time-to-insight with clear visuals
- **[REQ-FE-002]** Intuitive navigation (multi-host → single host)
- **[REQ-FE-003]** Real-time updates (<2s refresh)
- **[REQ-FE-004]** Dark/Light theme support
- **[REQ-FE-005]** Responsive layout (desktop/tablet)
- **[REQ-FE-006]** Keyboard navigation & WCAG AA
- **[REQ-FE-007]** Render performance (p95 <2.5s)

#### Views (REQ-VIEW-xxx)
- **[REQ-VIEW-001–019]** Fleet Overview, Host Detail, Alerts Center, Reports

#### User Experience (REQ-UX-xxx)
- **[REQ-UX-001–016]** Status chips, charts, tables, empty states, global nav

#### Accessibility (REQ-A11Y-xxx)
- **[REQ-A11Y-001–004]** Keyboard nav, color contrast, screen readers, ARIA

#### Frontend Auth (REQ-FE-AUTH-xxx)
- **[REQ-FE-AUTH-001–005]** Login form, session management, role display

#### Data Fetching (REQ-FE-DATA-xxx)
- **[REQ-FE-DATA-001–004]** Polling/WebSocket, auto-refresh, error handling

#### Charts (REQ-FE-CHART-xxx)
- **[REQ-FE-CHART-001–004]** Chart library, lazy load, zoom/pan, export

#### Filtering (REQ-FE-FILTER-xxx)
- **[REQ-FE-FILTER-001–003]** Tag filters, quick filters, URL persistence

#### Export (REQ-FE-EXPORT-xxx)
- **[REQ-FE-EXPORT-001–002]** CSV/PDF download

#### Performance (REQ-FE-PERF-xxx)
- **[REQ-FE-PERF-001–005]** TTI, FCP, chart render time, bundle size, code splitting

#### Scalability (REQ-FE-SCALE-xxx)
- **[REQ-FE-SCALE-001–002]** Handle 200+ hosts, virtual scrolling

#### Reliability (REQ-FE-REL-xxx)
- **[REQ-FE-REL-001–003]** Offline fallback, error boundaries, graceful degradation

#### Security (REQ-FE-SEC-xxx)
- **[REQ-FE-SEC-001–004]** No localStorage secrets, CSP, XSS protection, HTTPS

#### Observability (REQ-FE-OBS-xxx)
- **[REQ-FE-OBS-001–003]** Client telemetry, usage analytics, error tracking

**Total Frontend Requirements**: ~170

---

### Backend Requirements (REQ-BE-xxx)

#### Goals
- **[REQ-BE-001]** Ingest ≥1M datapoints/min
- **[REQ-BE-002]** API latency p95 <500ms
- **[REQ-BE-003]** Horizontal scalability
- **[REQ-BE-004]** Reliable alerting
- **[REQ-BE-005]** Data retention (7d raw, 30–90d downsampled)
- **[REQ-BE-006]** Security (TLS, RBAC, isolation)
- **[REQ-BE-007]** Observability (metrics, logs, health)

#### Agent (REQ-AGENT-xxx)
- **[REQ-AGENT-001–007]** Lightweight collector, 5s interval, secure push, buffering, heartbeat, resource limits

#### Ingest (REQ-INGEST-xxx)
- **[REQ-INGEST-001–006]** Stateless gateway, validation, batching, idempotent, backpressure

#### Store (REQ-STORE-xxx)
- **[REQ-STORE-001–005]** Hot storage, retention, downsampling, sharding, replication

#### API (REQ-API-xxx)
- **[REQ-API-001–007]** RESTful JSON, auth, RBAC, rate limiting, caching, pagination, real-time

#### Alerts (REQ-ALERT-xxx)
- **[REQ-ALERT-001–007]** Rule evaluation, types, states, noise reduction, channels, retry

#### Reports (REQ-REPORT-xxx)
- **[REQ-REPORT-001–003]** Scheduled, on-demand, signed URLs

#### Hosts (REQ-HOST-xxx)
- **[REQ-HOST-001–004]** CRUD, attributes, discovery, status derivation

#### Metrics (REQ-METRIC-xxx)
- **[REQ-METRIC-001–005]** Collection, naming, format, batching, transport

#### Retention (REQ-RETENTION-xxx)
- **[REQ-RETENTION-001–004]** Raw/downsampled retention, cleanup, aggregation

#### Alert Rules (REQ-ALERTRULE-xxx)
- **[REQ-ALERTRULE-001–006]** Schema, evaluation, fire/resolve, acknowledge, cooldown

#### RBAC (REQ-RBAC-xxx)
- **[REQ-RBAC-001–005]** Roles, auth methods, session management, API authorization, audit log

#### Export (REQ-EXPORT-xxx)
- **[REQ-EXPORT-001–003]** CSV, weekly reports, storage

#### Performance (REQ-BE-PERF-xxx)
- **[REQ-BE-PERF-001–004]** Ingest throughput, API latency, query perf, alert eval

#### Scalability (REQ-BE-SCALE-xxx)
- **[REQ-BE-SCALE-001–004]** Horizontal scaling, sharding, host support, cardinality limits

#### Reliability (REQ-BE-REL-xxx)
- **[REQ-BE-REL-001–004]** Uptime, heartbeat, idempotent ingest, graceful degradation

#### Security (REQ-BE-SEC-xxx)
- **[REQ-BE-SEC-001–006]** TLS, hashed secrets, data isolation, rate limiting, input validation, HSTS
- **[REQ-BE-SEC-TLS-001–002]** TLS version, HSTS header
- **[REQ-BE-SEC-AUTH-001–004]** Password hashing, API keys, JWT, RBAC enforcement
- **[REQ-BE-SEC-ISO-001–002]** Tenant namespace, query scoping
- **[REQ-BE-SEC-AUDIT-001–003]** Admin logs, retention, format
- **[REQ-BE-SEC-VAL-001–003]** JSON validation, SQL injection, metric whitelist

#### Privacy (REQ-BE-PRIV-xxx)
- **[REQ-BE-PRIV-001–003]** No PII, process name masking, audit log security

#### Observability (REQ-BE-OBS-xxx)
- **[REQ-BE-OBS-001–004]** Emit metrics, structured logs, health endpoints, tracing

**Total Backend Requirements**: ~250

---

## Shared/Boundary Definitions

### API Contracts
Documented in **both** PRPs with clear request/response examples:
- **POST /api/v1/auth/login**: Auth flow
- **GET /api/v1/hosts**: Fleet overview data
- **GET /api/v1/hosts/{id}**: Host detail with time-series
- **GET /api/v1/metrics/query**: Metric query API
- **GET /api/v1/processes/top**: Process list
- **GET /api/v1/alerts/events**: Alert list
- **POST /api/v1/alerts/events/{id}/acknowledge**: Alert acknowledgment
- **POST /api/v1/reports/export**: CSV/PDF export

### Metric Naming Convention
- **Pattern**: `<category>.<subcategory>.<unit>`
- **Examples**: `cpu.utilization`, `mem.used_pct`, `gpu.memory_used_mb`, `disk.iops_read`, `net.tx_mbps`
- **Documented in**: Backend (owned), Frontend (consumed)

### Alert States
- **firing**: Condition met, notification sent
- **acknowledged**: User acknowledged, no further notifications
- **resolved**: Condition no longer met
- **Documented in**: Both PRPs

### Error Taxonomy
Consistent error response structure:
```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Invalid token",
    "details": {}
  }
}
```
**Codes**: UNAUTHORIZED (401), FORBIDDEN (403), NOT_FOUND (404), RATE_LIMIT_EXCEEDED (429), INTERNAL_ERROR (500)

### Visual Thresholds
- **OK (Green)**: <70%
- **Warn (Amber)**: 70–90%
- **Crit (Red)**: >90%
- **Documented in**: Frontend (visual), Backend (status derivation)

### Rate Limiting
- **Viewer**: 60 req/min
- **Admin**: 30 req/min (writes)
- **Header**: `X-RateLimit-Remaining: N`
- **Documented in**: Backend (enforced), Frontend (handled)

---

## Deduplication & Consistency

### Items Deduplicated
- **Original Section 11 (API)**: Now fully detailed in Backend PRP; Frontend has contract documentation
- **Metric definitions**: Owned by Backend; Frontend references them
- **Thresholds**: Backend derives status; Frontend applies visual styling
- **RBAC**: Backend enforces; Frontend hides/shows UI elements

### Cross-References Added
- **Frontend → Backend**: API contracts, metric names, alert states
- **Backend → Frontend**: Expected UI flows, real-time update requirements

### No Orphaned Requirements
Every section from the original PRP is accounted for in either Frontend, Backend, or both.

---

## Key Differences from Original PRP

### Frontend PRP Additions
- Granular UI component requirements (buttons, cards, charts, tables)
- Accessibility (WCAG AA, keyboard nav, screen readers)
- Client-side state management and caching
- Theme system details
- Error boundary and offline fallback
- Testing strategy (unit, integration, E2E, a11y)

### Backend PRP Additions
- Agent implementation details (buffering, heartbeat, resource limits)
- Ingest pipeline (validation, batching, idempotency)
- Time-series DB schema and query optimization
- Alert engine evaluation logic
- Security deep-dive (TLS, hashing, audit logs)
- Downsampling and retention automation
- Component sizing examples

---

## Validation Checklist

- [x] No contradictory constraints (e.g., FE cache TTL vs BE freshness) ✓
- [x] All requirements classified (Frontend, Backend, or Shared) ✓
- [x] API contracts defined with examples in both PRPs ✓
- [x] Metric naming consistent across both PRPs ✓
- [x] Alert states and workflows aligned ✓
- [x] Thresholds synchronized (visual + status derivation) ✓
- [x] Glossary terms consistent ✓
- [x] Milestones aligned (4+4+3+3 weeks) ✓
- [x] Acceptance criteria measurable and testable ✓

---

## Quick Reference Table

| Requirement ID | Description | Frontend | Backend | Shared |
|---------------|-------------|----------|---------|--------|
| [REQ-FE-001] | Fast time-to-insight | ✓ | - | - |
| [REQ-BE-001] | Ingest ≥1M datapoints/min | - | ✓ | - |
| [REQ-VIEW-001] | Fleet Overview grid/list | ✓ | - | - |
| [REQ-AGENT-001] | Lightweight collector | - | ✓ | - |
| [REQ-API-001] | RESTful JSON APIs | - | ✓ | ✓ |
| [REQ-A11Y-001] | Keyboard navigation | ✓ | - | - |
| [REQ-ALERT-001] | Rule evaluation (30s cycle) | - | ✓ | - |
| [REQ-FE-SEC-001] | No localStorage secrets | ✓ | - | - |
| [REQ-BE-SEC-001] | TLS 1.2+ | - | ✓ | - |
| [REQ-FE-PERF-001] | TTI <3s | ✓ | - | - |
| [REQ-BE-PERF-002] | API latency p95 <500ms | - | ✓ | - |

---

## Next Steps

1. **Review**: Stakeholders review Frontend_PRP.md and Backend_PRP.md
2. **Refine**: Address open questions in each PRP
3. **Backlog**: Convert requirements to user stories/tasks
4. **Design**: Create UI mockups (FE) and API specs (BE)
5. **Implementation**: Follow milestone plan (M1–M4)

---

**End of Mapping Document**
