# Frontend Product Requirements Plan (PRP)
## Real-Time System Usage Dashboard - Frontend

---

## 1) Frontend Summary

Build a beautiful, responsive single-page application (SPA) that visualizes real-time system usage across one or more hosts. The UI provides live monitoring, short-term troubleshooting, and lightweight historical trends without requiring users to write queries. The frontend consumes backend APIs to display CPU, GPU, memory, disk, network, processes, and system health metrics.

**Elevator pitch:**
"Open the app, immediately see how your machines are behaving now and how they've behaved recently—beautiful charts, zero terminal commands."

---

## 2) Frontend Goals & Non-Goals

### Goals
- **[REQ-FE-001]** Fast time-to-insight with clear visuals, smart defaults, and minimal clicks
- **[REQ-FE-002]** Intuitive navigation: multi-host overview with drill-down to single machine
- **[REQ-FE-003]** Real-time updates: dashboard refreshes within <2s of new data
- **[REQ-FE-004]** Dark/Light theme support with accessible color palettes
- **[REQ-FE-005]** Responsive layout for desktop and tablet viewports (mobile read-only)
- **[REQ-FE-006]** Keyboard navigation and WCAG AA compliance
- **[REQ-FE-007]** Render performance: p95 host detail load < 2.5s

### Non-Goals (v1)
- **[REQ-FE-NG-001]** Mobile-optimized layouts or native mobile apps
- **[REQ-FE-NG-002]** Custom query builder or formula editor UI
- **[REQ-FE-NG-003]** Embedded dashboards or iframe support
- **[REQ-FE-NG-004]** Real-time collaboration features (shared cursors, annotations)

---

## 3) Users & Frontend Use Cases

### Primary Personas
- **Ops Engineer (Alice)**: Quick health checks, spot bottlenecks at a glance
- **ML/Compute Owner (Ben)**: Confirm GPU saturation/fragmentation with detailed charts
- **Team Lead (Cara)**: High-level status view, export weekly reports

### Key Frontend Use Cases
- **[REQ-UX-001]** "See all hosts at a glance; which ones are red/orange?"
- **[REQ-UX-002]** "Drill into one host; what's spiking right now?"
- **[REQ-UX-003]** "Which processes are hogging GPU memory?"
- **[REQ-UX-004]** "Did yesterday's deploy increase CPU by 30%?" (compare time windows)
- **[REQ-UX-005]** "Toggle between light and dark theme for comfort"

---

## 4) Frontend Scope

### Views & Components

#### 4.1 Fleet Overview
- **[REQ-VIEW-001]** Grid/list toggle showing all hosts with status badges (OK/Warn/Crit)
- **[REQ-VIEW-002]** Quick sorting by hostname, CPU%, GPU%, Memory%, status
- **[REQ-VIEW-003]** Tag-based filtering (role=db, zone=ap-sg, gpu=true, etc.)
- **[REQ-VIEW-004]** Search bar for hostname/IP lookup
- **[REQ-VIEW-005]** Color-coded summary cards per host: CPU/GPU/Memory/Disk/Net

#### 4.2 Host Detail View
- **[REQ-VIEW-006]** Time range picker: 15m, 1h, 6h, 24h, 7d
- **[REQ-VIEW-007]** Metric charts: CPU, Memory, GPU (if applicable), Disk, Network
- **[REQ-VIEW-008]** Single-metric line charts with hover tooltips showing exact values
- **[REQ-VIEW-009]** Mini-legends for chart clarity
- **[REQ-VIEW-010]** Top N processes table (sortable by CPU, Memory, GPU VRAM)
- **[REQ-VIEW-011]** GPU breakdown: per-GPU small multiples (util %, mem %, temp)
- **[REQ-VIEW-012]** Quick stats cards: CPU %, Mem %, GPU %, Disk used %, Net I/O

#### 4.3 Alerts Center
- **[REQ-VIEW-013]** List of active alerts with status chips (Firing, Acknowledged, Resolved)
- **[REQ-VIEW-014]** Alert rule management UI (create/edit/delete for Admin role)
- **[REQ-VIEW-015]** Acknowledge and suppress actions
- **[REQ-VIEW-016]** Alert history table with filters (date range, severity, host)

