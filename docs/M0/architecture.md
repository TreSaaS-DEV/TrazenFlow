# System architecture and technical decisions

**Status:** Proposed architecture for review. No code or infrastructure has been created.

## 1. Architectural direction

Start with a **modular application plus independently scalable worker processes**, not a fleet of microservices. Keep clear module boundaries so high-load components can be split later without taking on early operational complexity.

Logical components:

1. Web client
2. Electron Windows client
3. API/application service
4. Authentication and authorization boundary
5. Workflow definition/version service
6. Execution orchestrator
7. Durable PostgreSQL job queue
8. Worker pool and integration adapters
9. File upload and processing service
10. Object storage
11. AI provider gateway and budget policy
12. Audit, telemetry, and notification services

The browser and Electron renderer should share UI packages where practical, but desktop-only capabilities must stay behind a narrow, validated preload API.

## 2. Candidate technology choices

| Area | Candidate | Rationale / caveat |
|---|---|---|
| Web | Next.js, React, TypeScript | Mature React ecosystem; evaluate deployment constraints before choosing server/client boundaries |
| UI | Tailwind CSS, accessible component primitives | Consistent responsive design system |
| Workflow canvas | React Flow | Node/edge editor; graph validation and execution semantics remain server-side |
| Desktop | Electron | Real Windows desktop capabilities; larger runtime footprint than native clients |
| API | Fastify with TypeScript | Schema-oriented API service; can share Zod contracts |
| Database | PostgreSQL | Transactions, relational integrity, JSON support, row-level security |
| Auth | Separate Supabase Auth project or another reviewed identity provider | Never reuse TrazenMarket auth/project/keys; final choice subject to isolation and cost review |
| Queue | PostgreSQL-backed queue, evaluate pg-boss | Avoid Redis as a mandatory initial dependency; validate compatibility and operational behaviour |
| Workers | Dedicated Node.js processes | Isolate slow/untrusted work from web request handling |
| Files | S3-compatible object storage | Large binary data should not be stored in relational rows |
| Validation | Zod | Shared boundary validation |
| Tests | Vitest and Playwright; Electron-capable Playwright tests | Unit, integration, browser and desktop workflows |
| Packaging | Electron Builder or equivalent | Produce Windows installer; validate build on Windows CI/host |

These are candidate choices, not installed dependencies or final commitments. Confirm supported versions and provider terms in M1 before locking them.

## 3. Deployment topology

### 3.1 Web/API

- Serve the web client and API through HTTPS.
- Keep API authorization server-side and verify workspace membership on every protected operation.
- Do not trust workspace IDs, role claims, file IDs, or ownership assertions supplied by the client without database-backed authorization.
- Use request IDs and audit correlation IDs across API and worker activity.

### 3.2 Database and durable queue

PostgreSQL is both the system of record and the initial queue substrate. Workflow execution state and queued work are related but distinct:

- Workflow tables describe immutable published definitions.
- Execution tables capture each run and its immutable input/configuration snapshot.
- Job tables or the selected Postgres queue library manage scheduling, attempts, leases, and delivery.
- Worker events and execution events record lifecycle transitions.

The queue must not be implemented as an in-memory array or a web-process background task. A process restart must not erase scheduled or pending work.

**Queue decision to validate in M1:** evaluate pg-boss against a narrowly scoped PostgreSQL job-table implementation. Prefer the maintained library if its lease, retry, scheduling, concurrency, and Supabase/PostgreSQL connection requirements match the product. Do not combine BullMQ and PostgreSQL as competing sources of truth. Redis is not an initial requirement.

Use a worker connection mode appropriate for long-lived worker sessions; verify connection-pooling behaviour and transaction semantics rather than assuming a serverless/transaction pool works for queue workers.

### 3.3 Execution lifecycle

Proposed state machine:

- `queued` → `claimed` → `running`
- `running` → `waiting_approval` → `queued` or `cancelled`
- `running` → `succeeded`, `failed`, `retry_wait`, or `cancelled`
- `retry_wait` → `queued`
- Expired worker lease → recoverable state and requeue subject to attempt policy

Every transition must be validated transactionally. Workers claim work with a lease and heartbeat. If the worker dies, the lease expires; a recovery process reclaims eligible work. A stale worker must not be allowed to overwrite newer state after its lease is lost.

### 3.4 Delivery and side-effect semantics

