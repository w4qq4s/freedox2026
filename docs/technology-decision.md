## 1. Backend Framework: Python FastAPI

**Requirement**
We need a backend that can serve RBAC-protected REST endpoints, handle relatively complex
derived data (risk scoring, duplicate detection), integrate cleanly with Supabase (Postgres),
and be fast enough to build within a hackathon timeframe like the current one.

**Options considered**
- Flask
- Django REST Framework
- Node.js (Express)
- FastAPI

**Evaluation**
- Flask is minimal but leaves us to hand-roll request validation and auth dependency injection.
- Django REST Framework gives a lot out of the box (admin panel, ORM) but its weight and
  opinionated structure would slow us down for a scoped hackathon MVP.
- Node/Express is a reasonable alternative, but our team has stronger Python familiarity.
- FastAPI gives us native async support, Pydantic-based request/response validation (which
  directly helps prevent malformed data at the API boundary, a security-relevant win, not
  just convenience), and built-in dependency injection that makes RBAC role/department scope
  checks a clean, reusable decorator-like pattern applied per route.

**Decision**
FastAPI. The combination of async support, first-class validation, and clean dependency
injection for auth/RBAC checks was decisive, it directly supports our security requirements
(input validation, per-route access scoping) rather than requiring us to build that separately.

**Evidence**
We prototyped the `/equipment/{id}/status` PATCH route with a role+department dependency guard
and confirmed a Lab Assistant token scoped to Department A receives a 403 when targeting
equipment in Department B, without any additional application-level branching logic.

---

## 2. Database: Supabase (Postgres)

**Requirement**
We need a relational database with strict foreign key constraints (Department → Laboratory →
Equipment hierarchy), CHECK constraints for data integrity (e.g., quantity ≥ 0, valid status
enum), row-level audit logging, and fast setup so the team can focus on the app layer rather
than infrastructure.

**Options considered**
- SQLite (local file-based)
- Firebase Firestore (NoSQL)
- Self-hosted PostgreSQL
- Supabase (managed Postgres)

**Evaluation**
- SQLite is simple but doesn't handle concurrent writes well, which matters once we simulate
  multiple roles (Lab Assistant, Dept Head, Superadmin) making updates during the live demo.
- Firestore's document model actively works against our need for strict relational integrity —
  the Department → Laboratory → Equipment chain and foreign-key-backed audit log both need
  proper relational constraints, not application-enforced document references.
- Self-hosted Postgres gives full control but adds setup/deployment overhead we don't have time
  for in a hackathon window, and doesn't add capability over a managed option for our scale.
- Supabase gives us managed Postgres (full constraint support, foreign keys, JSONB for audit
  `old_value`/`new_value` diffs) plus built-in auth and row-level security policies we can layer
  on top of our own RBAC as defense-in-depth, without managing our own database server.

**Decision**
Supabase. It gives us real Postgres, needed for our integrity constraints and audit log design
— without infrastructure overhead, and its row-level security features let us enforce access
scoping at the database layer as a second line of defense behind our FastAPI RBAC checks.


---

## 3. Caching: Redis

**Requirement**
Department-wise and category-wise dashboard aggregates (equipment counts, functional vs.
non-functional ratios, risk-score summaries) are read far more often than the underlying data
changes, and recomputing them on every dashboard load adds unnecessary query load.

**Options considered**
- No caching (recompute on every request)
- In-memory Python dict cache
- Redis

**Evaluation**
- No caching is simplest but means every dashboard view re-runs aggregate queries across
  equipment/laboratory joins, which is wasteful for data that changes infrequently relative to
  how often it's viewed.
- An in-memory Python dict cache works for a single-process demo but doesn't survive restarts,
  doesn't support TTL-based invalidation cleanly, and won't scale if we run multiple API workers.
- Redis gives us TTL-based cache invalidation (so stale dashboard numbers self-correct within a
  short window even if we miss an explicit invalidation call), and is a standard, defensible
  choice that's easy to justify rather than an ad hoc workaround.

**Decision**
Redis, used specifically for dashboard aggregate caching (equipment-by-lab counts, functional/
non-functional ratios), not for core transactional data — audit logs and equipment records
always read from Postgres directly to avoid any staleness in security-relevant data.

**Evidence**
We deliberately excluded audit_log and RBAC role lookups from the cache layer — those are always
read live from Postgres — so a cache-poisoning or staleness issue can't be used to mask an
unauthorized action or serve stale permission data.

---

## 4. Frontend: Plain HTML/CSS/JS (with a lightweight QR reader)

**Requirement**
We need a UI that can run the standard Create → View → Search/Filter → Update → Report flow, a
mobile-usable QR scan-to-status-page flow (for the "physical-to-digital binding" feature), and
be buildable quickly without a heavy build pipeline for a hackathon timeline.

**Options considered**
- React / Vue SPA
- Plain HTML/CSS/JS

**Evaluation**
- A React/Vue SPA gives more structure for larger apps, but introduces build tooling overhead
  we don't need for our scope, and our QR scan flow specifically benefits from being a very
  lightweight, fast-loading page (scanned from a phone camera, often on lab wifi) rather than a
  full SPA bundle.
- Plain HTML/CSS/JS keeps the QR kiosk page minimal and fast to load, which matters for the
  actual use case (a lab assistant scanning a physical asset), and is sufficient for our
  dashboard/CRUD views given our team size and timeline.

**Decision**
Plain HTML/CSS/JS, with a browser-based QR reader library for the scan flow, calling the
FastAPI backend directly via fetch.

**Evidence**
The QR kiosk status page loads and renders equipment status without any build step, and we
confirmed scan-to-status-page latency is dominated by the network call, not client-side
framework overhead.

---

## 5. Security-Specific Decisions

**Requirement**
Beyond the base V11 problem statement, we chose to treat this as a system holding
institution-owned physical assets, which warranted explicit security engineering rather than
security as an afterthought.

**Decisions made, with reasoning:**

- **RBAC enforced server-side, not just hidden in the UI.** Hiding an admin button in the
  frontend doesn't stop a direct API call. Every protected route checks role + department scope
  via a FastAPI dependency before executing the query.
- **Passwords hashed with bcrypt, never stored in plaintext.** Standard practice; the cost of
  implementing this correctly is negligible against the cost of a plaintext credential leak.
- **Parameterized queries only, no string-concatenated SQL.** We use Supabase's client library
  / SQLAlchemy parameter binding throughout; we specifically tested `' OR 1=1--`-style input in
  the equipment search/filter field and confirmed it's treated as literal text, not executed SQL.
- **Audit log is append-only at the database grant level** (INSERT/SELECT only, UPDATE/DELETE
  revoked), so even a compromised application-layer account can't quietly edit history.
- **QR payloads are HMAC-signed**, not raw equipment IDs, so a cloned or hand-crafted QR code
  pointing at a different asset ID is rejected server-side before the kiosk page renders any data.
- **Secrets (DB credentials, HMAC signing key) live in environment variables**, excluded from
  version control via `.gitignore`, and are never hardcoded in source.

**Evidence**
We documented and will demonstrate live: (1) a Dept-Head-scoped token receiving 403 on a
superadmin-only route via direct API call, (2) a SQL-injection payload in the search box being
safely handled, and (3) an audit log entry remaining intact and traceable after an equipment
status update, with old/new values correctly diffed.