#### 4.4 Reports View
- **[REQ-VIEW-017]** On-demand export button (CSV) for charts and tables
- **[REQ-VIEW-018]** Weekly scheduled report configuration (Admin only)
- **[REQ-VIEW-019]** Export confirmation and download link

---

## 5) UX & Visual Design Requirements

### 5.1 Global Navigation
- **[REQ-UX-010]** Top bar with:
  - Environment switcher (if multi-tenant enabled)
  - Host search field (autocomplete)
  - Theme toggle (Dark/Light)
  - User menu (profile, logout)
- **[REQ-UX-011]** Sidebar navigation: Fleet Overview, Alerts, Reports (collapsible)

### 5.2 Visual Elements
- **[REQ-UX-012]** Status chips:
  - **OK**: Green (#10B981)
  - **Warn**: Amber (#F59E0B)
  - **Crit**: Red (#EF4444)
- **[REQ-UX-013]** Charts: clean line charts with gridlines, axis labels, and responsive scaling
- **[REQ-UX-014]** Tables: sticky headers, sortable columns, row hover state
- **[REQ-UX-015]** Empty states: helpful guidance (e.g., "No GPU detected on this host")
- **[REQ-UX-016]** Loading skeletons for async data (no spinners where possible)

### 5.3 Accessibility
- **[REQ-A11Y-001]** Keyboard navigation: tab order, focus indicators, Esc to close modals
- **[REQ-A11Y-002]** WCAG AA color contrast (min 4.5:1 for text)
- **[REQ-A11Y-003]** Screen reader announcements for dynamic content updates
- **[REQ-A11Y-004]** ARIA labels for charts, status badges, and icon buttons

---

## 6) Functional Requirements (Frontend)

### 6.1 Authentication & Authorization (Frontend)
- **[REQ-FE-AUTH-001]** Login form (email/password) with validation
- **[REQ-FE-AUTH-002]** Session management: store auth token (Bearer) in memory/secure cookie
- **[REQ-FE-AUTH-003]** Auto-redirect to login on 401 responses
- **[REQ-FE-AUTH-004]** Display user role (Viewer/Admin) in UI
- **[REQ-FE-AUTH-005]** Hide/disable admin-only actions for Viewer role

### 6.2 Data Fetching & Real-Time Updates
- **[REQ-FE-DATA-001]** Poll or subscribe (WebSocket/SSE) for near-real-time metric updates
- **[REQ-FE-DATA-002]** Auto-refresh fleet overview every 5-10s (configurable)
- **[REQ-FE-DATA-003]** Graceful handling of API errors (retry with exponential backoff)
- **[REQ-FE-DATA-004]** Display last-update timestamp on dashboards

### 6.3 Chart Rendering
- **[REQ-FE-CHART-001]** Use charting library (e.g., Chart.js, Recharts, D3) for line charts
- **[REQ-FE-CHART-002]** Lazy-load chart data on view mount
- **[REQ-FE-CHART-003]** Zoom/pan interactions for detailed inspection (optional v1.1)
- **[REQ-FE-CHART-004]** Export chart as PNG/SVG (optional v1.1)

### 6.4 Filtering & Sorting
- **[REQ-FE-FILTER-001]** Tag filters: apply multiple tags (AND logic)
- **[REQ-FE-FILTER-002]** Quick filters: "GPU > 80%", "Disk > 85%"
- **[REQ-FE-FILTER-003]** Persist filter state in URL query params for bookmarking

### 6.5 Exporting
- **[REQ-FE-EXPORT-001]** CSV export: trigger backend export API, download file
- **[REQ-FE-EXPORT-002]** PDF export: trigger backend report generation, download link (Admins only)

---

## 7) Non-Functional Requirements (Frontend)

### 7.1 Performance
- **[REQ-FE-PERF-001]** Time-to-interactive (TTI): <3s on 3G connection
- **[REQ-FE-PERF-002]** First Contentful Paint (FCP): <1.5s
- **[REQ-FE-PERF-003]** Chart render time: <500ms for 1h window (~720 data points)
- **[REQ-FE-PERF-004]** Bundle size: <500KB gzipped (main bundle)
- **[REQ-FE-PERF-005]** Lazy-load routes and components for code splitting

### 7.2 Scalability
- **[REQ-FE-SCALE-001]** Handle ≥200 hosts in fleet overview without pagination lag
- **[REQ-FE-SCALE-002]** Virtual scrolling for long process lists (>100 items)

### 7.3 Reliability
- **[REQ-FE-REL-001]** Offline fallback: display cached data with staleness indicator
- **[REQ-FE-REL-002]** Error boundaries: catch React errors, show fallback UI
- **[REQ-FE-REL-003]** Graceful degradation: if WebSocket fails, fallback to polling

### 7.4 Security
- **[REQ-FE-SEC-001]** No sensitive data in localStorage (use secure cookies or memory)
- **[REQ-FE-SEC-002]** Content Security Policy (CSP) headers enforced
- **[REQ-FE-SEC-003]** XSS protection: sanitize user inputs (search, tags)
- **[REQ-FE-SEC-004]** HTTPS only (HSTS enforced via backend)

### 7.5 Observability
- **[REQ-FE-OBS-001]** Client-side telemetry: page load time, API latency, error rates
- **[REQ-FE-OBS-002]** Usage analytics: most-viewed charts, common filters, time ranges
- **[REQ-FE-OBS-003]** Error tracking: send client errors to monitoring service (e.g., Sentry)

---

## 8) API Contracts (Frontend → Backend)

### 8.1 Authentication
**POST /api/v1/auth/login**
- **Request**: `{ "email": "alice@example.com", "password": "***" }`
- **Response**: `{ "token": "Bearer <JWT>", "role": "Admin" }`
- **Frontend action**: Store token, redirect to overview

### 8.2 Fleet Overview
**GET /api/v1/hosts**
- **Query params**: `?tags=role:db,zone:us-west&status=warn`
- **Response**: Array of hosts with last sample:
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
    ]
  }
  ```
- **Frontend action**: Render grid/list with status badges and summary cards

### 8.3 Host Detail
**GET /api/v1/hosts/{id}**
- **Query params**: `?range=1h` (options: 15m, 1h, 6h, 24h, 7d)
- **Response**: Host metadata + timeseries data
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
- **Frontend action**: Render charts with time-series data

### 8.4 Top Processes
**GET /api/v1/processes/top**
- **Query params**: `?host_id=host-001&metric=cpu&limit=10`
- **Response**:
  ```json
  {
    "processes": [
      { "pid": 1234, "name": "python3", "cpu_pct": 95.2, "mem_mb": 512, "gpu_mem_mb": 2048 }
    ]
  }
  ```
- **Frontend action**: Render sortable table

### 8.5 Alerts
**GET /api/v1/alerts/events**
- **Query params**: `?state=firing&host_id=host-001`
- **Response**:
  ```json
  {
    "alerts": [
      {
        "id": "alert-001",
        "rule_name": "CPU Hot",
        "host_id": "host-001",
        "state": "firing",
        "started_at": "2025-11-13T08:45:00Z",
        "severity": "critical"
      }
    ]
  }
  ```
- **Frontend action**: Display alert list with acknowledge buttons

**POST /api/v1/alerts/events/{id}/acknowledge**
- **Request**: `{ "user": "alice@example.com" }`
- **Response**: `{ "status": "acknowledged" }`

### 8.6 Alert Rules (Admin)
**GET /api/v1/alerts/rules**
- **Response**: Array of alert rules
**POST /api/v1/alerts/rules**
- **Request**: `{ "name": "CPU Hot", "expression": "cpu.utilization > 90", "duration": "10m", ... }`

### 8.7 Export
**POST /api/v1/reports/export**
- **Request**: `{ "format": "csv", "host_id": "host-001", "metric": "cpu.utilization", "range": "24h" }`
- **Response**: `{ "download_url": "https://..." }`
- **Frontend action**: Download file via link

### 8.8 Error Responses
All APIs return consistent error structure:
```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Invalid token",
    "details": {}
  }
}
```
**Frontend handling**: Display error toast, retry or redirect as appropriate

### 8.9 Rate Limiting
- **Viewer**: 60 req/min
- **Admin**: 30 req/min (writes)
- **Response header**: `X-RateLimit-Remaining: 45`
- **Frontend action**: Display warning if approaching limit

---

## 9) Frontend Technology Stack (Recommended)

- **Framework**: React 18+ (with hooks) or Vue 3 (for SPA)
- **State Management**: Zustand, Recoil, or Pinia (lightweight)
- **Charting**: Recharts or Chart.js
- **Styling**: Tailwind CSS with dark mode support
- **Real-time**: WebSocket (via Socket.IO) or Server-Sent Events (SSE)
- **HTTP Client**: Axios or Fetch API with interceptors
- **Build Tool**: Vite or Webpack 5
- **Testing**: Vitest + React Testing Library

---

## 10) Frontend Data Flow

1. **User loads app** → Login form → POST /api/v1/auth/login → Store token
2. **Navigate to Fleet Overview** → GET /api/v1/hosts → Render host grid
3. **Select host** → GET /api/v1/hosts/{id}?range=1h → Render charts
4. **Real-time updates** → WebSocket/SSE subscription → Update charts in-place
5. **Filter by tag** → Update query params → Re-fetch GET /api/v1/hosts?tags=...
6. **Export CSV** → POST /api/v1/reports/export → Download file

---

## 11) Frontend Thresholds & Defaults

### 11.1 Visual Thresholds
- **OK (Green)**: CPU/GPU/Memory/Disk < 70%
- **Warn (Amber)**: 70–90%
- **Crit (Red)**: > 90%

### 11.2 Default Settings
- **Default time range**: 1h
- **Auto-refresh interval**: 10s (configurable to 5s, 30s, off)
- **Top processes count**: 10
- **Chart resolution**: 1-minute granularity for <24h, 5-minute for >24h

---

## 12) Frontend Acceptance Criteria (v1)

- **[AC-FE-001]** From login to first fleet overview render in <2.5s with ≥50 hosts
- **[AC-FE-002]** Host detail page shows CPU, Memory, GPU, Disk, Network charts with selectable time ranges (1m–24h–7d)
- **[AC-FE-003]** Top processes table updates within <5s of manual refresh
- **[AC-FE-004]** Can toggle between dark and light themes; preference persists across sessions
- **[AC-FE-005]** Viewer role cannot see "Edit Rule" or "Delete Host" buttons
- **[AC-FE-006]** Can acknowledge an alert; UI updates to "Acknowledged" state immediately
- **[AC-FE-007]** Keyboard navigation: Tab through all interactive elements, Enter to activate
- **[AC-FE-008]** Export CSV: download link appears within 3s of clicking "Export"
- **[AC-FE-009]** Error state: if backend returns 500, display user-friendly error message (not stack trace)
- **[AC-FE-010]** Responsive: fleet overview readable on tablet (≥768px width)

---

## 13) Frontend Success Metrics

- **[METRIC-FE-001]** Median time-to-first-insight: <10s from login to key chart rendered
- **[METRIC-FE-002]** p95 page load time: <2.5s
- **[METRIC-FE-003]** Client-side error rate: <1% of page views
- **[METRIC-FE-004]** Theme adoption: ≥40% users enable dark mode
- **[METRIC-FE-005]** Accessibility audit score: ≥90 (Lighthouse)

---

## 14) Frontend Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Chart rendering slow with large datasets | Poor UX, frustration | Downsample data on backend; virtual scrolling; lazy load charts |
| WebSocket disconnect | Stale data | Auto-reconnect with exponential backoff; fallback to polling |
| Browser compatibility (old versions) | Broken UI | Define supported browsers (Chrome/FF/Safari last 2 versions); polyfills |
| Large bundle size | Slow initial load | Code splitting, lazy routes, tree shaking, CDN for static assets |
| API rate limit exceeded | Errors in UI | Client-side throttling; show warning before limit; cache responses |

---

## 15) Frontend Milestones

### M1 – Prototype (4 weeks)
- **[FE-M1-001]** Login page + session management
- **[FE-M1-002]** Fleet overview with mock data
- **[FE-M1-003]** Single host detail with CPU/Mem charts
- **[FE-M1-004]** Theme toggle (dark/light)

### M2 – Multi-host & GPU (4 weeks)
- **[FE-M2-001]** Fleet grid with real backend integration
- **[FE-M2-002]** GPU charts and per-GPU breakdowns
- **[FE-M2-003]** Top processes table (sortable)
- **[FE-M2-004]** Tag filtering and search

### M3 – Alerts & Reports (3 weeks)
- **[FE-M3-001]** Alerts center view (list, acknowledge, history)
- **[FE-M3-002]** Alert rule editor (Admin only)
- **[FE-M3-003]** CSV export UI

### M4 – Hardening (3 weeks)
- **[FE-M4-001]** RBAC UI enforcement (hide admin controls for Viewer)
- **[FE-M4-002]** Accessibility audit and fixes
- **[FE-M4-003]** Error boundaries and offline fallback
- **[FE-M4-004]** Performance optimization (bundle size, lazy loading)

---

## 16) Open Questions (Frontend)

- **[Q-FE-001]** Should we support IE11? (Assume no for v1)
- **[Q-FE-002]** Do we need a mobile-responsive layout, or is tablet (≥768px) sufficient?
- **[Q-FE-003]** Preferred charting library: Recharts (React), Chart.js (framework-agnostic), or D3 (custom)?
- **[Q-FE-004]** Should we embed a "Getting Started" tutorial in the UI (tooltips/tour)?
- **[Q-FE-005]** Do we support custom branding (logo, colors) in v1?

---

## 17) Glossary (Frontend Context)

- **SPA**: Single-page application (React/Vue app consuming REST APIs)
- **TTI**: Time-to-interactive (performance metric)
- **FCP**: First Contentful Paint (performance metric)
- **WCAG AA**: Web Content Accessibility Guidelines (level AA compliance)
- **SSE**: Server-Sent Events (for real-time updates)
- **Virtual scrolling**: Technique to render only visible items in a long list

---

## 18) Appendix — Frontend Widget Inventory (v1)

| Widget | Description | Priority |
|--------|-------------|----------|
| **Fleet Health Heatmap** | Hosts × key metric severity (color-coded grid) | P0 |
| **Top Hotspots** | Hosts with highest CPU/GPU/Memory (top 5 cards) | P0 |
| **GPU Matrix** | Per-GPU small multiples: util %, mem %, temp | P1 |
| **Host Quick Stats** | Cards: CPU %, Mem %, GPU %, Disk used %, Net I/O | P0 |
| **Trend Charts** | Selectable metric (CPU, Mem, GPU, Disk, Net) | P0 |
| **Top N Processes** | Table: CPU, Mem, GPU VRAM columns (sortable) | P0 |
| **Recent Alerts** | List with acknowledge/suppress buttons | P1 |
| **Tag Filters** | Multi-select dropdown (zone, role, team) | P0 |

---

## 19) Testing Strategy (Frontend)

### 19.1 Unit Tests
- **[TEST-FE-001]** Component rendering (smoke tests)
- **[TEST-FE-002]** Chart data transformation logic
- **[TEST-FE-003]** Utility functions (date formatting, threshold calculations)

### 19.2 Integration Tests
- **[TEST-FE-004]** API client (mocked responses)
- **[TEST-FE-005]** Auth flow (login → store token → redirect)
- **[TEST-FE-006]** Filter/sort interactions

### 19.3 E2E Tests
- **[TEST-FE-007]** Login → Fleet Overview → Host Detail → Export CSV
- **[TEST-FE-008]** Alert acknowledgment flow
- **[TEST-FE-009]** RBAC: Viewer cannot edit rules

### 19.4 Accessibility Tests
- **[TEST-FE-010]** Lighthouse audit (score ≥90)
- **[TEST-FE-011]** Keyboard navigation (tab order, focus management)
- **[TEST-FE-012]** Screen reader compatibility (NVDA/JAWS)

---

## 20) Dependencies (Frontend)

- **Backend APIs**: All frontend features depend on backend API availability
- **WebSocket/SSE**: Real-time updates depend on backend push mechanism
- **Auth service**: Login depends on backend auth endpoint
- **Export service**: CSV/PDF export depends on backend report generation

---

**End of Frontend PRP**