Assume **at-least-once delivery**. Deduplicate jobs by stable idempotency key. Before repeating an external side effect, the adapter must check whether the provider supports idempotency or provide a reconciliation strategy. When neither is possible, retries after ambiguous network outcomes must pause for policy or human review rather than falsely promise exactly-once behaviour.

### 3.5 Workflow definitions and execution snapshots

- Draft definitions can be edited.
- Publishing creates an immutable version.
- Each run records its workflow version, graph snapshot, resolved non-secret configuration snapshot, input references, and policy version.
- Edits to a draft do not mutate an existing published version or in-flight run.
- Sensitive credentials are referenced by credential ID/version, not copied into execution JSON.

## 4. AI orchestration

A provider-neutral gateway should normalize request/response, streaming, usage, errors, rate limits, and timeouts without flattening provider-specific capabilities.

Each AI step must declare:
- Allowed provider/model set
- Maximum steps/tool calls
- Time budget
- Input/output token or provider-usage budget where supported
- Estimated monetary budget and hard stop
- Allowed tools and scopes
- Approval requirements
- Data classification policy

No model call should occur merely because an agent requests it: the orchestrator enforces policy before every call and tool invocation. Provider outages should trigger bounded backoff, circuit breaking, or a visible failure state.

## 5. File processing pipeline

1. Create an authenticated upload session bound to a workspace, user, intended object key, maximum size, and expiry.
2. Upload directly/resumably to private object storage using short-lived signed access.
3. Finalize upload and verify object existence, size, checksum, and expected metadata.
4. Quarantine the object until validation/scanning policy completes.
5. Enqueue extraction/chunking/index jobs with durable state.
6. Process in bounded-memory streams or controlled chunks; enforce CPU, memory, time, decompression, and page limits.
7. Store derived outputs with tenant-aware metadata and access checks.
8. Expose status, retry, cancellation, deletion, and retention controls.

Do not load a 1 GB file entirely into application memory. Do not trust extension or client-supplied MIME type. Archives and document parsers must be isolated and resource-limited.

## 6. Desktop architecture

Electron has a renderer/main-process security boundary:

- Renderer runs with Node integration disabled and context isolation enabled.
- Use sandboxing where compatible.
- Expose only explicit, narrow preload methods.
- Validate IPC arguments and caller/frame identity.
- Keep secrets and filesystem operations out of renderer JavaScript.
- Ask the user to select specific folders; persist permissions transparently.
- Monitor only selected paths and provide visible pause/stop/remove controls.
- Use safe path handling and avoid following symlinks outside approved roots unless specifically designed and tested.
- Treat local files as untrusted input and pass them through the same upload validation pipeline.
- Updates and installers require integrity verification; code signing is recommended for distribution and may add cost.

Prefer Windows-native CI or a real Windows build host for the release installer. Cross-compilation may be useful but is not a substitute for validating install, update, permissions, tray, notifications, and uninstall on Windows.

## 7. Security boundaries

- Identity provider authenticates users; application authorization determines access.
- Database RLS is defence in depth, not a replacement for server-side permission checks.
- Every query and object key must be tenant-scoped.
- Integration credentials are encrypted using envelope encryption and a key-management strategy external to the database.
- Do not place secrets in client bundles, source control, logs, prompts, or execution snapshots.
- Outbound HTTP integrations require SSRF controls, including private/link-local/loopback address blocking, DNS rebinding considerations, redirect revalidation, and restricted ports/protocols.
- File parsing must be sandboxed and resource-bounded.
- Audit security-sensitive actions and credential use without recording credential values.

See the security threat model for more detail.

## 8. Environments and operations

Separate development, staging, and production by project, credentials, storage, database and callback URLs. Never promote production data into tests by default. Use synthetic fixtures. Backups, restore tests, migrations, health checks, alerting, rate limits and incident response must be defined before production release.

## 9. Decisions deliberately deferred

- Final auth provider and hosting arrangement
- Specific PostgreSQL queue library and worker connection model
- S3-compatible provider and storage region
- AI provider billing model (BYOK versus platform-managed)
- Search/vector technology, selected only after workload and privacy review
- Integration launch order and OAuth review effort
- Windows code-signing and updater distribution
- Retention defaults and subscription/pricing model

Each decision needs a recorded rationale, owner, validation method and cost impact before implementation.